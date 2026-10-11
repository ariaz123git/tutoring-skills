# add-diagrams — drawing helpers (computed-geometry SVG)

Reusable primitives for building a problem-solving diagram whose geometry is **computed, not
eyeballed** (an arrow that says 40° actually points 40°; a triangle actually closes). Run them
in Node at author-time and paste the emitted SVG string into the lesson/homework as a static
figure (prints fine, no runtime JS). Colours are **literal palette hex** (matching the standard
materials palette) — *not* CSS `var()` — so the figure renders identically in a browser **and
in a WeasyPrint PDF export** (WeasyPrint's SVG renderer does not resolve `var()`, so a
var-coloured arrow would silently render black in the PDF). Override the `color:` option
per-call to re-theme.

**Convention:** SVG y grows *downward*. Physics "up = +" means smaller y. Keep a helper
`PY(yPhysical)` in the caller if you map real units to pixels; for a schematic sketch you can
place elements directly. Always draw the axes/sign marker so the reader (and you) can't get a
sign wrong.

```js
// ---------- core ----------
const R = d => d * Math.PI / 180;
function svg(w, h, body, extra=''){
  return `<svg viewBox="0 0 ${w} ${h}" xmlns="http://www.w3.org/2000/svg" `
    + `font-family="Segoe UI,Arial,sans-serif" ${extra} style="width:100%;height:auto;max-width:${w}px">`
    + body + `</svg>`;
}
// Text carries a white halo by default (paint-order): a label that must sit near a stroke
// stays legible instead of being cut in half by it. The halo is a safety net for the
// unavoidable near-misses — it is NOT a licence to place labels carelessly, and WeasyPrint
// may ignore paint-order, so placement must still be correct on its own.
function txt(x,y,s,{size=12,fill='#1a2238',anchor='start',weight=400,italic=false,halo='#ffffff'}={}){
  return `<text x="${x}" y="${y}" font-size="${size}" fill="${fill}" text-anchor="${anchor}" `
    + `font-weight="${weight}"${italic?' font-style="italic"':''}`
    + (halo?` paint-order="stroke" stroke="${halo}" stroke-width="4" stroke-linejoin="round"`:'')
    + `>${s}</text>`;
}

// ---------- label placement (compute the side; never hand-pick an offset) ----------
// The commonest figure defect is a label that lands INSIDE the shape, on top of another
// stroke or label. A fixed `{dx,dy}` nudge is what causes it: it is correct for one geometry
// and wrong the moment an angle or scale changes. Place labels relative to the shape instead.

const centroid = (...pts) => [ pts.reduce((s,p)=>s+p[0],0)/pts.length,
                               pts.reduce((s,p)=>s+p[1],0)/pts.length ];

// Label a segment perpendicular to its midpoint, on the side AWAY from `away`
// (pass the triangle's centroid). Both normals are candidates; this picks the outward one.
function sideLabel(x1,y1,x2,y2,label,{away=null,off=26,...o}={}){
  const mx=(x1+x2)/2, my=(y1+y2)/2, dx=x2-x1, dy=y2-y1, L=Math.hypot(dx,dy)||1;
  let px=-dy/L, py=dx/L;
  if(away && (px*(away[0]-mx) + py*(away[1]-my)) > 0){ px=-px; py=-py; }  // flip to point outward
  return txt(mx+px*off, my+py*off+4, label, {anchor:'middle', ...o});
}

// Minimum radius at which a label of `textPx` width fits inside a wedge of `spanDeg`.
// The wedge is only 2·R·sin(span/2) wide at radius R, so a narrow angle (< ~25°) pushes the
// label onto one of its own arms unless R grows. Returns Infinity for a degenerate span.
function arcLabelRadius(spanDeg, textPx){
  const half = Math.abs(spanDeg)*Math.PI/360;
  return half < 1e-6 ? Infinity : textPx/(2*Math.sin(half));
}
// Decide how to label an angle: numerically if it fits, otherwise a symbol + a legend line.
// `armPx` = the shorter arm of the angle (the label cannot sit beyond the figure's own lines).
function angleLabelPlan(spanDeg, valueText, armPx, {size=12}={}){
  const need = arcLabelRadius(spanDeg, valueText.length*size*0.52);
  return need <= armPx*0.85
    ? {mode:'inline', r:Math.max(need, 18), text:valueText}
    : {mode:'symbol', r:Math.max(18, armPx*0.35), text:'&theta;',
       legend:`&theta; = ${valueText}`};   // draw the legend in a clear corner of the figure
}

// ---------- pre-render label audit (catches clipping + overlap before you look) ----------
// Approximate, deliberately: it uses a character-width estimate, so treat a clean result as
// "worth rendering now", never as proof. It reliably catches the two defects that survive
// logic checks — a label off the canvas edge, and two labels on the same spot.
function auditLabels(svgStr, {w, h, margin=4, charRatio=0.52} = {}){
  const warn=[], boxes=[];
  for(const m of svgStr.matchAll(/<text\b([^>]*)>([\s\S]*?)<\/text>/g)){
    const at = a => (m[1].match(new RegExp(a+'="([^"]*)"'))||[,''])[1];
    const x=+at('x'), y=+at('y'), size=+(at('font-size')||12), anchor=at('text-anchor')||'start';
    const s = m[2].replace(/<[^>]*>/g,'').replace(/&[a-z]+;|&#\d+;/g,'x');   // entity ≈ 1 glyph
    const wpx = s.length*size*charRatio;
    const x0 = anchor==='middle' ? x-wpx/2 : anchor==='end' ? x-wpx : x;
    const b = {x0, x1:x0+wpx, y0:y-size*0.8, y1:y+size*0.25, s};
    if(b.x0<margin || b.x1>w-margin || b.y0<margin || b.y1>h-margin) warn.push(`clipped: "${s}"`);
    for(const p of boxes)
      if(b.x0<p.x1 && b.x1>p.x0 && b.y0<p.y1 && b.y1>p.y0) warn.push(`overlap: "${s}" / "${p.s}"`);
    boxes.push(b);
  }
  return warn;   // author-time: if (warn.length) throw new Error(warn.join('\n'))
}

// ---------- arrow / vector (manual arrowhead = renders everywhere) ----------
function arrow(x1,y1,x2,y2,{color='#2f6fed',w=2,head=8}={}){
  const a = Math.atan2(y2-y1, x2-x1);
  const lx = x2 - head*Math.cos(a-0.4), ly = y2 - head*Math.sin(a-0.4);
  const rx = x2 - head*Math.cos(a+0.4), ry = y2 - head*Math.sin(a+0.4);
  return `<line x1="${x1}" y1="${y1}" x2="${x2}" y2="${y2}" stroke="${color}" stroke-width="${w}"/>`
    + `<polygon points="${x2},${y2} ${lx.toFixed(1)},${ly.toFixed(1)} ${rx.toFixed(1)},${ry.toFixed(1)}" fill="${color}"/>`;
}
// a labelled vector at an angle (deg measured CCW from +x, i.e. physics "up" is +)
function vecAt(x,y,deg,len,label,{color='#2f6fed',w=2,lab=true}={}){
  const x2 = x + len*Math.cos(R(deg)), y2 = y - len*Math.sin(R(deg)); // minus: screen-y down
  let s = arrow(x,y,x2,y2,{color,w});
  if(lab) s += txt(x2 + 4*Math.sign(Math.cos(R(deg)))-2, y2 - 4, label, {fill:color, size:11.5, weight:700});
  return s;
}

// ---------- angle arc from +x axis to a ray at `deg`, radius r, centred at (x,y) ----------
// `label` sits on the bisector. For a narrow wedge (roughly < 25°) a numeric label does NOT
// fit there and will overprint one of the arms — pass the wedge through angleLabelPlan first
// and fall back to a symbol + legend. `arm` = length of the shorter arm, so the label can't
// be pushed past the end of the lines that define the angle.
function angleArc(x,y,r,deg,label,{color='#5b6478',arm=Infinity,size=11}={}){
  const ex = x + r*Math.cos(R(deg)), ey = y - r*Math.sin(R(deg));
  const large = deg>180?1:0;
  const s = `<path d="M ${x+r} ${y} A ${r} ${r} 0 ${large} 0 ${ex.toFixed(1)} ${ey.toFixed(1)}" `
          + `fill="none" stroke="${color}" stroke-width="1.2"/>`;
  const plan = angleLabelPlan(deg, label, Math.min(arm, r*3), {size});
  const mid = R(deg/2), rl = Math.max(r+12, plan.r);
  return s + txt(x + rl*Math.cos(mid), y - rl*Math.sin(mid) + 4, plan.text,
                 {fill:color, size, anchor:'middle'})
    + (plan.legend ? `<!-- also draw in a clear corner: ${plan.legend} -->` : '');
}
// A general angle between two rays a1 -> a2 (not just from +x). Same wedge-width caveat.
function angleArcBetween(x,y,r,a1,a2,label,opts={}){
  const p = a => [x + r*Math.cos(R(a)), y - r*Math.sin(R(a))];
  const [x1,y1]=p(a1), [x2,y2]=p(a2);
  const large = Math.abs(a2-a1)>180?1:0, sweep = a2>a1?0:1;
  const s = `<path d="M ${x1.toFixed(1)} ${y1.toFixed(1)} A ${r} ${r} 0 ${large} ${sweep} `
          + `${x2.toFixed(1)} ${y2.toFixed(1)}" fill="none" stroke="${opts.color||'#5b6478'}" stroke-width="1.2"/>`;
  const plan = angleLabelPlan(a2-a1, label, opts.arm ?? r*3, {size:opts.size||11});
  const mid = R((a1+a2)/2), rl = Math.max(r+12, plan.r);
  return s + txt(x + rl*Math.cos(mid), y - rl*Math.sin(mid) + 4, plan.text,
                 {fill:opts.color||'#5b6478', size:opts.size||11, anchor:'middle'});
}
// Right-angle tick at vertex V, opening toward A and B (computed, so it sits in the corner
// that actually exists rather than assuming an axis-aligned square).
function rightAngle(vx,vy,ax,ay,bx,by,{size=10,color='#5b6478'}={}){
  const u=(px,py)=>{const dx=px-vx,dy=py-vy,L=Math.hypot(dx,dy)||1;return [dx/L,dy/L];};
  const [ux,uy]=u(ax,ay), [wx,wy]=u(bx,by);
  const p1=[vx+ux*size,vy+uy*size], p3=[vx+wx*size,vy+wy*size], p2=[p1[0]+wx*size,p1[1]+wy*size];
  return `<polyline points="${p1} ${p2} ${p3}" fill="none" stroke="${color}" stroke-width="1.4"/>`;
}

