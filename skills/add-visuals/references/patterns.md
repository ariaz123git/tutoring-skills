# add-visuals — implementation patterns (bug-safe, drop-in)

Reusable skeletons for the animation engine, layout, and verification. They already encode
the fixes for the *Must-avoid bugs* in `SKILL.md` — start from these rather than hand-rolling
a loop. Everything is vanilla JS/CSS/SVG (no dependencies) so it drops into any self-contained
HTML lesson/homework. Reuse the target file's own CSS variables/palette for colours.

Two reveal modes:
- **Homework (gated):** wrap each visual as `…<button onclick="toggleViz(this)">Show visualization</button><div class="viz" data-kind="…" data-open="0" data-built="0"></div>` and let `toggleViz` lazy-build it.
- **Lesson (inline):** skip the button; call `buildViz(el)` for each `.viz` on load.

---

## 1. Builder contract

Each visual is a **builder** registered by `data-kind`. A builder fills the panel body with
SVG(s), caches its element refs, and returns an **api**:

- **Continuous** (scrub/play a single timeline): return `{ Tmax, update(t) }`.
- **Phased** (multi-stage, play one stage at a time): return
  `{ phased:true, Tmax, tB:[stageEndTimes], phases:[…], update(t, revealT), cardHTML(i), totalHTML? }`
  where `tB` are cumulative stage-end times and `update`'s optional `revealT` says how much of
  the graph/scene to draw (so stages reveal progressively).

`update(t)` is the ONLY place state is drawn. It must be cheap (setAttribute / textContent),
never a heavy `innerHTML` rebuild every frame.

```html
<script>
window.VIZ_BUILDERS = {};   // registry: data-kind -> builder(root, body) -> api
</script>
```

---

## 2. Toggle + lazy build + dispatch

```js
function toggleViz(btn){
  var viz = btn.nextElementSibling;
  if(viz.dataset.open === '1'){
    viz.style.display = 'none'; viz.dataset.open = '0'; btn.textContent = '▸ Show visualization';
  } else {
    if(viz.dataset.built !== '1'){ buildViz(viz); viz.dataset.built = '1'; }
    viz.style.display = 'block'; viz.dataset.open = '1'; btn.textContent = '▾ Hide visualization';
  }
}
function buildViz(viz){
  viz.innerHTML = '<div class="viz-body panels"></div><div class="ctrl"></div><div class="readout"></div>';
  var body = viz.querySelector('.viz-body');
  var api = window.VIZ_BUILDERS[viz.dataset.kind](viz, body);
  (api.phased ? wirePhased : wireContinuous)(viz, api);
}
// Lesson (inline): document.querySelectorAll('.viz').forEach(buildViz);
```

---

## 3. Continuous player — internal float clock (fixes restart + stutter)

The clock is an internal float (`curT`). The slider is **display only** — it is written to,
never read for state (a range input snaps its value to `step`, which is what breaks restart
and causes stutter). Time-based, so no per-frame drift.

```js
function wireContinuous(root, api){
  root.querySelector('.ctrl').innerHTML =
    '<button class="playbtn" type="button">Play</button>' +
    '<input class="scrub" type="range" min="0" max="' + api.Tmax + '" step="0.001" value="0" aria-label="time">';
  var slider = root.querySelector('.scrub'), btn = root.querySelector('.playbtn');
  var Tmax = api.Tmax, durMs = Math.min(6000, Math.max(2500, Tmax * 350)); // whole-run wall time
  var raf = null, playing = false, startTs = null, baseT = 0, curT = 0;    // curT = the real clock

  function render(t){ curT = Math.max(0, Math.min(Tmax, t)); slider.value = curT; api.update(curT); }
  function tick(ts){
    if(!playing) return;
    if(startTs === null) startTs = ts;
    var t = baseT + (ts - startTs) / durMs * Tmax;      // time-based (no accumulation drift)
    if(t >= Tmax){ render(Tmax); playing = false; raf = null; btn.textContent = 'Replay'; return; }
    render(t); raf = requestAnimationFrame(tick);
  }
  function play(){
    if(playing) return;                                  // guard double-start
    baseT = (curT >= Tmax - 1e-6) ? 0 : curT;            // restart from 0 when at the end
    playing = true; startTs = null; btn.textContent = 'Pause';
    raf = requestAnimationFrame(tick);
  }
  function pause(){ playing = false; if(raf) cancelAnimationFrame(raf); raf = null; if(btn.textContent === 'Pause') btn.textContent = 'Play'; }

  btn.addEventListener('click', function(){ playing ? pause() : play(); });
  slider.addEventListener('input', function(){ pause(); render(parseFloat(slider.value)); }); // read slider ONLY on user input
  render(0);
}
```

