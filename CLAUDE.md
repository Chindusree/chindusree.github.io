# CLAUDE.md

Read this before touching anything. Full detail in `HANDOVER.md`.

This is a hand-written static site on GitHub Pages. **No build step, no framework, no
dependencies, no package.json.** Each essay is one self-contained `index.html` holding
its own markup, CSS and JavaScript. That is deliberate. Do not introduce a bundler, a
static site generator, a CSS framework, or a shared stylesheet unless the author asks.

---

## Hard rules

**1. `.nojekyll` must stay.** Deleting it reintroduces the Jekyll build, whose failure
mode is silent — a push sits undeployed with no error. This cost 15 minutes on
30 Sep 2026. See `HANDOVER.md` §7.

**2. Never renumber sidenotes by hand without checking DOM order.** The script numbers
`.sidenote` elements in document order and matches them to `#ref-N`. Inserting a note
mid-document means renumbering every `ref-N` after it. The page will *look* right and
the links will be wrong. Always verify after editing notes:

```
sups == dataNums == 1..N, endnote count == note count
```

**3. Three blocks are byte-identical across all three essays.** They are marked
`portable block` in each file:
- the mobile note sheet (CSS + markup + script)
- the print stylesheet

Change one, change all three, then verify by hashing the extracted blocks.

**4. Markdown is not the source.** `index.html` is. There is no `.md` for any essay;
don't generate one and don't treat a stale export as authoritative.

**5. Don't "fix" the author's prose.** Apply edits as given. Where a paste has lost
italics or reverted an earlier agreed edit — common when the author pastes from an
older draft — preserve the live version and *say so*, rather than silently accepting
or silently overriding.

**6. Never commit internal reports or assessments.** `.nojekyll` serves every file in
this repo raw, at a guessable URL, with no exclude mechanism. `REPORT-*.md` and
anything of that kind must live outside the served repo. `HANDOVER.md` and this file
may stay — they are operational notes with nothing sensitive in them. Check before
adding any new file at the repo root or in a new directory.

---

## Before you edit

```bash
cd ~/Projects/chindusree.github.io
git tag -a pre-<what-you-are-doing> -m "state before X"
python3 -m http.server 8733 --bind 127.0.0.1
```

Preview at `http://127.0.0.1:8733/<slug>/`. For the author's phone, bind `0.0.0.0` and
use `ipconfig getifaddr en0`.

---

## After you edit — the standard check

Load the page in headless Chrome through an iframe harness and assert:

- superscripts and `data-number`s both run `1..N` with no gaps
- endnote count equals note count
- `documentElement.scrollWidth == clientWidth` at 390px (no horizontal overflow)
- tag balance and no duplicate ids
- the bottom sheet opens with the right note at ≤900px, and is `display:none` above it

**For structural edits to a published page, prove nothing moved.** Hash the rendered
text before and after:

```js
var txt = document.body.innerText.replace(/\s+/g,' ').trim();
var h=0; for (var i=0;i<txt.length;i++) h=((h<<5)-h+txt.charCodeAt(i))|0;
```

Identical hash and paragraph count means the markup changed and the reading experience
did not. This is how the *Age of Generation* structural repairs were validated.

---

## Deploying

Push to `main`. Live in under a minute.

**Verify the deploy actually happened** — pick a string unique to the new commit:

```bash
curl -sSL "https://chindusree.github.io/<slug>/?cb=$RANDOM" | grep -c '<new string>'
```

Zero means it has not deployed. Do not assume a cache. Check `git show
origin/main:<path>` to confirm the blob is right, then look at the repo's Deployments
tab.

---

## Working with the author

- Short and decisive. Give a recommendation, not a menu of options.
- Nothing goes live without an explicit instruction to push.
- Tag before significant changes; the author asks for revertability by name.

**Flag only genuine faults.** Do not manufacture issues, hedge, or over-qualify in
order to look thorough. A false flag spends the author's attention and buries the real
findings among noise. Where you are unsure whether something is wrong, say so plainly
as uncertainty — do not assert it as a defect. The author's prose and citation choices
stand unless you can point to a concrete, demonstrable error.

Don't narrate standard practice as though it were a judgement call, either. If a thing
is simply what one does, do it and move on.

---

## Current state

Live tag `release-2026-10-01`. Three essays, all carrying the note sheet and print
stylesheet. `reimagining-university/index.html` was structurally repaired on 1 Oct and
is now fully balanced.

The author has ruled on the open editorial items. They are settled; do not reopen them
or re-flag them. Detail sits in the publication report, which is kept outside this
repo.

The one genuinely open structural matter: the homepage positions its satellites by
percentage and will need rethinking rather than extending at a fourth essay.
