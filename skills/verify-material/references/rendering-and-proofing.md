# Rendering & proofing — how to actually look at what you built

Every build skill ends with "render it to an image and look at it." That instruction fails in
practice for a boring reason: **a long HTML page does not fit in a screenshot**, so the pass
gets skipped or done on the first viewport only, and defects ship from the parts nobody saw.

This file is the mechanics. Referenced by `add-diagrams`, `add-visuals`, `create-homework`,
`create-lesson`, `verify-material`, and `verify-light`.

---

## 0. The expectation that makes the pass work

**Assume the first render of any figure is wrong.** Across builds, the recurring defects are
labels sitting inside a shape, labels overprinting a line, text clipped at the canvas edge, and
multi-panel figures whose labels collide across the gutter. None of these are visible to tag
checks, `node --check`, or answer verification — and none of them are rare.

So budget the loop: **generate → audit → render → look → fix → re-render.** One render is a
formality; the second one is the check. If a figure passed on the first look, be suspicious that
you looked at a thumbnail rather than the figure.

**Proof figures separately from the page.** Emit all figures into a scratch preview page, one
per row, at their true size, and inspect that first. Figure defects are far easier to see
without the surrounding prose, and it costs one extra render.

---

## 1. Headless capture (Chrome/Edge)

Chrome is available on Windows and macOS and needs no Python imaging stack.

```bash
chrome --headless --disable-gpu --no-sandbox \
  --user-data-dir="<scratch>/chromeprofile" \
  --screenshot="<abs-out>.png" --window-size=880,1400 --hide-scrollbars \
  --virtual-time-budget=3000 "file:///<abs-in>.html"
```

Non-obvious flags, each of which is load-bearing:

- **`--user-data-dir` is required.** Without it the screenshot can fail with `Access is denied`
  and write nothing. The failure is silent unless you check the exit status.
- **Absolute paths** for both `--screenshot` and the `file:///` URL.
- **`--virtual-time-budget`** lets fonts/SVG settle before the capture.
- **`--allow-file-access-from-files`** only if the page pulls in another local file (e.g. an
  `<iframe>`, as in the segment trick below).

Windows paths: give the `.exe` a Windows-style output path even when invoking from a POSIX
shell, and quote it.

## 2. Capturing a page taller than the window

`--screenshot` captures the **viewport only** — anchors (`#id`) do not scroll it, so naive
attempts silently return the top of the page N times.

**Step 1 — measure the real height.** Write the height into the title and dump the DOM:

```bash
# inject before </body>:  <script>document.title="H="+document.documentElement.scrollHeight;</script>
chrome --headless --dump-dom --virtual-time-budget=3000 "file:///…/measure.html" | grep -o 'H=[0-9]*'
```

**Step 2 — capture segments** through a scrolled `<iframe>` (works without a debugger protocol):

```html
<!-- seg.html: one per offset Y -->
<style>html,body{margin:0;padding:0;overflow:hidden}
iframe{position:absolute;top:-YPX;left:0;width:880px;height:TOTALpx;border:0}</style>
<iframe src="page.html" scrolling="no"></iframe>
```

Capture each at `--window-size=880,<segment height>` with `--allow-file-access-from-files`.
Overlap segments slightly so nothing hides on a seam.

**Expand collapsed content first.** Anything behind `<details>`/toggles renders closed and its
contents are never proofed. Write a scratch copy with `<details>` → `<details open>` (and any
`display:none` panels forced visible) and proof *that*; ship the original.

## 3. PDFs

`--print-to-pdf` is the reliable route when WeasyPrint/ReportLab aren't installed. It embeds
subsetted fonts, so Unicode math (`√ Δ × − ½ ² °`) renders without the glyph-box problem that
ReportLab's built-in fonts have.

```bash
chrome --headless --disable-gpu --no-sandbox --user-data-dir="<scratch>/chromeprofile" \
  --print-to-pdf="<abs-out>.pdf" --no-pdf-header-footer --virtual-time-budget=5000 \
  "file:///<abs-in>.html"
```

**Rasterising a PDF to look at it** needs poppler (`pdftoppm`) or `pypdfium2`, which may not be
installed. When they aren't, proof the **print HTML** in segments (§2) rather than the PDF, and
check the PDF itself structurally:

```python
len(re.findall(rb'/Type\s*/Page[^s]', pdf_bytes))          # page count
set(re.findall(rb'/BaseFont\s*/([A-Za-z0-9+\-]+)', pdf_bytes))   # fonts embedded
```

A page count far above your estimate is itself a defect signal — see §4.

## 4. Print CSS: `break-inside: avoid` is not free

Applying `break-inside/page-break-inside: avoid` to a block **taller than one page** makes the
renderer push it to a fresh page and *still* break it, leaving most of a page blank. Applied to
long solution blocks it inflates page count substantially with whitespace.

The working split:

| Apply `break-inside: avoid` to | Use `break-after: avoid` on | Leave breakable |
|---|---|---|
| figures, tables, callout boxes, short problem blocks — things that fit on a page | headings, step labels, and any caption that must stay with its figure | long solution/section bodies |

