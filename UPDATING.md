# Updating this website

This site rebuilds and republishes itself every time you save a change on GitHub.
You never need to run anything on your own computer. Give it two or three minutes
after saving, then reload <https://weitingyen-as.github.io/weitingyensite/>.

Everything below is done in your browser at
<https://github.com/weitingyen-as/weitingyensite>.

---

## Before you start: read this once

**Please review the People page.** The affiliations and external links on
<https://weitingyen-as.github.io/weitingyensite/authors/> were assembled
automatically: your co-authors were collected from the `authors:` lines on your
papers and talks, and each person's link was found by web searching their name.
**Nobody has checked these.** Name searches collide and university pages go stale,
so it is possible that a link points at the wrong person entirely. Go through every
card once and fix or delete anything wrong. Eight collaborators have no link at all
because no page could be confidently matched — each of those files contains an HTML
comment saying so.

**Two folder rules that matter forever.**

1. **Never rename a folder** under `content/publication/`, `content/talk/`, or
   `content/authors/` once the site is live. The folder name *is* the web address,
   and addresses get cited in other people's papers. If a title changes, edit the
   `title:` line inside the file and leave the folder alone.
2. **Never put a leading slash on a file link.** Write `files/paper.pdf`, not
   `/files/paper.pdf`. The leading slash breaks the link on GitHub Pages.

**Notes to yourself go in HTML comments.** Anything written as
`<!-- like this -->` is invisible to visitors. Several such notes are already in
the site flagging things worth your attention — search the repository for
`NOTE FOR THE SITE OWNER` to find them.

---

## Add a paper

1. Upload the PDF to the `static/files/` folder
   (**Add file → Upload files**). Name it something plain and permanent, like
   `2027_journal-name_short-title.pdf`.
2. Go to `content/publication/`, click **Add file → Create new file**, and type a
   filename of the form `short-slug/index.md` — the slash creates the folder. The
   slug becomes the web address, so keep it short, lowercase, and hyphenated.
3. Paste this in and edit it:

```yaml
---
title: "The Full Title of the Paper"
date: 2027-03-01
authors: ["Wei-Ting Yen", "Co Author"]
publication_types: ["journal_article"]
publication: "Journal Name, 12(3): 45–67"
abstract: "Two or three sentences describing what the paper shows."
links:
  - name: "Article (PDF)"
    url: "files/2027_journal-name_short-title.pdf"
  - name: "Publisher's Version"
    url: "https://doi.org/10.1234/example"
doi: "10.1234/example"
tags: ["welfare state", "taiwan"]
---
```

4. Click **Commit changes**.

**`publication_types`** decides which tab the paper lands on:

| Value | Tab |
|---|---|
| `journal_article` | Articles |
| `book_chapter` | Chapters |
| `book` | Books |
| `review` | Reviews |
| `report` | Reports |
| `thesis` | Theses |
| `op_ed` | Op-Eds |

**`tags`** decide which research area the paper appears under on the homepage, and
they feed the "See also" links at the bottom of every page. Use tags that already
exist elsewhere on the site — you can see the full list at
<https://weitingyen-as.github.io/weitingyensite/tags/>. A tag that matches one of
the `subcategories` in `data/research_areas.json` puts the paper in that area.

You do not have to do anything to make the paper show up on the homepage, in
search, or in other papers' "See also" lists. That all happens on its own.

### Optional extras

- **A prize:** add `award: "S.C. Lee Best Paper Award, Michigan State University."`
  and it appears as a small gold badge.
- **A cover image:** upload a file named `featured.jpg` into the same folder as
  `index.md` and it is picked up automatically.
- **More links:** add as many `- name:` / `url:` pairs as you like. Name them for
  what they are — "Online Appendix", "Replication Data" — never "click here".
- **Force a "See also" link:** add `related_papers: ["other-folder-name"]`.

---

## Add a talk

Same idea, in `content/talk/`. Create `some-slug/index.md`:

```yaml
---
title: "Title of the Talk"
date: 2027-05-14
authors: ["Wei-Ting Yen"]
publication_types: ["presentation"]
event: "Department of Political Science, Some University"
location: "Taipei, Taiwan"
talk_role: "invited"
talk_role_label: "Invited talk"
publication: "Department of Political Science, Some University"
tags: ["welfare state"]
---
```

`talk_role_label` is the small tag shown next to the title. Use `Invited talk`,
`Panel`, or `Conference paper`.

---

## Add or fix a person

Each person has a folder under `content/authors/`. To add one, create
`their-name/index.md`:

```yaml
---
title: "Their Name"
superuser: false
user_groups: ["Collaborators"]
role: "Assistant Professor of Political Science, Some University"
website: "https://their-page.example.edu"
---
```

- `user_groups` is either `["Collaborators"]` or `["Students Advised"]`.
- Leave `website` out entirely if you don't have a good link — the card just
  shows their name and affiliation.
- Add a photo by uploading `avatar.jpg` into the same folder. Without one they get
  a neat monogram of their initials, which is a perfectly good default.

Names on your papers link to these pages automatically when the spelling in the
paper's `authors:` list matches the person's `title:` exactly. Diacritics and
middle initials count.

---

## Update your bio or C.V.

- **The bio page:** edit `content/bio/_index.md`. It is ordinary text with a few
  simple tables; edit it the way you would a document.
- **The C.V. PDF:** upload the new file over `static/files/wei-ting-yen-cv.pdf`,
  keeping that exact filename. Every link to it then updates at once.
- **The homepage introduction:** edit the `intro:` block in `content/_index.md`.
  Keep it to two to four sentences — it is deliberately different from the bio
  page, and the two should never say the same thing twice.

---

## Other pages

| What you want to change | File to edit |
|---|---|
| Homepage intro, job title, photo | `content/_index.md` |
| Bio, appointments, grants, honours | `content/bio/_index.md` |
| Courses and advising | `content/teaching/_index.md` |
| Who Governs Taiwan, public talks | `content/public-engagement/_index.md` |
| Email and address | `content/contact/_index.md` |
| Research area names and blurbs | `data/research_areas.json` |
| Menu order, site title | `hugo.yaml` |
| Button wording used across the site | `i18n/en.yaml` |

---

## When something looks wrong

Go to the **Actions** tab in the repository. Each save starts a run. A green tick
means the site published; a red cross means the build stopped and the *previous*
version of the site is still live, unharmed.

Click the red run to see what happened. Almost always it is one of these:

- A missing quotation mark or a stray `:` in the block between the `---` lines.
- A date that isn't in `YYYY-MM-DD` form.
- Two folders with the same name.

Fix the file and commit again; the site republishes itself.

---

## Things that are automatic — don't maintain them by hand

- **"See also"** at the bottom of every paper and talk. It is computed at build
  time from shared authors, shared tags, and overlapping title words. Add a paper
  and the rest of the site links to it on its own.
- **The homepage research areas**, including the item counts.
- **Search.** The index is rebuilt on every publish.
- **The Writings page tabs, filters, sorting, and citation download.**
- **Cover images**, if a `featured.jpg` sits next to an `index.md`.
