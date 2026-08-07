# Website architecture playbook

For whoever — person or AI assistant — maintains this site next. What the pieces
are, why they were chosen, and which decisions are load-bearing.

Owner-facing instructions are in [UPDATING.md](UPDATING.md). This is the
engineering side.

---

## 1. What this site is

A **single page**, served from `https://weitingyen-as.github.io/weitingyensite/`.
There is exactly one HTML page plus a 404. Sections stack in this order:

| Section | `id` | Source |
|---|---|---|
| Hero | — | `content/_index.md` front matter |
| About | `#about` | `content/bio/_index.md` |
| Research | `#research-areas` | `content/publication/*` (articles and chapters) |
| Public Writing | `#public-writing` | `content/public-engagement/_index.md` + op-eds + edited volumes |
| Contact | `#contact` | `content/contact/_index.md` |

Menu entries are in-page anchors. There is no footer.

`#research-areas` is a historical id — it was the accordion section before the
hashtag rewrite. It stayed so that any link already shared still lands correctly.

### The single-page mechanism

Sub-pages remain **separate Markdown files** so they are pleasant to edit, but
emit no URLs of their own. Each section index carries:

```yaml
cascade:
  build:
    render: never
    list: always
build:
  render: never
  list: always
```

`layouts/landing/home.html` pulls them in with `site.GetPage` and renders
`.Content` inline. `list: always` keeps them in `site.RegularPages` so the
publication query still finds them.

**It is `build:`, not `_build:`.** The underscore form was removed in Hugo 0.145
and now fails the build with an error rather than a warning.

---

## 2. Stack

| Layer | Choice |
|---|---|
| Generator | Hugo **extended**, ≥ 0.148.2 (CI pins 0.164.0) |
| Theme module | `blox-tailwind` v0.10.0, vendored in `_vendor/` |
| Styling | One hand-written stylesheet, `assets/css/custom.css` |
| Search | Pagefind 1.5, built in CI after Hugo |
| Hosting | GitHub Pages project site |
| CI | `.github/workflows/deploy.yml` on push to `main` |

**Hugo extended is required** and the version floor is real: Blox v0.10.0 refuses
to load below 0.148.2.

**Hugo ≥ 0.146 template layout.** Templates live at `layouts/baseof.html`,
`layouts/_partials/…`, `layouts/_shortcodes/…` — *not* the old
`layouts/_default/` and `layouts/partials/`.

### The deliberate departure from Blox

**This site uses none of Blox's CSS or page templates.** `layouts/baseof.html` is
a complete replacement shell that loads `custom.css` directly and never calls
Blox's `_partials/css.html`.

Why: the owner chose <https://yawenlei.com/> as the visual model, and matching a
specific design is far more reliable in ~400 lines of plain CSS than by fighting
Tailwind utility classes emitted by someone else's blocks. It also means no
Tailwind CLI at build time.

The module is still imported and vendored so its helper partials stay available.
If you ever wire up a Blox *visual* component you will need to add
`{{ partial "css.html" . }}` to `baseof.html` and keep `@tailwindcss/cli` in
`package.json` (it is already there for that reason).

Files that no longer exist because nothing renders them: `layouts/list.html`,
`layouts/single.html`, `i18n/en.yaml`, `data/`. Adding a second page means
writing a template for it deliberately.

---

## 3. Traps that cost real time here

Every one of these produced a **silently wrong result**, not a build error.

**Menu URLs are already resolved.** Hugo resolves `site.Menus.main` `.URL`
against `baseURL`. Passing it through `relURL` again yields
`/weitingyensite/weitingyensite/…` and 404s every navigation link. Use `.URL`
raw. This is invisible on the dev server, whose baseURL has no subpath.

**`jsonify` inside a `<script>` double-escapes.** Go applies JS-context escaping
*on top of* jsonify's quoting, so `{{ $url | jsonify }}` became a string
containing a quoted string and the Pagefind import failed. For a plain string,
interpolate inside quotes: `var URL = "{{ "pagefind/pagefind.js" | relURL }}";`.
For a JSON payload in `<script type="application/json">`, use `| jsonify | safeJS`.

**Stale CSS rules win by source order.** Three iterations of the Research section
left dead rules further down the stylesheet; `#research-list .pub-item { padding:
.85rem 0 }` beat the card rule and the padding silently did nothing. When a style
"doesn't work", grep the whole file for the selector before changing the value.

