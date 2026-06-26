# Adding a Tool to app.preritpramod.com

The hub repo (`preritpramod-io/app-hub`) serves `app.preritpramod.com` and routes
sub-apps through `vercel.json`. Deploys happen automatically on push to `main`.
Never use the Vercel UI.

## Pattern A — Self-contained tool (recommended for single-page tools)

The tool is one self-contained file, reached by a catch-all rewrite. Used by
`/study/astrodynamics` and `/tools/valuation`.

1. Add `tools/<name>/index.html`. Inline everything: CDN libraries, CSS, and JS.
   Reference shared assets from `/brand/...` or `/public/...`, never from a path
   under `tools/<name>/` (the catch-all below would intercept it).
2. Append three rewrites to `vercel.json`:

       { "source": "/tools/<name>",      "destination": "/tools/<name>/index.html" }
       { "source": "/tools/<name>/",     "destination": "/tools/<name>/index.html" }
       { "source": "/tools/<name>/(.*)", "destination": "/tools/<name>/index.html" }

3. Commit to `main`. Live at `app.preritpramod.com/tools/<name>`.

## Pattern B — Full app (its own Vercel project)

For multi-file apps. Deploy the app as its own Vercel project that serves under the
base path `/tools/<name>`, then proxy from the hub (as `/activities/defence/india` does):

       { "source": "/tools/<name>",      "destination": "https://<project>.vercel.app/tools/<name>" }
       { "source": "/tools/<name>/",     "destination": "https://<project>.vercel.app/tools/<name>/" }
       { "source": "/tools/<name>/(.*)", "destination": "https://<project>.vercel.app/tools/<name>/$1" }

The sub-app must serve its content under that exact base path.

## Brand assets

Hosted under `public/brand/<entity>/`, served at `app.preritpramod.com/brand/<entity>/...`.

- Vector (`.svg`) assets commit reliably from any surface (they are text).
- Binary assets (`.png`, `.jpg`) must be pushed from Cowork or Claude Code, where git
  handles binaries natively. Do not hand-encode base64 binary through a chat tool call;
  large encodes corrupt mid-string.

## Conventions

- Project names are kebab-case; no internal project codes in public names.
- The catch-all routes every sub-path to `index.html`, so a Pattern-A tool must not
  request its own sub-path assets; inline them, or host under `/public`.
- Secrets live in environment variables, never in the repo.
