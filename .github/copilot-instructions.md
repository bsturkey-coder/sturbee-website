# Copilot instructions for Sturbee Financial Website

## Project snapshot
- Simple static website (no build system): top-level HTML files: `index.html`, `budget-dashboard.html`, `refund-policy.html`, `thankyou.html` and a single CSS file embedded in each page.
- Assets: `images/` (logos/photos) and `downloads/` (PDF guide).
- Minimal JS: a tiny inline script in `index.html` to set the copyright year.

## Goals for AI agents (quick checklist)
- Make safe, minimal edits that preserve site styles and relative links.
- Prefer small, focused PRs that change only necessary HTML/CSS and list files changed.
- When updating shared visual tokens (colors/typography), update every file that defines `:root` variables (see examples below).

## Key patterns and examples
- Styles are embedded per-page. To change a variable used across site, update `--accent` and other `:root` vars found in `index.html` and `budget-dashboard.html`.
  - Example: change brand gold in `index.html` and `budget-dashboard.html`:

```css
:root { --accent: #b68a2c; }
```

- Navigation/header is duplicated in pages; updating links or the logo requires editing each HTML file that contains the nav block (`index.html`, `budget-dashboard.html`, `refund-policy.html`, etc.).
- Downloads live at `downloads/` and are referenced with relative paths. E.g., `thankyou.html` points to `downloads/Sturbee_Financial_Reset_Guide.pdf`.
- Images are in `images/`; use existing assets when possible (e.g., `images/sfc.png`, `images/sfc-text.png`). Provide `alt` text when adding new `<img>` tags.
- Minimal JS: keep inline scripts small; there's only `document.getElementById('year').textContent = new Date().getFullYear();` in `index.html`.

## Developer workflows (discoverable/helpful commands)
- Local preview: open HTML directly in the browser or run a static server to test relative links and downloads:
  - Quick: `open index.html` (macOS)
  - Serve via Python: `python3 -m http.server 8000` then visit `http://localhost:8000/`
- No build/test pipeline present. Treat changes as direct edits that must be validated by manual browser testing.

## Style & content conventions
- Filenames use lowercase, hyphen-delimited words for pages (e.g., `budget-dashboard.html`). The PDF in `downloads/` uses underscores—preserve exact names when linking.
- Typography: Google Fonts are loaded in pages that include the longer style block (e.g., `index.html`, `budget-dashboard.html`) — prefer using those fonts for new content to maintain consistency.
- Accessibility: images in existing pages include `alt` attributes; follow the same pattern for additions.

## PR guidance for agents
- PR title should be concise and task-oriented (e.g., "Update brand color and adjust hero styles").
- Include a short checklist in the PR description listing changed files and manual validation steps (pages to open and smoke-test: `index.html`, `budget-dashboard.html`, `thankyou.html`).
- Avoid sweeping refactors since there is no test harness; break changes into small, reviewable commits.

## What to avoid
- Don’t assume a build system or tooling (there is none). Avoid adding heavy tooling without maintainer approval.
- Avoid globally renaming files without updating all relative references.

---

## Deployment (Netlify) ✅
- Recommended: Use **Netlify** for continuous deploys from GitHub (static site, no build needed).
- Quick setup (Netlify UI):
  1. In Netlify: **New site from Git** → **GitHub** → select this repository and branch (e.g., `main`).
  2. Build command: **leave blank**. Publish directory: **/** (root of repo).
  3. Enable automatic deploys on push.
- Optional: add a small `netlify.toml` at the repo root to make publish settings explicit:

```toml
[build]
  publish = "."
  command = ""
# Optional redirects for SPA clients:
# [[redirects]]
#   from = "/*"
#   to = "/index.html"
#   status = 200
```
- Optional: GitHub Actions workflow example (useful when you want Netlify deploys driven from CI rather than Netlify's Git webhook). Set these repository secrets first: `NETLIFY_AUTH_TOKEN` (personal access token from Netlify) and `NETLIFY_SITE_ID` (site id found on your site's dashboard).

Example workflow to add as `.github/workflows/deploy-netlify.yml`:

```yaml
name: Deploy to Netlify
on:
  push:
    branches: [ main ]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install Netlify CLI
        run: npm install -g netlify-cli
      - name: Deploy to Netlify
        env:
          NETLIFY_AUTH_TOKEN: ${{ secrets.NETLIFY_AUTH_TOKEN }}
          NETLIFY_SITE_ID: ${{ secrets.NETLIFY_SITE_ID }}
        run: netlify deploy --prod --dir=. --site=$NETLIFY_SITE_ID
```

**Notes:**
- You can also connect the repo directly in Netlify and skip creating files—Netlify will build from your branch. `netlify.toml` is helpful when you want explicit settings checked in.
- After setup, pushing to the configured branch should trigger an automatic deploy. Test by making a small content change and pushing.

If anything important is missing (preferred host, build steps, CI rules, or other conventions), tell me and I will update this file accordingly.