---

## 4. Phased player — uniform rate + persistent per-stage cards

Plays one stage, freezes its numbers onto a card that stays, then unlocks the next. **One
uniform rate** (`rate` ms per unit-time) so short and long stages feel consistent.

```js
function wirePhased(root, api){
  var nP = api.phases.length;
  var rate = Math.min(900, Math.max(220, 4500 / api.Tmax));   // uniform ms per unit-time
  root.querySelector('.ctrl').innerHTML =
    '<button class="playbtn" type="button">Play stage 1</button>' +
    '<button class="resetbtn" type="button">Reset</button>' +
    '<input class="scrub" type="range" min="0" max="0" step="0.001" value="0" disabled aria-label="time">';
  var cards = document.createElement('div'); cards.className = 'cards'; root.appendChild(cards);
  var pbtn = root.querySelector('.playbtn'), rbtn = root.querySelector('.resetbtn'), slider = root.querySelector('.scrub');
  var idx = -1, raf = null, startTs = null, curT = 0, unlockedT = 0, animating = false;

  function draw(t){ api.update(t, Math.max(unlockedT, t)); }   // reveal grows with unlocked stages
  function stopAnim(){ animating = false; if(raf) cancelAnimationFrame(raf); raf = null; startTs = null; }
  function finish(i){
    stopAnim(); idx = i; unlockedT = api.tB[i]; curT = unlockedT;
    slider.max = unlockedT; slider.disabled = false; slider.value = unlockedT; draw(unlockedT);
    cards.insertAdjacentHTML('beforeend', api.cardHTML(i));
    if(i + 1 >= nP){ pbtn.disabled = true; pbtn.textContent = '✓ All stages done'; if(api.totalHTML) cards.insertAdjacentHTML('beforeend', api.totalHTML()); }
    else { pbtn.textContent = 'Play stage ' + (i + 2); pbtn.disabled = false; }   // MUST re-enable Play, or only stage 1 is ever playable
    rbtn.disabled = false;
  }
  function playStage(){
    if(idx + 1 >= nP) return;
    var i = idx + 1, t0 = i ? api.tB[i - 1] : 0, t1 = api.tB[i], span = t1 - t0, durMs = Math.max(350, span * rate);
    curT = t0; animating = true; startTs = null; pbtn.disabled = true; rbtn.disabled = true;
    function tick(ts){ if(startTs === null) startTs = ts; var t = t0 + (ts - startTs) / durMs * span; if(t >= t1){ finish(i); return; } curT = t; draw(t); raf = requestAnimationFrame(tick); }
    raf = requestAnimationFrame(tick);
  }
  function reset(){ stopAnim(); idx = -1; unlockedT = 0; curT = 0; cards.innerHTML = ''; slider.max = 0; slider.value = 0; slider.disabled = true; pbtn.disabled = false; pbtn.textContent = 'Play stage 1'; draw(0); }

  slider.addEventListener('input', function(){ stopAnim(); curT = Math.min(parseFloat(slider.value), unlockedT); draw(curT); });
  pbtn.addEventListener('click', function(){ if(!animating) playStage(); });
  rbtn.addEventListener('click', reset);
  draw(0);
}
```

---

## 5. Layout CSS (prevents the too-tall / laggy balloon)

The `.panels` flex wrapper + capped panel width + SVG `max-height` are what stop a near-square
SVG from stretching to full-width-tall. Always wrap sibling SVG panels in `.panels`.