// ---------- coordinate axes / sign marker ----------
function axes(x,y,len=26,{xl='x',yl='y'}={}){
  return arrow(x,y,x+len,y,{color:'#8b93a5',w:1.5,head:6}) + txt(x+len+3,y+4,xl,{fill:'#8b93a5',size:10})
       + arrow(x,y,x,y-len,{color:'#8b93a5',w:1.5,head:6}) + txt(x-3,y-len-3,yl,{fill:'#8b93a5',size:10,anchor:'end'});
}

// ---------- dashed construction line ----------
function dash(x1,y1,x2,y2,{color='#b9c2d4'}={}){
  return `<line x1="${x1}" y1="${y1}" x2="${x2}" y2="${y2}" stroke="${color}" stroke-width="1.2" stroke-dasharray="4 3"/>`;
}

// ---------- dimension line (measure a distance, e.g. a height) ----------
function dimLine(x1,y1,x2,y2,label,{color='#5b6478'}={}){
  // double-headed arrow + label at midpoint
  let s = arrow(x1,y1,x2,y2,{color,w:1.2,head:6}) + arrow(x2,y2,x1,y1,{color,w:1.2,head:6});
  const mx=(x1+x2)/2, my=(y1+y2)/2;
  const vertical = Math.abs(x2-x1) < Math.abs(y2-y1);
  s += txt(mx + (vertical? -6:0), my + (vertical? 4:-4), label, {fill:color, size:11, anchor: vertical?'end':'middle'});
  return s;
}

