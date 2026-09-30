# chindusree.github.io — working notes

Plain static site on GitHub Pages. No build step, no framework, no dependencies.
Push to `main` and it is live in about a minute.

---

## 1. Layout

```
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

## 7. Local preview

```bash
cd ~/Projects/chindusree.github.io
python3 -m http.server 8733 --bind 127.0.0.1     # this machine
python3 -m http.server 8733 --bind 0.0.0.0       # also reachable from your phone
```

For the phone, get the LAN address with `ipconfig getifaddr en0` and browse to
`http://<that address>:8733/cold-gods-of-quantity/`. Same network, and allow the
macOS firewall prompt.

---

## 8. Known issues

**`reimagining-university/index.html`**
- Unclosed `<div class="subsection">` at line ~758 — pre-dates all current work.
- Two `</head>` tags.
- ~~Duplicate `id="ref-51"`~~ — fixed 30 Sep 2026. An orphaned empty marker span sat
  just before the real one; the script matched the orphan by ID, leaving the real span
  blank. Removing the orphan restored valid HTML with no change on screen.

It does not carry the small-caps section-opener convention consistently, and its
sections are named (*Prologue, Genesis, Unmaking…*) rather than following the
Overture/Dissection pattern. Both are deliberate to that piece.

**Homepage** has no breakpoint above 700px; the two satellites are positioned by
percentage and will crowd if a fourth piece is added.

---

## 9. Recovery

Tags mark the state before each significant change:

```
pre-restructure-2026-09-29   six-section version of Cold Gods
pre-bottom-sheet             before the mobile sheet
pre-sheet-port               before the sheet reached the other two essays
```

```bash
git checkout <tag> -- <path>      # restore one file
```

---

*Last updated 30 September 2026.*