```css
.panels{display:flex;gap:12px;flex-wrap:wrap;align-items:center}
.panel{flex:1 1 240px;min-width:0;max-width:340px}     /* cap so a square SVG can't balloon */
.viz svg{width:100%;height:auto;display:block;max-height:260px}   /* hard height cap */
.ctrl{display:flex;align-items:center;gap:10px;margin-top:9px;flex-wrap:wrap}
.ctrl .scrub{flex:1;min-width:120px}
.playbtn{border:0;border-radius:7px;background:var(--blue,#2f6fed);color:#fff;font-weight:700;font-size:12px;padding:5px 12px;cursor:pointer}
.playbtn:disabled{background:#c7ccd6;cursor:default}
.resetbtn{border:1px solid var(--line,#dfe3ec);background:#fff;color:var(--muted,#5b6478);border-radius:7px;font-weight:700;font-size:12px;padding:5px 12px;cursor:pointer}
.readout{margin-top:8px;font-size:.85em}
.readout .v{font-weight:700}
.cards{display:flex;flex-wrap:wrap;gap:8px;margin-top:10px}
.card{flex:1 1 158px;border:1px solid var(--line,#dfe3ec);border-left:4px solid var(--muted,#5b6478);border-radius:8px;padding:7px 10px;font-size:12px;background:#fff}
@media print{ .ctrl,.playbtn,.resetbtn{display:none} }   /* if the file is ever printed */
```

---

## 6. Example builder (a scene synced to one graph)

Shows the pattern: compute geometry with mapping helpers, cache refs, update from a formula,
write a live readout. Positions are a **linear-in-time** attribute (good for testing, §8).

```js
window.VIZ_BUILDERS.motion = function(root, body){
  var d = root.dataset;                     // read params from data-* attributes
  var v0 = +d.v0, a = +d.a, Tmax = +d.tmax; // e.g. <div class="viz" data-kind="motion" data-v0="20" data-a="-5" data-tmax="4">
  function x(t){ return v0*t + 0.5*a*t*t; }
  function v(t){ return v0 + a*t; }
  var total = x(Tmax);

  // scene panel + graph panel, both inside .panel wrappers (so .panels flex applies)
  var W=300,H=120, trackY=70, sx = d => 20 + d/total*(W-40);
  var scene = '<div class="panel"><svg viewBox="0 0 '+W+' '+H+'"><line x1="20" y1="'+trackY+'" x2="'+(W-20)+'" y2="'+trackY+'" stroke="var(--line,#dfe3ec)" stroke-width="6"/><circle class="marker" cy="'+(trackY-10)+'" r="7" fill="var(--blue,#2f6fed)"/></svg></div>';
  var GW=260,GH=160,gx=t=>36+t/Tmax*(GW-48),gy=vv=>(GH-24)-vv/(v0*1.1)*(GH-38);
  var graph = '<div class="panel"><svg viewBox="0 0 '+GW+' '+GH+'"><polyline points="'+gx(0)+','+gy(v0)+' '+gx(Tmax)+','+gy(v(Tmax))+'" fill="none" stroke="var(--green,#1f9d6b)" stroke-width="2.5"/><circle class="gdot" r="4.5" fill="var(--orange,#e8842c)"/></svg></div>';
  body.innerHTML = scene + graph;

  var marker = body.querySelector('.marker'), gdot = body.querySelector('.gdot'), ro = root.querySelector('.readout');
  function update(t){
    marker.setAttribute('cx', sx(x(t)));
    gdot.setAttribute('cx', gx(t)); gdot.setAttribute('cy', gy(v(t)));
    ro.innerHTML = 't = <span class="v">'+t.toFixed(2)+'</span> s · v = <span class="v">'+v(t).toFixed(1)+'</span> · x = <span class="v">'+x(t).toFixed(1)+'</span>';
  }
  return { Tmax: Tmax, update: update };
};
```

For a **phased** builder, precompute `tB` (cumulative stage-end times) and a `phases` array of
per-stage numbers, expose `cardHTML(i)` returning the frozen-card markup, and have `update(t,
revealT)` draw the scene/graph up to `revealT` and the moving marker at `t`.

