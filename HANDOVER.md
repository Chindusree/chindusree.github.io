# chindusree.github.io — working notes

Plain static site on GitHub Pages. No build step, no framework, no dependencies.
Push to `main` and it is live in about a minute.

> **If you are an AI coding agent, read `CLAUDE.md` first.** It carries the hard
> rules and the verification procedure in short form. This file is the detail
> behind them.

---

## 1. Layout

```
.nojekyll                       disables the Jekyll build — see §7
index.html                      homepage — one centred piece, two satellites
cold-gods-of-quantity/          The Cold Gods of Quantity  (2026)
walk-to-the-heavens/            A Walk to the Heavens      (2025)
reimagining-university/         The Age of Generation      (2025)
appeasing-the-cold-gods-of-quantity/   redirect stub only — see §6
```

Each essay directory holds:

```
index.html              the whole piece: markup, CSS and JS in one file
assets/cover.jpg        1200×630 social preview
assets/cover.html       the template the cover is rendered from
fonts/*.woff            et-book, copied locally per essay
```

Markdown sources are **not** kept in the repo. `index.html` is the source of truth;
edit the prose directly inside the `<p>` tags.

---

## 2. House conventions

**Headline case.** Title Case everywhere — `<title>`, `og:title`, `twitter:title`,
`<h1>`, cover image, homepage entry, citation line.

**Standfirst punctuation.** No full stop on the on-page `<h2 class="subtitle">` or on
the homepage entry. Full stop *is* used in `og:description`, `twitter:description` and
on the cover image.

**Serial commas.** Oxford throughout.

**Typography.** et-book, `#fdfcf9` ground, `#111` text, 660px column. Curly quotes and
em dashes are written as entities (`&ldquo; &rdquo; &rsquo; &mdash;`).

**Section heads.** Bare small-caps word, no numerals — *Overture, Dissection, Response,
Philosophy*. Section `id`s are words, not digits: `#dissection`, not `#2`. Digits are a
valid fragment but an invalid CSS selector, which will bite later.

**Links.** External open in a new tab with `rel="noopener"`. Internal (byline, Home,
the citation URL) stay in the same window.

**Section openers.** The first paragraph of each section opens with its first three or
four words in small caps:

```html
<p><span class="first-word">Most reviewer criticism</span> can be traced to…</p>
```

Break at a natural phrase boundary, not at a word count. Three or four words is the
usual length, but a comma or clause end takes precedence — *Cold Gods* opens its
Philosophy section on two words ("Peer review") because that is where the comma falls.
*Age of Generation* runs from one word to four for the same reason.

The opening paragraph of a piece also carries `class="raised"`, which enlarges and
lifts its initial letter. The two are designed to combine: a raised initial followed by
small caps is the intended look, not a collision. *Age of Generation* is the exception —
its raised paragraph is a short standalone question and takes no small caps.

---

## 3. The sidenote system

Shared, identical across all three essays. **Do not rebuild it.**

A note is two spans, hard against the preceding text with no space between:

```html
questions.<span id="ref-1" class="sidenote-number"></span><span
class="sidenote collapsible">note text</span> Does the study…
```

On load, the script numbers every `.sidenote` **in DOM order**, fills the matching
`#ref-N` with a superscript, and clones the notes into `#endnotes`.

### The one trap

Numbering follows DOM order, so:

- **Adding a note at the end** — safe.
- **Inserting one in the middle** — you must renumber every `ref-N` after it. The
  numbers you *see* will still look right; the links will point at the wrong notes.
- **Deleting one** — same.

### Behaviour by context

| | Desktop (>900px) | Mobile (≤900px) | Print |
|---|---|---|---|
| Margin sidenote | click superscript to reveal | hidden | hidden |
| Bottom sheet | disabled | tap superscript | hidden |
| `#endnotes` at foot | hidden | hidden | **shown, own page** |

Two blocks in each file are marked `portable block` — the mobile sheet (CSS + markup +
script) and the print stylesheet. They are byte-identical across the three essays. If
you change one, change all three.

The sheet dismisses on backdrop tap, grip tap, swipe down, or Esc; it locks background
scroll and respects `prefers-reduced-motion`.

### JavaScript is required

Superscript markers and the endnotes list are both *generated* by the script. With JS
off there are no markers and no notes at all. This has always been true; there is no
no-JS fallback to preserve.

---

## 4. Print

`@media print` reclaims the endnotes onto their own page and expands every external URL
after its link text, so a printed copy keeps its citations. Check with Cmd-P.

---

## 5. Covers

Rendered from `assets/cover.html`, never drawn by hand:

```bash
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --headless=new --disable-gpu --hide-scrollbars --force-device-scale-factor=1 \
  --window-size=1200,630 --virtual-time-budget=5000 \
  --screenshot=/tmp/cover.png "file://$PWD/<slug>/assets/cover.html"
sips -s format jpeg -s formatOptions 92 /tmp/cover.png --out <slug>/assets/cover.jpg
```

Change the title in `cover.html` and re-render — don't edit the JPEG.

