# Website architecture playbook

Written for whoever — person or AI assistant — maintains this site next. It records
what the pieces are, why they were chosen, and which decisions are load-bearing.

The owner-facing instructions are in [UPDATING.md](UPDATING.md). This file is the
engineering side.

---

## 1. Stack

| Layer | Choice |
|---|---|
| Generator | Hugo **extended**, ≥ 0.148.2 (CI pins 0.164.0) |
| Theme module | `github.com/HugoBlox/hugo-blox-builder/modules/blox-tailwind` v0.10.0, vendored into `_vendor/` |
| Styling | One hand-written stylesheet, `assets/css/custom.css` |
| Search | Pagefind 1.5, built in CI after Hugo, indexed against `public/` |
| Hosting | GitHub Pages project site at `/weitingyensite/` |
| CI | `.github/workflows/deploy.yml` — build → index → deploy on every push to `main` |

**Hugo extended is required** and the version floor is real: Blox v0.10.0 refuses
to load on anything below 0.148.2. The build instructions this site was made from
suggested 0.147.0; that does not work.

**Hugo ≥ 0.146 template layout.** Templates live at `layouts/single.html`,
`layouts/list.html`, `layouts/_partials/…`, `layouts/_shortcodes/…` — *not* the old
`layouts/_default/` and `layouts/partials/`. Blox itself uses `layouts/_partials/`;
putting an override in `layouts/partials/` fails silently.

---

## 2. The one deliberate departure from stock Blox

**This site does not use Blox's Tailwind CSS pipeline or its page templates.**
`layouts/baseof.html` is a complete replacement shell that loads
`assets/css/custom.css` directly and never calls Blox's `_partials/css.html`.

Why: the owner chose <https://yawenlei.com/> as the visual model, and the
instruction that a chosen reference design takes precedence over default styling is
the strongest one in the brief. Matching that design — serif display headings on a
warm off-white ground, a fixed slim navbar, a single institutional accent — is far
more reliably done with ~500 lines of plain CSS than by fighting Tailwind utility
classes emitted by someone else's blocks.

What this buys: no Tailwind CLI needed at build time, no Blox preflight overriding
the palette, total control of the markup. What it costs: Blox partials that assume
theme CSS classes will look unstyled if dropped in.

The module is still imported and vendored, and Blox's helper partials are still
available and used — `functions/get_featured_image.html` is called by
`layouts/publication/single.html`. If you ever wire up a Blox *visual* component,
you will need to include `{{ partial "css.html" . }}` in `baseof.html` and keep
`@tailwindcss/cli` in `package.json` (it is already there for exactly this reason).

---

## 3. URLs are a contract

Academic pages get cited in work that outlives the website. Two rules protect that:

- **`permalinks` uses `:contentbasename`, not `:slug`.** `:slug` falls back to the
  *title*, so editing a title would silently move the page and break every existing
  citation — and it collided outright here, because two talks share the title
  "Territorial State Identity and Solidarity: An Experimental Approach". With
  `:contentbasename` the folder name fixes the address permanently.
- **Never rename a content folder.** Change `title:` freely; leave the directory.

### Subpath safety

This is a *project* Pages site served from `/weitingyensite/`, so a literal
`/files/foo.pdf` resolves to the host root and 404s.

- In templates: `{{ "pagefind/pagefind.js" | relURL }}` — **no leading slash**.
  `relURL "/x"` returns `/x` and escapes the base path; `relURL "x"` returns
  `/weitingyensite/x`.
- In front matter `links:`: write `url: "files/paper.pdf"`. The single template
  passes anything not starting with `http` through `relURL`.
- In Markdown body text: `{{</* staticrel "files/paper.pdf" */>}}`.

### Two Go-template escaping traps that bit this build

Both produced *silently broken* JavaScript, not build errors:

1. `{{ $x | jsonify }}` inside a `<script>` gets Go's JS-context escaping applied
   **on top of** jsonify's quoting, yielding `"\"/pagefind/pagefind.js\""` — a
   string containing a quoted string. For a plain string, interpolate inside
   quotes instead: `var URL = "{{ "pagefind/pagefind.js" | relURL }}";`
2. For a JSON payload in `<script type="application/json">`, the same doubling
   turns the object into a JSON *string*. Use `{{ $data | jsonify | safeJS }}`.

