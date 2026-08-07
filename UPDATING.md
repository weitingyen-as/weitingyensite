# Updating this website

The site is **one page**. Everything is on it, stacked in this order:

**Hero** (photo, name, intro, C.V. and email buttons) → **About** → **Research**
→ **Public Writing** → **Contact**

It rebuilds and republishes itself every time you save a change on GitHub. You
never need to run anything on your own computer. Give it two or three minutes,
then reload <https://weitingyen-as.github.io/weitingyensite/>.

Everything below is done in your browser at
<https://github.com/weitingyen-as/weitingyensite>.

---

## Two rules that matter

1. **Never put a leading slash on a file link.** Write `files/paper.pdf`, not
   `/files/paper.pdf`. The leading slash breaks the link on GitHub Pages.
2. **Notes to yourself go in HTML comments.** Anything written as
   `<!-- like this -->` is invisible to visitors. A few such notes are already in
   the files flagging things worth checking — search the repository for
   `NOTE FOR THE SITE OWNER` to find them.

Folder names no longer appear in any web address, so renaming a folder is safe.

---

## Where everything lives

| What you want to change | File |
|---|---|
| Photo, name, job title, intro paragraph | `content/_index.md` |
| The About narrative | `content/bio/_index.md` |
| A publication, or its hashtags | `content/publication/<folder>/index.md` |
| Who Governs Taiwan, the Mandarin post list | `content/public-engagement/_index.md` |
| Email and address | `content/contact/_index.md` |
| Menu items, site title | `hugo.yaml` |
| Colours, spacing, card styling | `assets/css/custom.css` |
| The C.V. PDF | `static/files/wei-ting-yen-cv.pdf` |
| Your portrait | `static/images/wei-ting-yen.jpg` |

---

## Add a publication

Journal articles and book chapters appear in **Research**, as cards, newest
first, filtered by hashtag.

1. Upload the PDF to `static/files/` (**Add file → Upload files**). Give it a
   plain, permanent name like `2027_journal-name_short-title.pdf`.
2. Go to `content/publication/`, click **Add file → Create new file**, and type
   a filename of the form `short-slug/index.md` — the slash creates the folder.
3. Paste this in and edit it:

```yaml
---
title: "The Full Title of the Paper"
date: 2027-03-01
authors: ["Wei-Ting Yen", "Co Author"]
publication_types: ["journal_article"]
publication: "Journal Name, 12(3): 45–67"
venue: "Journal Name"
venue_detail: "12(3): 45–67"
abstract: "Two or three sentences describing what the paper shows."
links:
  - name: "Article (PDF)"
    url: "files/2027_journal-name_short-title.pdf"
  - name: "Publisher's Version"
    url: "https://doi.org/10.1234/example"
doi: "10.1234/example"
hashtags: ["WelfareState", "TaiwanPolitics"]
---
```

4. Click **Commit changes**.

### The fields, one at a time

**`date`** puts the card in order. Only the year is displayed. If two papers
share a year, the later date sorts first — so use `2027-01-01`, `2027-01-02`
and so on to control the order within a year.

**`publication_types`** decides which part of the page the item lands on:

| Value | Where it appears |
|---|---|
| `journal_article` | Research |
| `book_chapter` | Research |
| `op_ed` | Public Writing, under "Commentary and op-eds" |
| `edited_volume` | Public Writing, as a card at the top |

**`venue` and `venue_detail`** are what make the citation line read properly.
`venue` is the journal or book title — it prints in the Academia Sinica blue.
`venue_detail` is everything after it. **Don't put the year in `venue_detail`**;
it already appears above the title.

For a **book chapter**, add `venue_prefix` for the editors:

```yaml
publication_types: ["book_chapter"]
publication: "In A. Editor (ed), Book Title, Chapter 4, 55–70. City: Publisher"
venue_prefix: "In A. Editor (ed), "
venue: "Book Title"
venue_detail: "Chapter 4, 55–70. City: Publisher"
```

`publication` is the full citation as one string. It is a fallback, used only if
`venue` is missing — keep it accurate but it is `venue`/`venue_detail` that show.

**`links`** become the small links in the card footer. The one named
`Publisher's Version` is special: the **title links to it**, and it does not get
its own footer link. Everything else appears in the footer. Name links for what
they are — "Article (PDF)", "Online Appendix", "Replication Data" — never "click
here". External links open in a new tab automatically.

