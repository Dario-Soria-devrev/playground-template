# Deployment Guide

This is a static site with no build step. The files inside `public/` are served directly. Deployment takes under two minutes on any of the platforms below.

---

## Prerequisites

You need these tools installed before following any deployment path:

| Tool | Install | Required for |
|---|---|---|
| Git | [git-scm.com](https://git-scm.com) | All paths |
| Node.js 18+ | [nodejs.org](https://nodejs.org) | Vercel CLI |
| Vercel CLI | `npm i -g vercel` | Vercel deployment |
| Wasmer CLI | `curl https://get.wasmer.io -sSfL \| sh` | Wasmer Edge deployment |

Verify your installs:
```bash
git --version
node --version
vercel --version
wasmer --version
```

---

## Option 1 — Vercel (Recommended)

Vercel is the fastest path. The repo is already connected and every push to `main` deploys automatically.

### 1a. Deploy via Git push (CI/CD — no CLI needed)

```bash
# Make your changes, then:
git add .
git commit -m "your message"
git push origin main
```

Vercel detects the push and deploys within ~30 seconds. The live URL is:
```
https://playground-template-nine.vercel.app
```

### 1b. Deploy manually with the Vercel CLI

Use this to deploy a preview or to set up a new Vercel project from scratch.

```bash
# Install the CLI (once)
npm i -g vercel

# Log in (once)
vercel login

# From the repo root — first run walks you through project setup
vercel

# Deploy to production
vercel --prod
```

**First-time project setup answers:**

| Prompt | Answer |
|---|---|
| Set up and deploy? | `Y` |
| Which scope? | Your Vercel team/account |
| Link to existing project? | `Y` if repo already exists on Vercel, else `N` |
| What's your project name? | `playground-template` (or any slug) |
| In which directory is your code? | `.` (repo root) |
| Want to override settings? | `N` |

Vercel auto-detects this as a static site (no framework, no build command). It serves the `public/` directory directly.

### 1c. Vercel project settings (if setting up from scratch)

If connecting via the Vercel dashboard ([vercel.com/new](https://vercel.com/new)):

| Setting | Value |
|---|---|
| Framework Preset | Other |
| Root Directory | `.` |
| Build Command | *(leave empty)* |
| Output Directory | `public` |
| Install Command | *(leave empty)* |

### 1d. Enable SPA routing (required)

The site uses URL-path routing (e.g. `/fintech` must serve `index.html`). Add a `vercel.json` at the repo root if one does not exist:

```json
{
  "rewrites": [
    { "source": "/((?!customers|favicon|computer).*)", "destination": "/index.html" }
  ]
}
```

This rewrites all non-asset paths to `index.html` so the JS can read `window.location.pathname` and fetch the correct customer JSON.

> **Note:** If `vercel.json` already exists in the repo, this is already configured.

---

## Option 2 — Wasmer Edge

Wasmer Edge uses the `wasmer.toml` manifest already present in the repo.

```bash
# Install Wasmer (once)
curl https://get.wasmer.io -sSfL | sh

# Log in (once)
wasmer login

# Deploy to Wasmer Edge from the repo root
wasmer deploy
```

On first deploy, Wasmer will ask for a namespace and app name. After that, subsequent `wasmer deploy` calls update the existing app.

To run locally using the exact same server as production:
```bash
wasmer run . --net
# → http://localhost:8080/<customer-slug>
```

---

## Option 3 — Any Static Host (Netlify, GitHub Pages, Cloudflare Pages, S3)

Because this is plain HTML/CSS/JS with no build step, it works on any host that can serve static files. The only requirement is **SPA fallback routing** — unknown paths must serve `index.html`.

### Netlify

Add a `public/_redirects` file:
```
/*  /index.html  200
```

Then drag and drop the `public/` folder at [app.netlify.com](https://app.netlify.com), or use the CLI:
```bash
npm i -g netlify-cli
netlify deploy --dir=public --prod
```

### Cloudflare Pages

1. Connect the GitHub repo at [pages.cloudflare.com](https://pages.cloudflare.com)
2. Set **Build output directory** to `public`
3. Leave build command empty
4. Add a `public/_redirects` file (same as Netlify above)

### GitHub Pages

GitHub Pages does not support SPA fallback natively. Use a `404.html` trick — copy `public/index.html` to `public/404.html`. GitHub Pages serves `404.html` for all unknown paths, which allows the JS routing to work.

### AWS S3 + CloudFront

In the S3 bucket static website settings, set both **Index document** and **Error document** to `index.html`. This routes all requests through `index.html`.

---

## Running Locally (for development)

```bash
# Option A — npx serve (recommended, no install)
npx serve public --single --listen 8080

# Option B — Wasmer (matches production exactly)
wasmer run . --net

# Option C — Python (basic, no SPA routing — root path only)
python3 -m http.server 8080 --directory public
```

Then open: `http://localhost:8080/<customer-slug>`

> The `--single` flag on `npx serve` is required. Without it, `/fintech` returns a 404 instead of serving `index.html`.

---

## Environment & Secrets

This site has no server-side secrets. All configuration is in the customer JSON files inside `public/customers/`. The `widgetAppId` in each JSON is a DevRev PLuG app ID — it is a public client identifier and safe to commit.

No `.env` file is needed.

---

## Adding a Customer After Deployment

1. Copy one of the built-in templates:
   - `public/customers/template-blackduck.json` (default)
   - `public/customers/template.json`
2. Save as `public/customers/<slug>.json`.
3. Update customer values (widget app id, questions, use-case cards, stories, livePanel, contact info).
4. Add referenced local images to `public/assets/`.
5. Commit and push:
   ```bash
   git add public/customers/<slug>.json
   git commit -m "Add <customer> demo config"
   git push origin main
   ```
6. The new URL is immediately live at `https://your-domain.com/<slug>` — no redeployment or config change required.

---

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| `/fintech` returns 404 | SPA routing not configured | Add `vercel.json` rewrites or `_redirects` file |
| Page loads but shows "Customer not found" | JSON file missing or wrong slug | Check `public/customers/<slug>.json` exists |
| PLuG chat widget doesn't appear | `widgetAppId` missing or wrong | Verify the ID in the customer JSON matches your DevRev org |
| Nav links don't scroll | Browser cached old JS | Hard refresh (`Cmd+Shift+R` / `Ctrl+Shift+R`) |
| Fonts not loading | Network blocked Google Fonts | Fonts fall back to system sans-serif — no action needed |
