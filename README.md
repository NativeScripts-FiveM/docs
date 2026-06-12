# Native Scripts — FiveM docs

VitePress documentation site for the **FiveM (GTA5)** side of Native Scripts.
Lives at **https://nativescripts-fivem.github.io/docs/**.

The RedM (RDR3) docs are a separate site: https://nativescripts-redm.github.io/docs/

## Structure

```
.
├── index.md                       ← homepage
├── guide/fivem/                   ← getting-started + concepts
├── scripts/fivem/                 ← one .md per script (synced from each script README)
├── docs-tree.json                 ← file manifest (store panels read this from raw CDN)
├── .vitepress/config.ts           ← site config (nav, sidebar, theme)
└── .github/workflows/             ← Pages deploy + docs-tree regeneration
```

Each script page is the script's `README.md`, synced via the `/sync-docs` skill — don't edit script pages here directly, they're overwritten on the next sync.

## Local dev

```bash
npm install
npm run docs:dev      # http://localhost:5173/docs/
npm run docs:build    # production build into .vitepress/dist
```

Pushing to `main` auto-deploys to GitHub Pages (Actions).
