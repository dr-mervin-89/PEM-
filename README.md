# PEM Calc — deployment package

A standalone, offline-capable paediatric emergency drug calculator. No login, no
backend, no build step — just static files.

## Files
- `index.html` — the entire app (HTML/CSS/JS in one file).
- `manifest.json` — PWA manifest so the page can be "Added to Home Screen" as an app icon.
- `sw.js` — cache-first service worker; after the first visit, the app keeps working fully offline.
- `icon-192.png`, `icon-512.png`, `apple-touch-icon.png` — app icons.

## Deploy in 2 minutes

### GitHub Pages
1. Create a new repo and push all files in this folder to it (root of the repo, or a `/docs` folder).
2. Repo → Settings → Pages → Source: deploy from branch → select the branch/folder above → Save.
3. GitHub gives you a URL like `https://<username>.github.io/<repo>/` — that's your shareable link.

### Netlify
1. Go to https://app.netlify.com/drop and drag this whole folder onto the page.
2. Netlify gives you a live URL immediately. Bookmark it or set a custom subdomain in site settings.

### Vercel
1. `npm i -g vercel` (or use the Vercel dashboard's "Add New Project" → drag & drop / upload folder).
2. `vercel --prod` from inside this folder, or upload via the dashboard.
3. Vercel gives you a live URL.

Any of these produce a plain URL that opens directly in a browser, works on phones/tablets/desktops,
needs no sign-in, and can be bookmarked and reopened later. Once opened once, the service worker
caches the app shell so it keeps working with no signal/wifi.

## Adding a new section later

The app uses a `registerModule({ id, label, color, buildInputs, render })` pattern in the `<script>`
block of `index.html`. Each clinical section is its own call to `registerModule(...)` with its own
drug list / logic — nothing shared, nothing to rewire. To add a new section:

1. Copy the shape of an existing `registerModule({...})` block.
2. Give it a new `id` (unique), a `label` (tab text), and a `color` (any hex, used for its tab dot
   and left-border accent — add a matching `--c-yourmodule` CSS variable near the top of `<style>`
   if you want it in the shared palette, or just inline a hex).
3. Write `render()` to return the HTML for that section's cards using the existing `drugCard({...})`
   helper for straightforward per-kg drug doses (handles bold values, formula-in-brackets, max/min
   clamping, and mg→mL conversion via `concMgPerMl` automatically), or plain HTML for tables/reference
   content.
4. If the section needs its own inputs (a toggle, a percentage box, observed values), add a
   `buildInputs(container)` function — it runs once when the tab is first opened and should attach
   its own `input`/`click` listeners that call `updateOutputs('your-id')`. This keeps typed input focus
   stable (no Android reverse-typing bug) because inputs are never re-created after their first build —
   only the `.module-outputs` div is refreshed on every recalculation.
5. That's it — nothing in any other module, in the patient bar, or in the core app shell needs to change.

Existing sections don't need to be touched to add a new one — each is fully independent, as required.

## Bumping the cache after edits

If you edit `index.html` after deploying, increment `CACHE_NAME` in `sw.js` (e.g. `pem-calc-v2`) so
returning users' browsers pick up the new version instead of serving the old cached copy.