**The dev server does not fingerprint assets.** An unfingerprinted
`custom.css` gets cached by the browser and serves stale CSS through repeated
edits. `baseof.html` therefore fingerprints in **every** environment, minifying
only in production. Don't "simplify" that back.

**The dev server writes to `public/`.** Running `hugo --gc --minify` while
`hugo server` is up means the server overwrites the production build with
localhost URLs. Stop the server before a build you intend to inspect. Deleting
`resources/` under a running server also leaves its asset cache stale — restart it.

### Subpath safety

The site is served from `/weitingyensite/`, so a literal `/files/foo.pdf`
resolves to the host root and 404s.

- Templates: `{{ "files/x.pdf" | relURL }}` — **no leading slash**.
  `relURL "/x"` returns `/x` and escapes the base path.
- Front matter `links:`: write `url: "files/paper.pdf"`. The card template passes
  anything not starting with `http` through `relURL`.
- Markdown body: `{{</* staticrel "files/paper.pdf" */>}}`.

---

## 4. Content model

Everything lives in `content/publication/` as a leaf bundle — a directory
containing `index.md`. PDFs go in `static/files/`.

`publication_types` is a small controlled vocabulary, and it is what routes an
item to a section of the page:

| Value | Rendered in | Count |
|---|---|---|
| `journal_article` | Research | 16 |
| `book_chapter` | Research | 4 |
| `op_ed` | Public Writing → Commentary | 23 |
| `edited_volume` | Public Writing → cards at top | 2 |

`op_ed` and `edited_volume` are not upstream Blox values. They exist because the
owner draws a hard line between refereed scholarship and public-facing writing,
and the page is built around that split.

### The citation line

`venue` / `venue_detail` / `venue_prefix` exist so the journal or book title can
be picked out in the accent colour, the way the reference design does it. They
were split out of `publication`, which is retained as a fallback and as the full
citation of record. Parsing them back out of one string is not viable — journal
names contain commas.

The year is printed above the title, so it is deliberately **absent** from
`venue_detail`. Adding it back produces a visible duplicate.

`abstract` is populated on every article and chapter but **not rendered**. It was
dropped when the cards were compacted. Restoring it is one block in
`layouts/_partials/citation_row.html`; if you do, make it click-to-expand rather
than always-on, or the list becomes unscannable.

Front matter that is **read**: `title`, `date`, `authors`, `publication_types`,
`venue`, `venue_detail`, `venue_prefix`, `links`, `award`, `hashtags`,
`publication` (fallback). Everything else — `doi`, `tags`, `abstract` — is stored
only.

---

## 5. The Research section

`layouts/landing/home.html` holds both the markup and the filter script.

**Hashtags have no registry.** The chip bar is derived at build time by counting
`hashtags:` across articles and chapters, ordered by frequency then alphabetically.
Adding a tag to a paper is the only step. This is the single most important
property of the design — there is no list to forget to update.

**Filtering is intersection, not union.** Selecting several hashtags requires a
paper to carry all of them, so combinations narrow. Chips that would return
nothing on top of the current selection get `.dim` rather than leading to an
empty list.

**The list is server-rendered in full.** JavaScript only toggles `hidden`. With
JS off, all 20 publications still read in order. Do not change this to
client-side rendering.

**Progressive disclosure.** `LIMIT = 6`. The button counts the *filtered* set and
hides itself when the filter already brings the count under the limit.

**State lives in `?tags=`, not the hash.** The hash is in use for section anchors;
putting filter state there would make the nav links clear the filter. Query
string via `history.replaceState` keeps both working and makes filtered views
shareable.

**Cards echo the reference design's "Selected Articles" format**: 1rem gaps,
1.25/1.5rem padding, a resting shadow, and a 1px lift with an accent border on
hover. Hashtags on a card use the same pill as the filter bar so the two read as
the same object, and light up when they are the ones doing the filtering.

---

## 6. Design system

`assets/css/custom.css` is the whole visual language. The palette is **Academia
Sinica's institutional identity**, sampled from the specification printed on the
emblem artwork rather than approximated:

| Token | Value | Official spec | Role |
|---|---|---|---|
| `--color-sinica-blue` | `#005b94` | C100 M70 Y20 | primary / interactive |
| `--color-sinica-blue-mid` | `#0082b4` | C80 M70 Y20 | section rules, focus ring |
| `--color-sinica-khaki` | `#b8a887` | C30 M30 Y50 | topical labels |
| `--color-sinica-cream` | `#d8d0bd` | C15 M15 Y25 | surfaces, rules |