---

## 7. Static diagram helper (accurate geometry, not eyeballed)

Compute arc/vertex coordinates in code so angles and triangles are correct.

```js
// arc from the +x axis, radius r, sweeping to `deg`, centred at (cx,cy) — for labelling an angle
function anglePath(cx, cy, r, deg){
  var a = deg*Math.PI/180;
  return 'M '+(cx+r)+' '+cy+' A '+r+' '+r+' 0 0 0 '+(cx+r*Math.cos(a))+' '+(cy-r*Math.sin(a));
}
// draw a right-triangle for component -> magnitude/angle, with the value on the arc, plus an axis/compass key.
```

Always: size for clarity, one-sentence caption, axis/compass key, arc-labelled angle.

### 7a. In-scene value labels (put the number ON the thing it measures)

Label every quantity a visual depicts **next to the thing it measures**, the way a good physics
diagram is — not only in a separate readout line. The radius carries `r = 2.0 m` along the radius
line; a velocity arrow is tagged `v = 5.0 m/s` at its tip; the centripetal arrow reads
`a_c = 12.5 m/s²`; an angle shows its value on its arc. In an animated scene these labels are
**live**: update their `textContent` inside `update(t)` right alongside the geometry, and keep
each pinned to its element (offset a few px from the arrow tip / the midpoint of the radius). A
compact readout can stay, but the on-scene labels are the point — the student reads the value off
the picture. This is now the **default** for every visual, animated or static.

```js
// a labelled live vector: an arrow + a value tag that follow the element together
function mkVec(svg, cls, color){                 // returns {line, head, label}
  svg.insertAdjacentHTML('beforeend',
    '<line class="'+cls+'-ln" stroke="'+color+'" stroke-width="2.4" marker-end="url(#ah)"/>' +
    '<text class="'+cls+'-tx" font-size="10.5" font-weight="700" fill="'+color+'"></text>');
  return { ln: svg.querySelector('.'+cls+'-ln'), tx: svg.querySelector('.'+cls+'-tx') };
}
function setVec(v, x0,y0, x1,y1, name, val, unit){
  v.ln.setAttribute('x1',x0); v.ln.setAttribute('y1',y0); v.ln.setAttribute('x2',x1); v.ln.setAttribute('y2',y1);
  var nx=(x1-x0), ny=(y1-y0), L=Math.hypot(nx,ny)||1, ox=-ny/L*9, oy=nx/L*9;   // offset ⟂ to the arrow
  v.tx.setAttribute('x', x1+ox + (nx>0?2:-2)); v.tx.setAttribute('y', y1+oy);
  v.tx.textContent = name+' = '+val+' '+unit;                                  // live value, ON the arrow
}
// define the arrowhead marker once: <defs><marker id="ah" .../></defs>
```

Anti-collision: offset labels ⟂ to their arrow, keep font ~10–11 px, and nudge apart any two that
land on the same point; **verify by rendering (§8)** — overlapping labels are a visual-only defect.

### 7b. Worked-solution diagrams (sketch it the way a tutor would)

When a worked example materially benefits from a visual representation, its solution reveal
should include a **labelled diagram of the setup — the picture a tutor sketches while solving**:
the circle/track, graph, sign chart, radius with its value, given vectors tagged with their
numbers (§7a), or a light dashed construction line where it clarifies reasoning. Draw it from
the same mapping helpers so the geometry is accurate (§7), label relevant givens in-scene, and
caption the takeaway in one line. The diagram is **part of the solution, not decoration**.
Skip it for direct recall, routine algebra, and one-step substitutions.

**Graphs — one coordinate source (avoids the "curve starts off the axis" bug).** Draw axes and
gridlines from the **same** `mapY`/`mapX` that positions the data: the value-axis is a line at
`mapY(0)`, a gridline for value *k* is at `mapY(k)`. Never hardcode an axis's pixel separately
from the data's value→pixel *range* — if the two drift (e.g. axis at `GH-18` but the mapping
spans `GH-30`), curves start above/below the axis or overshoot it. This one is nasty because a
render made from the same code reproduces the misalignment, so eyeballing won't reliably catch
it — the shared-mapping structure is the real fix.

