# CLAUDE.md

Guidance for Claude / Cursor agents working in this repository.

## Project overview

Multi-customer PoC landing-page hub. Each customer of DevRev's solutions team gets an isolated, single-file landing page that documents the PoC scope, success criteria and timeline, plus a login-gated DevRev plugSDK chatbot (CX Agent) and a "Launch Computer" button into their playground workspace.

All pages live as static HTML on the same Vercel project at `https://poc-delta-cyan.vercel.app`. Routing relies on Vercel's static-file precedence beating the catch-all rewrite in `vercel.json` — creating `public/<Customer>/index.html` is enough to serve `https://poc-delta-cyan.vercel.app/<Customer>`.

## Active subsites

| Customer | URL path | File | Language | DevRev workspace (devo) |
| --- | --- | --- | --- | --- |
| Amtech Software | `/` | `public/index.html` | English | `1oEuMwGIss` |
| Euromotors Altos Andes | `/EuromotorsAltosAndes` | `public/EuromotorsAltosAndes/index.html` | Spanish | `1seWEKR7EE` |
| Euromotors Volkswagen | `/EuromotorsVolkswagen` | `public/EuromotorsVolkswagen/index.html` | Spanish | `1WFFmxNvEE` |
| Phenom | `/Phenom` | `public/Phenom/index.html` | English | `1kDBFjrvrr` |

Each subsite file is ~67 KB of self-contained HTML + inline CSS + inline vanilla JS, ~2,000 lines. No framework, no build step, no external CSS files.

## Common architecture

- **Tabs:** Scope · (EX Agent / Postventa / Computer — name varies) · CX Agent · Success Criteria · Timeline. The middle product tab is sometimes hidden via the `hidden` attribute on the nav button.
- **Auth:** client-side IIFE near the bottom of each file with:
  - `USERS` map: lowercase email → `{ name, password? }`. Optional per-user `password` field overrides the global default.
  - `PASSWORD` constant (shared default).
  - `STORAGE_KEY` (unique per subsite — keeps sessions independent on the shared `poc-delta-cyan.vercel.app` origin).
  - Session in `localStorage` as `{ email, name, ts }`.
  - Email is normalised with `.trim().toLowerCase()` before lookup, so map keys must be lowercase.
- **CX widget:** DevRev plugSDK loaded after login. Initialised with a *namespaced* `user_ref` of the form `<customer-slug>-poc-<email>` (see Critical pitfalls below).

## Deployment

- **Primary:** Vercel project `poc` (`prj_9EpTtEHvgYcUJCvuAH6CiUuVMKaV`, team `dariosoria-3316s-projects`), aliased to `poc-delta-cyan.vercel.app`. Auto-deploys from `pocwebsite/main`.
- **Manual deploy:** `vercel --prod --yes` from the workspace root. Takes ~10s.
- **Local dev:** `cd public && python3 -m http.server 8080` then visit `http://localhost:8080/<Customer>/`. Wasmer (`wasmer run . --net`) also works but Python is faster.

## Critical pitfalls — read every session

These have all bitten us at least once. Re-check before you do similar work.

1. **MCP credentials must be updated per customer.** When the user asks for a new PoC subsite (or any verification work in DevRev), the first action — *before writing code* — is to confirm that the `user-devrev_<Customer>` MCP credentials point to the new customer's DevRev workspace. Confirm by calling `get_current_user` from that MCP and matching the `devoid` against the workspace that owns the customer's plug `app_id`. Without this, any DevRev-side verification produces data from the wrong tenant.
2. **Always namespace `user_ref`.** Use `<customer-slug>-poc-<email>` (`altosandes-poc-`, `volkswagen-poc-`, `phenom-poc-`, `amtech-poc-`). Otherwise `/internal/rev-users.identify` returns HTTP 409 ("input external ref is already associated with verified user") for any email already present as a verified Rev User and plugSDK silently fails to inject the launcher iframe — the chatbot appears to vanish after login. The real email keeps flowing via `user_traits.email`/`display_name` so the AI still sees the right contact.
3. **GitHub remote is `pocwebsite` (not `origin`)**. `origin` points to an unrelated `playground-template` repo. Vercel auto-deploys from `pocwebsite/main`. Pushes require the `Dario-Soria-devrev` GitHub account — run `gh auth switch --user Dario-Soria-devrev` first, otherwise the push fails with "Repository not found".
4. **`vercel --prod --yes` must run from the workspace root.** If launched from `public/` (e.g. after `cd public && python3 -m http.server`), the CLI creates a brand-new project called `public` and aliases to a different domain. The root-level `.vercel/project.json` is what targets the right `poc` project. If a phantom project ever appears, delete it with `echo y | vercel projects rm public`.
5. **DevRev plug origin allowlist.** A freshly created plug `app_id` may not have `https://poc-delta-cyan.vercel.app` in its allowed origins. Symptom: 4xx on `/api/plug-config`, empty launcher iframe. Ask the workspace admin to add the origin.
6. **Sessions are not cross-site-isolated automatically.** All subsites share the `poc-delta-cyan.vercel.app` origin so `localStorage` is shared. Per-subsite isolation comes from each file using a different `STORAGE_KEY`. Never reuse a `STORAGE_KEY` across subsites.

## How to add things

- **New customer subsite:** see [`docs/PoC-Playbook.md`](docs/PoC-Playbook.md).
- **New login user for an existing subsite:** add a lowercase-keyed entry to the `USERS` map in the relevant file; optionally include `password: '...'` to give that user their own password; commit + push to `pocwebsite/main` (Vercel auto-deploys).

## Style conventions

- Fonts loaded from Google Fonts CDN (Space Grotesk, Bricolage Grotesque, JetBrains Mono).
- All styling is inline in each `index.html` via a `<style>` block — no external CSS files, no build step.
- Responsive breakpoints at 968px (tablet) and 640px (phone). Phenom also tightens at 380px.
- Subsite copy should match the language of the customer's PoC document.