Every surface, border and text value is a tint of those four. Semantic tokens
(`--color-accent`, `--color-link`, `--color-border`, …) point at them, so a
palette change is confined to `:root`.

**Colours mean things.** Blue = clickable or institutional. Khaki = a topic
label, never interactive-looking. Gold-family = an award. Don't reuse them.

**The hero is the one deliberate exception.** Its gradient is a warm blush
(`#f3e3dc`), not the institutional blue: the owner's portrait has a red jacket
and the cool blue fought it. Everything below the hero is blue. If you "fix" this
for consistency, you will be reverting a considered decision.

**Typography is a native system stack** — serif for headings, OS sans for body.
The reference site loads Inter and Noto Serif Display from Google Fonts; this one
does not, because nothing needed to render the site should depend on a
third-party host that can go dark or log visitors. The typographic *structure* is
preserved.

**Light mode is forced.** `.dark` is overridden back to light surfaces.

Layout invariants:

- Hero is **photo left, text right** on desktop, stacked and centred at ≤640px.
- Navbar brand is her full name, uppercase, far left, always linking to `/`.
  Search is a bare magnifying glass.
- Every section id carries `scroll-margin-top: calc(var(--nav-h) + 16px)` so
  anchors land below the fixed bar.
- Wide content scrolls inside its own container; the body never scrolls sideways.
- `prefers-reduced-motion` strips transforms and transitions.

Contrast, checked: accent on background 6.8:1, white on accent 7.2:1, body
14.4:1, muted 5.7:1, khaki labels 4.6:1 on their own tint. All clear WCAG AA.

---

## 7. Deployment

`.github/workflows/deploy.yml`: checkout → Hugo extended → `npm ci` → build with
`--baseURL` from `actions/configure-pages` → `npx pagefind --site public` →
upload → deploy. Pagefind must run **after** Hugo and against `public/`.

`_vendor/` and `go.sum` are both committed. Without `go.sum`, module verification
fails in CI.

**Not yet done:** the repository has no remote push configured on the author's
machine, and GitHub Pages has not been enabled. Pages must be set to build from
**GitHub Actions**, not from a branch.

---

## 8. Editorial rules that were applied

- **Hero intro and About are deliberately different text.** Never paste one into
  the other.
- **No process commentary ships.** Nothing on the page says where content came
  from. Notes to the owner are HTML comments, searchable as
  `NOTE FOR THE SITE OWNER`.
- **No filler subtitles.** A section gets a useful one-liner or none. Research
  and Contact have none by the owner's choice.
- **Her wording is preserved**, including "political behaviors" over the more
  usual singular, and the hashtag spellings.
- **Understated tone.** Awards stated flatly, once, no intensifiers.

---

## 9. What is deliberately absent

Removed at the owner's direction over successive passes. Each is recoverable from
git history; none should be reinstated without asking.

| Gone | Notes |
|---|---|
| Teaching section | courses and advising remain on the C.V. |
| Presentations | 46 invited talks and conference papers |
| Writings page | the filterable multi-tab index; hashtags replaced it |
| Per-publication pages | citations link straight to the publisher |
| People / collaborators | 14 co-author cards, and a closing invitation line |
| Site footer | copyright line went with it |
| Appointments, grants, honours | folded into the C.V. PDF only |
| Book review, 2 editor-reviewed pieces, World Bank report, both theses | "articles and book chapters" only |
| "See also" cross-linking | had no per-paper page left to sit on |
| Taxonomies and permalinks | nowhere for a term page to lead |

---

## 10. Known gaps

- **No cover images.** Journal and book covers were meant to be fetched at build
  time; Cambridge, books.com.tw and eslite all serve bot-blocked pages to
  automated requests. Republishing publisher cover art is the owner's call anyway.
- **Four 2026 op-eds have no URL** — the C.V. printed the word "Link" rather than
  an address. Flagged in each file.
- **`reform-without-transformation`** is on the legacy site but not in the C.V.,
  and has no DOI or link, so its title is not clickable. Its citation details are
  flagged for confirmation.
- **Two hashtags are very broad.** `PoliticalBehavior` and `TaiwanPolitics` each
  cover about half the corpus, which limits their value as filters.
- **PDF self-archiving is unverified.** The hosted PDFs are publisher-typeset
  versions. Most journals permit author self-archiving; a few restrict it to
  accepted manuscripts. Worth checking against the agreements before launch.