---

## 8. Verification harness (jsdom + simulated step-snapping) — REQUIRED

This is the test that catches the restart/stutter bug that logic-only checks miss. It patches
the range input to **snap to step like a real browser**, drives frames with a mocked clock, and
probes a full-precision attribute (not a rounded readout).

```js
// verify.js   —   npm i jsdom   then   node verify.js
const {JSDOM} = require('jsdom');
const script = require('fs').readFileSync('./viz.js', 'utf8');   // your extracted <script> contents
const dom = new JSDOM(
  '<!doctype html><body><button onclick="">x</button>' +
  '<div class="viz" data-kind="motion" data-v0="20" data-a="-5" data-tmax="4" data-open="0" data-built="0"></div></body>',
  { runScripts: 'outside-only' });
const win = dom.window;

// controllable requestAnimationFrame
win.__q = []; let id = 0;
win.requestAnimationFrame = cb => (id++, win.__q.push([id, cb]), id);
win.cancelAnimationFrame  = i  => { win.__q = win.__q.filter(x => x[0] !== i); };

// SIMULATE THE BROWSER: range inputs snap value to step (this is the bug surface)
const proto = win.HTMLInputElement.prototype, desc = Object.getOwnPropertyDescriptor(proto, 'value');
Object.defineProperty(proto, 'value', {
  get(){ return desc.get.call(this); },
  set(v){
    if(this.type === 'range'){
      const s = parseFloat(this.step)||1, mn = parseFloat(this.min)||0, mx = parseFloat(this.max);
      let n = Math.round((parseFloat(v) - mn)/s)*s + mn; if(!isNaN(mx)) n = Math.min(n, mx);
      desc.set.call(this, String(n));
    } else desc.set.call(this, v);
  }
});

win.eval(script);
const viz = win.document.querySelector('.viz');
win.toggleViz(win.document.querySelector('button'));            // build (or buildViz for inline)

const pos = () => parseFloat(viz.querySelector('.marker').getAttribute('cx'));  // full-precision, linear in t
function drive(){ let ts = 1000, seq = []; while(win.__q.length && seq.length < 1e5){ const [i, cb] = win.__q.shift(); cb(ts); ts += 16; const p = pos(); if(!isNaN(p)) seq.push(p); } return seq; }

let fails = 0; const ok = (c, m) => { if(!c){ console.log('FAIL ' + m); fails++; } else console.log('PASS ' + m); };

viz.querySelector('.playbtn').click();
const s1 = drive();
let stalls = 0, gaps = []; for(let i = 1; i < s1.length; i++){ const g = Math.abs(s1[i] - s1[i-1]); gaps.push(g); if(g < 1e-9) stalls++; }
const varc = Math.max.apply(null, gaps.slice(2, -2)) / Math.min.apply(null, gaps.slice(2, -2).filter(g => g > 1e-9));
ok(s1.length > 20, 'produced many frames (' + s1.length + ')');
ok(stalls === 0, 'no stall frames under step-snapping (smooth)');
ok(varc < 1.6, 'per-frame steps uniform (max/min=' + varc.toFixed(2) + ')');

viz.querySelector('.playbtn').click();          // replay
const s2 = drive();
ok(s2.length > 5 && Math.abs(s2[0] - s1[0]) < 0.5, 'restart returns to the start');

console.log(fails === 0 ? '\nALL VIZ CHECKS PASSED' : '\n*** ' + fails + ' FAILED ***');
process.exit(fails ? 1 : 0);
```

For a **phased** builder, additionally: click "Play stage N" for each stage, drain frames,
and assert a card appears per stage (and a total after the last), and that per-stage
ms-per-unit-time is roughly equal across stages (uniform rate).

Then **render to images and eyeball** the built HTML (headless capture, or WeasyPrint for
static SVG) to catch clipped labels, colliding text, and oversizing — defects no logic test sees.