// ---------- objects & ground ----------
function ground(x1,x2,y,{color='#1a2238'}={}){
  let s = `<line x1="${x1}" y1="${y}" x2="${x2}" y2="${y}" stroke="${color}" stroke-width="2"/>`;
  for(let gx=x1; gx<x2; gx+=8) s += `<line x1="${gx}" y1="${y}" x2="${gx-6}" y2="${y+7}" stroke="#b9c2d4" stroke-width="1"/>`;
  return s;
}
function ball(cx,cy,r,{color='#d8443c',label='',lc='#1a2238'}={}){
  return `<circle cx="${cx}" cy="${cy}" r="${r}" fill="${color}"/>` + (label?txt(cx+r+3,cy-r,label,{fill:lc,size:11.5,weight:700}):'');
}
function box(x,y,w,h,{fill='#e3e9f4',stroke='#c4ccd9',label=''}={}){
  return `<rect x="${x}" y="${y}" width="${w}" height="${h}" fill="${fill}" stroke="${stroke}"/>`
    + (label?txt(x+w/2,y+h/2+4,label,{anchor:'middle',size:11}):'');
}

// ---------- free-body diagram: forces radiating from one point ----------
// forces: [{deg, len, label, color}]   (deg CCW from +x)
function fbd(cx,cy,forces){
  let s = `<circle cx="${cx}" cy="${cy}" r="5" fill="#1a2238"/>`;
  for(const f of forces) s += vecAt(cx,cy,f.deg,f.len,f.label,{color:f.color||'#2f6fed',w:2.5});
  return s;
}

