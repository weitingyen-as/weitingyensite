# weitingyen-as.github.io/weitingyensite

The personal academic website of Wei-Ting Yen, Institute of Political Science,
Academia Sinica.

- **To add a paper, a talk, or a person, or to update the bio:** read
  [UPDATING.md](UPDATING.md). No software needs to be installed; everything is
  edited in the browser on GitHub, and the site republishes itself.
- **To understand how the site is built:** read
  [WEBSITE_PRINCIPLES.md](WEBSITE_PRINCIPLES.md).

Built with Hugo (extended) and Hugo Blox, searched with Pagefind, published to
GitHub Pages by `.github/workflows/deploy.yml` on every push to `main`.

## Building locally (optional)

Requires Hugo extended ≥ 0.148.2 and Node 20+.

```bash
npm ci
hugo server
```

For a production build with a working search index:

```bash
hugo --gc --minify
npx pagefind --site public
```