**It fails in both directions, so measure before you choose.** Too little `avoid` orphans a
block from its heading (a work box stranded at the top of the next page). Too much, applied to
blocks that don't tile the page, wastes most of a page each time one won't fit in the remainder
— the pathological case is *one item per page* with half of each page blank.

So size the repeating block to the page rather than fighting the paginator. Measure the real
rendered heights first — don't estimate:

```js
// inject before </body>, then run headless with --dump-dom and read the title
document.title = [...document.querySelectorAll('h2,.card')]
  .map(e => e.className + ':' + Math.round(e.getBoundingClientRect().height)).join('|');
```

Compare against the printable content height (US Letter at 15 mm margins ≈ **947 px** at 96 dpi;
A4 ≈ 1010 px) and adjust the block — usually by trimming generous whitespace — so a whole number
of them tiles a page. Then confirm the page count is close to your estimate; a large excess in
either direction means the block and the page still disagree.

## 4b. Monospace fonts draw `√` without an overbar

An equation set in the material's `.mono` face (Cascadia Code / Consolas) renders U+221A as a
bare rising stroke with **no bar over the radicand**. At body size it reads as a capital `J`, and
`√(a² + b²)` becomes ambiguous — the radicand's extent is carried only by the parentheses.

This is the one glyph the monospace face genuinely gets wrong for school maths. Take the sign
from a math face and draw the bar with a border:

```css
.rad{white-space:nowrap}
.rad>i{font-family:"Cambria Math","Segoe UI Symbol","DejaVu Sans",serif;font-style:normal;font-size:1.1em}
.rad>u{text-decoration:none;border-top:1.3px solid currentColor;padding:2px 3px 0 2px;margin-left:-2px}
```
```html
<span class="rad"><i>&radic;</i><u>6.0<sup>2</sup> + 2.0<sup>2</sup></u></span>
```

Convert them at build time (match the radicand's parentheses **by depth**, not with a non-greedy
regex) and assert no bare `&radic;` survives. Everything else — `− × ° α θ φ → ≤ ² ₀` — renders
correctly in the monospace face and needs no special handling.

## 5. Don't judge glyphs from a downscaled render

A superscript `²` reads convincingly as `³` in a downscaled screenshot, and thin strokes drop
out entirely. **Never "fix" the source from a low-resolution render.** Confirm against the file
first — grep the entities (`&sup2;` vs `&sup3;`, literal `²` vs `³`) — and only then decide
there is a defect. Re-render the suspect region at larger scale if you still need to see it.

## 6. Verify the checker before you "fix" the artifact

A failing assertion in a validation script is roughly as likely to be a bug in the assertion as
a defect in the file. Two that recur:

- **A hardcoded expected count** that was simply counted wrong when the script was written.
- **A substring test whose exclusion is applied to the wrong operand** — e.g. checking for
  external URLs with `'http://' not in s and 'https://' not in s.replace(namespace,'')`, where
  the first clause still sees the stripped-out namespace. SVG's `xmlns="http://www.w3.org/2000/svg"`
  is a namespace identifier, not a network fetch; whitelist it explicitly.

When a check fails: **read the matched text before editing the file.** Print the surrounding
context, confirm the finding is real, then fix whichever of the two is actually wrong.

## 7. Author scripts with the file tools, not shell heredocs

Generation and validation scripts belong in a file written with the editor tool. Heredocs
mangle escapes (`"\n"` arriving as a literal newline and breaking the string), and quoting
interacts badly with the shell. Symptoms — `SyntaxError: unterminated string literal`,
`unexpected EOF while looking for matching quote` — are shell artifacts, not logic bugs.

Also set `PYTHONIOENCODING=utf-8` for any Python that prints non-ASCII on Windows, or printing
`−`/`×`/`√` raises `UnicodeEncodeError` from `cp1252` and kills the script mid-run.

## 8. Extracting content from a built HTML file

When one artifact is generated from another (e.g. a print twin built from the interactive
file — the right way to keep them from drifting), **do not carve blocks out with a non-greedy
regex**. `<div class="card">(.*?)</div>` stops at the *first* nested `</div>` and silently
returns a truncated block; the count of matches can still look correct.

Match by depth instead, and assert on the result:

```python
def extract_blocks(text, opener):
    """Inner HTML of each `opener` block, matching <div>/</div> by depth."""
    out = []
    for m in re.finditer(re.escape(opener), text):
        i, depth = m.end(), 1
        while depth:
            nxt = re.search(r'<div\b|</div>', text[i:])
            if not nxt: raise ValueError("unbalanced %s" % opener)
            depth += 1 if nxt.group(0) != '</div>' else -1
            i += nxt.end()
        out.append(text[m.end():i - len('</div>')])
    return out
```

Then assert each extracted block still contains the marker it must contain (a solution reveal,
a figure). A silent truncation that passes a count check is the failure mode this prevents.