// ---------- motion diagram: dots at equal intervals + velocity arrows ----------
// pts: [[x,y],...]  spacing shows speeding up/slowing
function motion(pts,{dot=3.5,vel=true}={}){
  let s='';
  pts.forEach(([x,y])=> s += `<circle cx="${x}" cy="${y}" r="${dot}" fill="#1a2238"/>`);
  if(vel) for(let i=0;i<pts.length-1;i++) s += arrow(pts[i][0],pts[i][1],pts[i+1][0],pts[i+1][1],{color:'#1f9d6b',w:1.6,head:6});
  return s;
}

// ---------- right-triangle component decomposition (magnitude v at angle θ) ----------
function componentTriangle(x,y,vx,vy,{vlabel='v',xlabel='vₓ',ylabel='v_y',angLabel='θ'}={}){
  // (x,y)=start; vx,vy in px (vy up => negative screen). draws hypotenuse + legs + angle arc
  const hx=x+vx, hy=y-vy;
  let s = arrow(x,y,hx,hy,{color:'#2f6fed',w:2.5}) + txt((x+hx)/2-4,(y+hy)/2-5,vlabel,{fill:'#2f6fed',weight:700,size:11.5,italic:true});
  s += dash(x,y,hx,y) + txt((x+hx)/2, y+13, xlabel, {anchor:'middle',size:11,italic:true});
  s += dash(hx,y,hx,hy) + txt(hx+4,(y+hy)/2, ylabel, {size:11,italic:true});
  s += angleArc(x,y,20,Math.atan2(vy,vx)*180/Math.PI,angLabel);
  return s;
}
```

## Worked example — a projectile-from-a-cliff situation sketch (Step 1)

Shows the discipline: ground, cliff, ball, the launch vector at its angle with an arc, the
height as a dimension line, the axes, every given labelled, and the unknown (`range = ?`)
marked with a dashed line to the landing point.

```js
function projectileCliff({v=15,angle=40,h=25,W=360,H=210}={}){
  const gy=H-30, topY=gy-90, cliffX=44, cliffW=30, launchX=cliffX+cliffW, launchY=topY;
  let b='';
  b += ground(cliffX, W-10, gy);
  b += box(cliffX, topY, cliffW, gy-topY, {label:''});              // cliff block
  b += dimLine(cliffX-12, topY, cliffX-12, gy, `h = ${h} m`);        // height
  b += ball(launchX+6, launchY, 5, {label:''});
  b += vecAt(launchX+6, launchY, angle, 60, `v = ${v} m/s`, {color:'#1f9d6b'}); // launch vector
  b += angleArc(launchX+6, launchY, 24, angle, `${angle}°`);         // launch angle
  b += axes(W-28, 34, 22, {xl:'x', yl:'y'});                        // sign convention (top-right, clear of labels)
  b += dash(launchX+6, gy, W-14, gy);                              // baseline to landing
  b += txt((launchX + (W-14))/2, gy-9, 'range = ?', {fill:'#e8842c', weight:700, size:11.5, anchor:'middle'});
  return svg(W,H,b);
}
```

## Label placement — the rules that stop the rework

Geometry is the easy half. Labels are where figures actually fail, and the failures are
invisible to every check except looking at the picture.

- **Compute the side, don't nudge.** Use `sideLabel(..., {away: centroid(...)})` for anything
  labelling a segment. A hardcoded `{dx,dy}` is right for one geometry and wrong for the next.
- **A narrow wedge cannot hold a number.** Below roughly 25°, a value on the bisector overprints
  an arm. Use `angleLabelPlan`: label the arc `θ` and put `θ = 11.0°` in a clear corner.
- **Give text a halo** (default in `txt`) so an unavoidable near-miss stays readable — and fix
  the placement anyway.
- **Budget a margin.** `text-anchor="end"` at a small x runs off the canvas. `auditLabels`
  catches it; leave ≥ 4 px.
- **Two panels in one figure need a real gutter.** Labels grow inward from each panel's far
  end and meet in the middle. If they collide, stack the panels vertically or emit two separate
  figures — do not shrink the type.
- **Run `auditLabels` before you render**, and treat any warning as a defect, not a suggestion.

## Conventions checklist (bake into every figure)

- Every given appears on the figure as `symbol = value (unit)`, near the thing it describes.
- The unknown is a visible `?` (in an accent colour).
- Axes or a `+↑/+→` marker are present; the sign convention shown = the one the algebra uses.
- Vectors are labelled arrows; angles are arcs from the reference axis with the value on the arc.
- Labels placed by computed side (`sideLabel`/`angleLabelPlan`), not by hand-tuned offsets.
- `auditLabels` returns no clipping/overlap warnings.
- One-line caption above/below; schematic, uncluttered.
- Geometry from the helpers (computed), then **rendered to an image and eyeballed** before shipping.
