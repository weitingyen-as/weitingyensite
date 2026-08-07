# weitingyen-as.github.io/weitingyensite

The personal academic website of Wei-Ting Yen, Institute of Political Science,
Academia Sinica. A single page: hero, About, Research, Public Writing, Contact.

- **To add a publication, change hashtags, or edit any section:** read
  [UPDATING.md](UPDATING.md). Nothing needs installing — it is all done in the
  browser on GitHub, and the site republishes itself.
- **To understand how the site is built:** read
  [WEBSITE_PRINCIPLES.md](WEBSITE_PRINCIPLES.md).

Built with Hugo (extended) and a vendored Hugo Blox module, searched with
Pagefind, published to GitHub Pages by `.github/workflows/deploy.yml` on every
push to `main`.

## Building locally (optional)

Requires Hugo extended ≥ 0.148.2 and Node 20+.

```bash
npm ci
hugo server
```

For a production build with a working search index — stop the dev server first,
or it will overwrite `public/` with localhost URLs:

```bash
hugo --gc --minify
npx pagefind --site public
```