---

## 6. Renaming an essay

`cold-gods-of-quantity` was once `appeasing-the-cold-gods-of-quantity`. When a slug
changes, leave a stub at the old path with a meta-refresh and a `rel="canonical"`.
GitHub Pages has no server redirects, so the stub is the only thing between an old
shared link and a 404. **Keep it permanently.**

Also update: `og:url`, both image URLs, the citation line, and both homepage links.

---

## 7. Deployment, and the Jekyll trap

Push to `main`; GitHub Pages serves it in under a minute. There is no CI and no
workflow file.

**`.nojekyll` must stay.** On 30 September a push sat undeployed for over fifteen
minutes with no error — the site kept serving the previous commit, and cache-busted
requests confirmed it was the build, not the CDN. Adding `.nojekyll` deployed the same
commit in thirty seconds.

By default Pages runs every push through Jekyll. This site uses no Jekyll feature at
all, so that step is pure risk: a silent build failure looks exactly like a slow deploy.
`.nojekyll` makes Pages copy the files and stop. Do not delete it.

**How to tell a stalled build from a cache.** Pick a string that exists only in the new
commit and request with a cache-buster:

```bash
curl -sSL "https://chindusree.github.io/<slug>/?cb=$RANDOM" | grep -c '<new string>'
```

If the string is absent, the commit has not deployed. The build log is under the repo's
**Deployments** tab on github.com — not visible from the command line without an
authorised `gh`.

---

## 8. Local preview

```bash
cd ~/Projects/chindusree.github.io
python3 -m http.server 8733 --bind 127.0.0.1     # this machine
python3 -m http.server 8733 --bind 0.0.0.0       # also reachable from your phone
```

For the phone, get the LAN address with `ipconfig getifaddr en0` and browse to
`http://<that address>:8733/cold-gods-of-quantity/`. Same network, and allow the
macOS firewall prompt.

---

## 9. Known issues

**`reimagining-university/index.html` — all cleared 1 Oct 2026.** The file is now
fully balanced: 23/23 `div`, 8/8 `section`, 147/147 `p`, no duplicate ids, one
`</head>`.

What was wrong, and how it was fixed:

- **Duplicate `id="ref-51"`** — an orphaned empty marker span sat just before the real
  one, so the script matched the orphan by id and left the real span blank. Orphan
  removed.
- **Two `</head>` tags** — the second deleted.
- **Unclosed `<div class="subsection">`** (~line 758) — closed before the epilogue.
- **Unclosed `<section id="unmaking">`** — closed in the same place.
- **Three unclosed `<p>`** — browsers auto-closed these, which is why nothing ever
  looked wrong. Each `</p>` was inserted exactly where the parser already inferred it.

**Verified no visual change.** Rendered `innerText` was hashed before and after: same
76,553 characters, same hash, same 197 paragraphs, same 51 notes and endnotes. Worth
repeating that method on any future structural edit to a published piece.

It does not carry the small-caps section-opener convention consistently, and its
sections are named (*Prologue, Genesis, Unmaking…*) rather than following the
Overture/Dissection pattern. Both are deliberate to that piece.

**Homepage** has no breakpoint above 700px; the two satellites are positioned by
percentage and will crowd if a fourth piece is added.

---

## 10. Verifying a change

Nothing here has tests, so verification is manual and worth doing the same way each
time. Serve locally, then drive the page in headless Chrome through an iframe harness
— an iframe gives a true narrow viewport, which `--window-size` cannot, since Chrome
clamps headless windows at 500px wide.

Assert on every change touching notes or layout:

- superscripts and `data-number`s both run `1..N`, no gaps
- endnote count equals sidenote count
- `documentElement.scrollWidth == clientWidth` at 390px
- tags balanced, no duplicate ids
- sheet opens with the right note below 900px; `display:none` above it
- desktop margin sidenote still reveals on click

**For structural edits to a published page, prove the rendering did not move.** Hash
the rendered text before and after:

```js
var txt = document.body.innerText.replace(/\s+/g,' ').trim();
var h=0; for (var i=0;i<txt.length;i++) h=((h<<5)-h+txt.charCodeAt(i))|0;
```

Identical hash plus identical paragraph count proves the markup changed and the reader
experience did not. This is how the 1 Oct repairs to *Age of Generation* were cleared:
76,553 characters, 197 paragraphs, 51 notes, same hash before and after.

For print, render to PDF — `--headless=new --print-to-pdf` applies print CSS — and read
the last pages to confirm the endnotes and their URLs are there.

---

## 11. Recovery

Tags mark the state before each significant change:

```
release-2026-10-01           current live state — all three essays, sheet + print
v-sheet-standard-2026-09-30  sheet + print ported (pre-.nojekyll)
pre-sheet-port               before the sheet reached the other two essays
pre-bottom-sheet             before the mobile sheet existed
pre-restructure-2026-09-29   six-section version of Cold Gods
```

```bash
git checkout <tag> -- <path>      # restore one file
```

---

*Last updated 1 October 2026.*
