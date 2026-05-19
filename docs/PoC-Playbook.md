# PoC Subsite Playbook

This is the recipe for adding (or maintaining) a customer PoC subsite in this repo. Follow it verbatim unless the user says otherwise.

For repo orientation and the critical pitfalls list, read [`CLAUDE.md`](../CLAUDE.md) first — that file is auto-loaded as a workspace rule on every session.

---

## 0. Pre-flight — ASK before writing code

The single most important step on every new project: **verify the DevRev MCP credentials point to the right customer workspace.**

1. Check whether a `user-devrev_<Customer>` MCP server already exists (see `/Users/dariosoria/.cursor/projects/.../mcps/`).
2. If it does not, or if it points to a previous customer's tenant, **stop and ask the user to update it.** Reference: "Necesito actualizar las credenciales del MCP de DevRev para apuntar al workspace de `<Customer>`. ¿Lo confirmás antes de seguir?"
3. Once the user confirms, run `get_current_user` against `user-devrev_<Customer>` and verify the returned `devoid` matches the workspace that will own the new plug `app_id`. Print the verification result in the chat before continuing.

Why this matters: every prior verification step we do (Rev User IDs, plug-settings, tickets, conversations) goes through this MCP. Wrong tenant = misleading data and wasted turns. The user explicitly asked to be reminded of this on every new project.

---

## 1. Required inputs to collect from the user

Use `AskQuestion` to batch these in 1–2 rounds of 1–2 questions each. Do not ask all at once.

1. **Customer name & URL path** — convention is `/<PascalCase>` (e.g. `/Phenom`, `/EuromotorsVolkswagen`). Confirm with the user.
2. **Language** — English or Spanish. Drives template choice.
3. **Content source** — usually a PDF dropped in `docs/` (e.g. `docs/<Customer> - PoC vN.pdf`). Read it before drafting copy.
4. **DevRev plug `app_id`** — long base64-ish string from a `plug_setting` record in the customer's DevRev workspace. Looks like `DvRvSt...xlxendsDvRv`. The middle portion base64-decodes to a `don:core:dvrv-us-1:devo/<id>:plug_setting/1__||__<timestamp>` string — verify the `devo/<id>` matches the customer's workspace.
5. **Live demo URL** — the "Launch Computer" button target, typically `https://app.devrev.ai/<slug>/computer`.
6. **Login allowlist** — list of emails + names. Phenom-style example: 4 customer contacts + 2–3 DevRev (AE, SE).
7. **Shared password** — one default for everyone (e.g. `Phenom1`, `EuroMotors1`, `AltosAndes1`, `Amtech-poc1`). Per-user overrides are possible but rare.
8. **Tab structure** — Scope and Success Criteria and Timeline are always visible. The middle product tab (`Computer` / `EX Agent` / `Postventa`) can be hidden, renamed, or unchanged. CX Agent is always visible.
9. **Sample questions** for each visible product tab (5 per tab is the convention).

---

## 2. Switch to plan mode

The full subsite is a large multi-file edit. Use `SwitchMode` to enter Plan mode before producing the implementation plan with `CreatePlan`. Past examples: see `docs/` and `.cursor/plans/`.

---

## 3. Scaffold

Pick the template by language:

| Language | Clone from | Notes |
| --- | --- | --- |
| English | `public/index.html` (Amtech) | Already has the `user_ref` namespacing fix (commit `4d1f19a`). |
| Spanish | `public/EuromotorsAltosAndes/index.html` *or* `public/EuromotorsVolkswagen/index.html` | Either works; both carry the same fix. |

```bash
mkdir -p public/<Customer>
cp public/<source>/index.html public/<Customer>/index.html
```

---

## 4. Edits checklist

### Identity & auth (always change)

- [ ] `<title>`
- [ ] Hero `.badge` text (e.g. `PROOF OF CONCEPT · <CUSTOMER>` or `PRUEBA DE CONCEPTO · <CUSTOMER>`)
- [ ] Hero H1 (often `Computer for Support` / `Computer para Ventas` / something customer-specific)
- [ ] Hero subtitle (1–2 sentences from the SoW)
- [ ] Footer / CTA `mailto:` contact (default `dario.soria@devrev.ai` unless told otherwise)
- [ ] Auth modal subtitle (`Sign in with your <Customer> credentials to continue.` etc.)
- [ ] `STORAGE_KEY = '<customer-slug>-poc-auth'`
- [ ] `PASSWORD = '<shared default>'`
- [ ] `CX_APP_ID = '<plug_setting_id>'`
- [ ] `userRef` prefix: `'<customer-slug>-poc-' + userEmail.toLowerCase()`
- [ ] `USERS` map — lowercase email keys, `{ name }` per entry; add `password: '...'` only for users with a non-default password

### Tabs

- [ ] Rename nav buttons as needed (e.g. `Computer` → `EX Agent`). Keep `data-tab` and `aria-controls` IDs in sync with the panel `id`.
- [ ] Hide a tab by adding `hidden` to its `<button class="nav-tab">`; leave the underlying panel intact so hash routing keeps working.

### Per-section content