**`abstract`** is stored but **not currently displayed**. The cards were made
compact so the list stays readable as it grows. The text is kept so it can be
switched back on.

**`award`** prints a gold badge under the citation:

```yaml
award: "S.C. Lee Graduate Research Paper Award, Michigan State University."
```

---

## Hashtags

Hashtags are the only way a reader filters your work, so they matter.

- They are defined **only** in each publication's `hashtags:` list. There is no
  separate list to maintain.
- **To add a new hashtag**, just use it on a paper. It appears in the filter bar
  by itself, with its count.
- **To rename one**, change it on every paper that carries it. Miss one and you
  get two chips.
- **To retire one**, remove it from every paper. The chip disappears.
- **Write them exactly**, in CamelCase with no spaces or `#`: `TaiwanPolitics`,
  not `Taiwan Politics` or `#taiwanpolitics`. They are case-sensitive.
- The chip bar orders itself by how many papers carry each tag, most first.

The 14 in use, in the order the bar shows them (count in brackets):

`TaiwanPolitics` 11 · `PoliticalBehavior` 10 · `COVID` 9 · `Identity` 5 · `EconomicInsecurity` 4 · `Framing` 4 · `DevelopmentalState` 3 · `SocialInsurance` 3 · `StateCapacity` 3 · `WelfareState` 3 · `IssueSecuritization` 2 · `Populism` 1 · `Teaching` 1 · `UniversalBasicIncome` 1

Selecting more than one hashtag **narrows** — a paper must carry all of them.
Tags that would leave nothing to show are greyed out rather than leading to an
empty list.

**A note on balance:** `PoliticalBehavior` and `TaiwanPolitics` are each on about
half your papers. That works as a label but barely narrows anything as a filter.
Worth watching as the list grows.

---

## Add an op-ed or a piece of commentary

Same as a publication, in `content/publication/`, with `publication_types:
["op_ed"]`. These need no `venue` fields and take no hashtags:

```yaml
---
title: "Title of the Piece"
date: 2027-04-18
authors: ["Wei-Ting Yen"]
publication_types: ["op_ed"]
publication: "Foreign Affairs"
links:
  - name: "Read at Foreign Affairs"
    url: "https://www.foreignaffairs.com/..."
---
```

The whole entry links out to the outlet.

---

## Edit the written sections

**About**, **Public Writing** and **Contact** are ordinary Markdown files. Edit
them the way you would a document — headings with `##`, links as
`[text](https://url)`.

To fold a long block away behind a click, wrap it like this — the Mandarin post
list already uses it:

```html
<details class="fold">
<summary>What the click-to-open label says</summary>

...your Markdown, with a blank line above and below...

</details>
```

**The hero intro** is the `intro:` block in `content/_index.md`. Keep it to two
to four sentences. It is deliberately different from the About narrative — don't
let the two say the same thing.

---

## Replace the C.V.

Upload the new file over `static/files/wei-ting-yen-cv.pdf`, keeping that exact
filename. The button in the hero then points at the new one. It opens in a new
tab.

## Replace the photo

Upload over `static/images/wei-ting-yen.jpg`. Square, about 720×720, under
150 KB. It is displayed as a 200px circle.

---

## When something looks wrong

Go to the **Actions** tab in the repository. Each save starts a run. A green
tick means the site published; a red cross means the build stopped and the
**previous version of the site is still live, unharmed**.

Click the red run to see what happened. Almost always it is one of these:

- A missing quotation mark, or a stray `:` inside a value, in the block between
  the `---` lines. Values with a colon in them must be quoted.
- A date that isn't `YYYY-MM-DD`.
- An indentation slip under `links:` — the `- name:` and `url:` lines must line
  up with the ones above them.

Fix the file and commit again; the site republishes itself.

**If a change seems to have no effect**, do a hard reload (Cmd-Shift-R). If it
still looks unchanged, check the Actions tab — the build may have failed.

---

## Things that happen on their own

- **The hashtag filter bar**, including which chips exist and their counts.
- **Ordering** of publications, newest first.
- **"Show all N publications"** — the list shows six until asked, and the button
  counts whatever the current filter leaves.
- **Search** (the magnifying glass, or Cmd-K), re-indexed on every publish.
- **Deployment** on every save.