---

## 4. Content model

Every item is a **leaf bundle**: a directory containing `index.md`, with its PDFs
in `static/files/` and its cover at `featured.jpg` beside `index.md`.

```
content/
  _index.md              type: landing — hero copy + photo live in front matter
  publication/           papers, chapters, books, reviews, reports, theses, op-eds
  talk/                  invited talks, panels, conference papers
  software/              empty; layouts exist so the section works if ever used
  authors/               one folder per co-author and advisee — this is /People/
  bio/  teaching/  public-engagement/  contact/
data/
  research_areas.json          area names, blurbs, and the tags that belong to each
  writings_legacy_map.json     optional per-slug tab override; currently empty
  featured_publications.yaml   optional homepage spotlight; currently empty
```

`publication_types` is a controlled vocabulary: `journal_article`, `book_chapter`,
`book`, `review`, `report`, `thesis`, `op_ed`, `presentation`. It drives Writings-page
tabs and "See also" labels. **It is never printed on an item's own page** — a reader
looking at a paper does not need to be told it is a `journal_article`.

`op_ed` is not in the upstream vocabulary; it was added because her commentary for
*Foreign Affairs*, the *Washington Post*, and Brookings is a substantial body of work
that belongs on the Writings page but must not be mixed in with refereed articles.

### Taxonomies

Only `tags` generates term pages. `categories` was unused, and `publication_types`
term pages would have surfaced bare `journal_article` pages to readers. Tag pages
also carry no `data-pagefind-body`, so search returns papers rather than keywords.

---

## 5. "See also" — `layouts/_partials/related_finder.html`

Runs at build time on every publication/talk/software page. Nothing is cached to
disk, so adding an item re-links the whole site on the next build. **Do not replace
this with a script that writes a `related_map.json`** — that file goes stale the
first time someone adds a paper and forgets to re-run it.

Priority order:

1. Explicit front matter: `related_papers`, `related_talks`, `related_software`,
   `related_datasets`, plus the singular legacy `related_paper` / `related_dataset`.
2. `see_also:` entries — `{url, title}` external, `{section, slug}` internal.
3. `dataverse_url`, or a Harvard Dataverse URL found in the abstract.
4. Research-area siblings from `data/research_areas.json`.
5. Scored backfill: **+2** per shared title token (stop words stripped, tokens under
   3 characters dropped), **+1** per shared author (exact, or last-name match so
   "Jonathan Katz" and "Jonathan N. Katz" count), **+2** per shared tag, **+1** for a
   tag in the same research area. Threshold 4, relaxed to 2 when fewer than three
   explicit picks exist.

Capped at 8, sorted score then year. Titles are normalised to lowercase alphanumerics
for deduplication. Labels come from `publication_types` first and the section name
only as a fallback, since an item under `/publication/` may be a book or a dataset.
Response/reply/comment papers pin their subject at the top; book editions sharing a
base title cross-link.

---

## 6. The Writings page — `layouts/publication/list.html`

The most important page on the site, and the one with the most ways to go wrong.

- **Every entry is server-rendered into the DOM.** JavaScript filters by toggling
  `hidden`; it never creates content. With JS off the full list still reads.
- **Every `<article class="pub-item">` carries `data-tab`, `data-year`,
  `data-areas`, `data-rank`, `data-title`, `data-haystack`.** Every
  `<button class="tab-btn">` carries a matching `data-tab` and has a real click
  listener. Tabs that render but do nothing is the classic failure here — if you
  touch this file, click every tab afterwards and watch the count change.
- **"Newest first" sorts by year, then by type rank** (articles → chapters → books →
  reviews → reports → theses → op-eds). Plain date ordering put four commentary
  pieces above her journal articles at the top of the page, which misrepresents the
  work. The server-side render order matches the JS sort so nothing jumps on load.
- **Filter state lives in `location.hash`** (`#articles&area=…&year=…&q=…&sort=…`),
  so a filtered view is shareable and the back button works. `hashchange` re-reads it.
- **BibTeX export** serialises the visible rows from a JSON block rendered from the
  same front matter, and picks the right field name per entry type (`journal`,
  `booktitle`, `school`, `institution`, `publisher`). It keys off the folder name in
  each row's link — another reason the permalink must be `:contentbasename`.

---

## 7. Search — `layouts/_partials/search_modal.html`