- [ ] Scope → Problem Statement (rewrite from PDF)
- [ ] Scope → Use cases (`.scope-card` blocks; can be 2 or 3, with optional `scope-card-wide` for cards that contain a `.card-criteria` list)
- [ ] EX Agent / Computer tab → sample queries (5 typical) and Launch Computer URL
- [ ] CX Agent tab → sample customer questions (5 typical) and sign-in card copy
- [ ] Success Criteria → metric cards. If count differs from 3, also adjust the desktop `.criteria-grid` CSS:
  - 2 cards → `repeat(2, 1fr)`
  - 3 cards → `repeat(3, 1fr)` (default)
  - 4 cards → `repeat(2, 1fr)` (2×2)
- [ ] Timeline → `<div class="timeline-item">` blocks. Add date sub-spans only when the SoW has concrete dates.

### Cleanup audit

- [ ] `grep -niE "<previous customer name>|<previous slug>"` over the new file → should return zero matches in content (excluding the brand-context paragraph if it intentionally lists multiple brands).
- [ ] Run `ReadLints` on the new file → zero linter errors.

---

## 5. Verification (always, before committing)

Boot the local server:

```bash
cd public && python3 -m http.server 8080
```

Then via the chrome-devtools MCP:

1. Load `http://localhost:8080/<Customer>/`. Snapshot the static content via `evaluate_script`: title, badge, tab labels, use-case card count, criteria card count, timeline item count, sample questions for each tab, Launch URL, modal subtitle. Compare against the plan.
2. Log in as at least two users:
   - One DevRev-side user already known to exist as a Rev User in *any* workspace (good 409 stress test): e.g. `dario.soria@devrev.ai`.
   - One customer-side user.
   For each, assert:
   - `POST https://api.devrev.ai/internal/rev-users.identify` returns **201**.
   - Request body `user_ref` starts with the customer prefix.
   - JWT in the response, decoded, has `devoid` equal to the customer workspace.
   - Launcher iframe (`https://plug-platform.devrev.ai/launcher`) injected at bottom-right.
   - Widget iframe (`/widget/home`) created.
   - Zero console errors related to plugSDK.
3. Cross-check that the other 3 subsites are unaffected: visit `/`, `/EuromotorsAltosAndes`, `/EuromotorsVolkswagen` (and `/Phenom`) in fresh tabs. Each should show its own greeting state and its own title/badge. Different `STORAGE_KEY` means logging into one does not auto-log into another.

---

## 6. Commit, push, deploy

```bash
git add public/<Customer>/index.html
git commit -m "Add <Customer> PoC subsite at /<Customer>"

# Ensure the right GitHub identity is active.
gh auth switch --user Dario-Soria-devrev

git push pocwebsite main
```

For an instant production deploy (don't wait on Vercel auto-deploy from the push):

```bash
# Make absolutely sure you are at the workspace root, NOT in public/.
cd "/Users/dariosoria/Code/Snapins/2026/POC Builder"
vercel --prod --yes
```

Look for the final line `Aliased: https://poc-delta-cyan.vercel.app`. That confirms the deploy landed on the right project.

If Vercel CLI ever creates a phantom project (e.g. `public`), delete it: `echo y | vercel projects rm public`.

---

## 7. Re-run verification against production

Same Chrome DevTools script as step 5, this time against `https://poc-delta-cyan.vercel.app/<Customer>`. The 201 from `rev-users.identify` is the key assertion.

Once production verification passes, share with the user:

- The public URL.
- The full list of authorised emails + names.
- The shared password.

---

## 8. Maintenance flows

### Add a user to an existing subsite

1. Locate the right `USERS` map (one of the 4 files).
2. Insert a lowercase-keyed entry: `'user@example.com': { name: 'Full Name' },`. Optional `password: '...'` for an override.
3. Single-file commit, push to `pocwebsite/main`. Vercel auto-deploys.
4. If the user reports the chatbot is invisible after login → it's the verified-Rev-User 409. Confirm the `userRef` prefix is in place; if so it should already work. If the file is missing the prefix (older version), apply the namespacing fix as we did in commits `149bc78`, `74faab6`, `4d1f19a`, etc.

### Update copy / branding for an existing subsite

Straight content edit. Apply the change to the single file, commit, push. Vercel auto-deploys. Cross-check the live URL.

### Look up Rev User IDs for verification

The DevRev MCP `search` tool does not expose a `rev_user` namespace. To map `email → REVU-id` for an entire allowlist without cycling the UI:

1. Open the subsite in the browser via chrome-devtools MCP.
2. From `evaluate_script`, call `fetch('https://api.devrev.ai/internal/rev-users.identify', ...)` directly for each `{ email, name }` pair, passing the right `setting_id`.
3. Decode the response `token` (a JWT) and read the `http://devrev.ai/revuid` claim.

Reference implementation already used: see the agent transcript "Altos Andes Rev User audit" (May 19, 2026).

---

## 9. Anti-patterns — do not do these

- Do not touch existing subsites when adding a new one. They are all live and being shared with active customers.
- Do not commit secrets (passwords are fine — they're shared and intentionally exposed in client JS for the demo; but do not commit anything that would actually grant DevRev API access, like service tokens or AATs).
- Do not modify `vercel.json` to add per-subsite routing. The static-file precedence is what we rely on; explicit routing rules can break it.
- Do not rename or delete an existing subsite without explicit confirmation from the user — the URL is in the wild.
- Do not run `vercel --prod --yes` from `public/`. See pitfall #4.
- Do not push to `origin`. Always `pocwebsite main`.