Pagefind, loaded as a modal overlay, `Cmd-K` / `Ctrl-K` to open, `Esc` to close,
arrow keys through results. The ~200 KB WASM bundle is imported on first *open*,
not on page load.

The import is memoised as **one shared promise**. An earlier version used a
`loading` boolean and returned `null` to any call that arrived mid-flight, so the
first keystroke after opening the modal always fell through to the "search
unavailable" branch. If you refactor this, keep the promise.

If Pagefind cannot load, the modal falls back to a Google `site:` search rather than
failing silently.

---

## 8. Design system

`assets/css/custom.css` holds the whole visual language as custom properties. The
palette derives from the reference site's structure with Harvard crimson replaced by
a desaturated slate drawn from Academia Sinica:

| Token | Value | Use |
|---|---|---|
| `--color-bg` | `#faf8f5` | page — warm off-white, never `#fff` |
| `--color-surface` | `#ffffff` | cards, filter panel, search modal |
| `--color-text` | `#23201d` | body — warm charcoal, never pure black |
| `--color-text-muted` | `#6b645d` | metadata, author lines |
| `--color-accent` / `--color-institution` | `#3d5a6c` | active tab rules, buttons, links |
| `--color-link-hover` | `#8a5a2b` | warm umber on hover |
| `--color-gold` | `#9a7238` | awards only |
| `--color-border` | `#e6e1da` | 1px hairlines |
| `--color-focus` | `#d4a96a` | `:focus-visible` outline |

Colours mean things: accent = institution and interaction, gold = an award, muted
grey = metadata. Don't reuse them for anything else.

**Typography is a native system stack by design** — a serif family for headings
(`Iowan Old Style`, Palatino, Georgia) and the OS sans for body. The reference site
loads Inter and Noto Serif Display from Google Fonts; this site does not, because
nothing required to render the site should depend on a third-party host that can go
dark or start logging visitors. The typographic *structure* — serif display headings
over a sans body — is preserved.

**Light mode is forced.** `.dark` is overridden back to light surfaces and the theme
toggle is hidden.

Layout invariants worth not breaking:

- Homepage hero is **photo left, text right** on desktop; stacked and centred at
  ≤640px. Not a centred avatar above the name — that reads as a social profile.
- The navbar brand is her full name, uppercase, far left, always linking home.
  Nav links flush right. Search is a bare magnifying glass, no label or border.
  Header horizontal padding matches the content gutter.
- `#research-areas` sits on the section *heading* with
  `scroll-margin-top: calc(var(--nav-h) + 16px)`, so the nav anchor lands just below
  the fixed bar instead of at the bottom of the page.
- Year-filter rows are a flex row: `[checkbox] [label]`, pinned together.
- Wide content scrolls inside its own container; the page body never scrolls
  sideways.
- `prefers-reduced-motion` strips transforms and transitions.

---

## 9. Editorial rules that were applied

- **The homepage intro and the bio page are deliberately different text.** Never
  paste one into the other.
- **No process commentary ships.** Nothing on any rendered page says where content
  came from or how the site was built. Notes to the owner are HTML comments,
  searchable as `NOTE FOR THE SITE OWNER`.
- **No filler subtitles.** A section either gets a genuinely useful one-liner in her
  voice or no subtitle at all.
- **Her wording is preserved.** Where the legacy site and the C.V. disagreed, the
  C.V. won (it is newer) and the discrepancy was flagged in a comment rather than
  silently resolved — see `content/publication/reform-without-transformation/`.
- **Understated tone.** Awards are stated flatly, once, with no intensifiers.

---

## 10. Known gaps

- **No `featured.jpg` cover images.** Journal and book covers were meant to be
  fetched during the build; the publishers involved (Cambridge, books.com.tw, eslite)
  serve bot-blocked pages to automated requests. Layouts handle their absence
  gracefully. Republishing publisher cover art is also the owner's call to make.
- **The People page links are unverified guesses** from web search, and eight
  collaborators have no link at all. Flagged in `UPDATING.md` and in per-file
  comments.
- **Four 2026 op-eds have no URL** because the C.V. printed the word "Link" rather
  than the address. Flagged in each file.
- **`content/software/` is empty.** The layouts exist and the section is absent from
  the nav; drop a leaf bundle in and add a menu entry if that changes.
