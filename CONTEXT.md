# CONTEXT.md — Nexus IQ working handoff

Purpose: the single onboarding document for anyone (human or agent) picking up this repo on
another machine with zero prior context. Read this before touching anything.
Sections are labelled **FACT** (verified now), **HISTORY** (done, immutable), or **PLAN** (not done yet).

Last updated: 2026-07-11 (fresh-eyes audit session — see Session log).

---

## 1. Snapshot — FACT (verify with `git status` before trusting)

| Item | State |
| --- | --- |
| Branch | `main`, tracks `origin/main` (https://github.com/ItsMonarch04/nexus-iq.git) |
| Last commit | `Commit v2.2.0: Revamp + Context` (requested timestamp 2026-07-13 00:30 IST), local-only on `main`; the prior pushed tip is `c277cf1` / v2.1.4 |
| Working tree at handoff | Clean after the v2.2.0 commit, which adds the MIT package license, tracks this context, and aligns the version metadata |
| Version identity | **Consistent:** release commit + README + `package.json` + lockfile use `v2.2.0` / `2.2.0`; the UI rail and `/api/health` read the package version and therefore display `2.2.0`. No git tags or CHANGELOG. |
| Tests | `npm test` → 855 tests, 853 pass, 2 conditionally skipped (one needs `NEXUS_IQ_REPORTING_DUMP`, one needs Python+numpy/pandas). Verified 2026-07-11. |
| Build | `npm run build` (Next 16.2.10 / Turbopack) green. Verified 2026-07-11. |
| Node | Requires >= 20.10 (`engines`); CI matrix 20.x + 22.x |
| CI | `.github/workflows/ci.yml`: npm ci → build → test → `npm audit` (report-only) on push/PR to main |

## 2. What this is — plain language

Nexus IQ is a **local-first qualitative text measurement platform**: import text corpora
(CSV/XLSX/DOCX/PDF/transcripts), draft measurement constructs, compile LLM- or
dictionary-based instruments, calibrate them against human-coded gold data, run labeled
analyses with cost metering, and export reproducible reports. Two processes:

- **Next.js shell** on port 3000 — an iframe host + `/api/*` proxy only. No React UI.
- **Node backend** on port 7341 (`server/`, plain `node:http`, no framework) — serves the
  real UI (vanilla JS in `public/app/`) and all APIs. Project data lives on disk in
  `projects/<slug>/` bundles (gitignored).

Everything works offline via the deterministic **MockModel** provider; real providers
(Anthropic, OpenAI, OpenRouter, Ollama) activate when keys exist in `config/keys.json`
(gitignored). Demo walkthrough: import `demo/techcorp-exit-survey.csv`, use mock provider.

## 3. Confirmed completed work — HISTORY

- `96b4c78` v2.1.0 (2026-06-29) — rebranding + build overhaul; repo effectively re-baselined (5-commit history).
- `0a61327` v2.1.1 (2026-07-09) — first audit round of fixes.
- `7ed2941` v2.1.2 / `084a3c7` v2.1.3 (2026-07-09) — CI workflow fixes, lockfile + package.json sync.
- `c277cf1` v2.1.4 (2026-07-09) — second audit round ("TCS5").
- v2.2.0 — documentation/handoff release: package and lockfile version aligned to the release, MIT package license recorded, README release version added, and this context tracked.
- Hardening visible throughout the code and locked in by tests: atomic writes + torn-tail
  NDJSON healing (`server/core/store.js`), DNS-rebinding + path-traversal guards
  (`server/router.js`), process crash guards + orphaned-run healing (`server/index.js`),
  privacy-mode gates as the only adapter constructor path (`server/providers/registry.js`),
  per-attempt cost accounting incl. failed attempts (`server/providers/base.js`).
- 2026-07-11 session: fresh-eyes audit (findings in §4), `license` field added to
  package.json, this file created.

## 4. Verified bugs / backlog — FACT (found 2026-07-11, none fixed yet)

Priority order. These are evidence-backed defects, distinct from ideas (§6).

1. **P1 — Anthropic adapter sends `temperature` unconditionally; current-gen Claude models reject it (HTTP 400).**
   `server/providers/anthropic.js:36` always sends `temperature: req.temperature ?? 0`.
   Anthropic rejects `temperature`/`top_p`/`top_k` on Opus 4.7/4.8 and Fable 5 (and rejects
   non-default values on Sonnet 5) with 400. `claude-opus-4-8` is the first row of this
   adapter's own static catalog, so the advertised flagship fails every call. 400 is
   non-retryable (`retryClass` in `server/providers/base.js`), so affected units fail/quarantine.
   Same pattern in `server/providers/openai.js:125` (`buildBody`) — OpenAI reasoning models
   (o-series, gpt-5.x) likewise reject non-default temperature; OpenRouter inherits the body but
   its gateway strips unsupported params for many upstreams, which masks it there. The catalog
   already has a `noTemperature` concept, but `server/routes/catalog.js:41` hardcodes it `false`
   for every provider except OpenRouter, and it's UI-advisory only — the wire body is never gated.
   Invisible to the 855-test suite because tests run on MockModel.
   **Fix direction:** omit `temperature` from the wire body unless the model is known to accept
   it (populate `noTemperature` for anthropic/openai catalogs, or model-prefix gate in the
   adapters), and/or catch a parameter-rejection 400 once and retry without the param.
2. **P1 — Stale Anthropic static pricing/context inflates cost metering 3x.**
   `server/providers/anthropic.js:13`: `claude-opus-4-8` priced $15/$75 per 1M; actual is
   $5/$25. `ctx` says 200_000 for opus-4-8 and sonnet-4-6; both are 1M-context models.
   Pricing feeds preflight estimates AND the live run meter (`server/runs/engine.js`
   `pricingFor` → `meter().add`), whose USD total drives the hard budget abort
   (`checkBudget` in `server/providers/costs.js`) — so a $10 cap aborts a real Opus 4.8 run
   around $3.30 of actual spend. `ctx` is display/filter-level (model picker), lower impact.
   **Fix direction:** correct the three static rows; consider a comment dating the prices.
3. **P3 — Unused devDependency `lint@1.2.2`** (`package.json`): no script, config, or import
   references it; it's an abandoned generic-linter package installed in every CI and Docker
   build. **Fix direction:** `npm rm lint` (a real linter setup is a separate idea, §6).
4. **P3 — Docker niggles.** `Dockerfile` `EXPOSE 7341` is misleading: the backend binds
   `127.0.0.1` inside the container (`server/index.js` `startServer`), so publishing 7341 can
   never work (the shell proxy on 3000 is the supported path — works fine). Runtime-stage
   `npm ci` installs devDependencies. `docker-compose.yml` declares no volume, so
   `projects/` data dies with the container. **Fix direction:** drop/comment `EXPOSE 7341`,
   use `npm ci --omit=dev` in the runtime stage, add a `projects/` volume.

Explicitly checked and NOT issues: test suite health (855 green), router security guards
(traversal/rebinding/size caps, with dedicated tests), crash-safe store, XSS-safe DOM layer
(`public/app/js/dom.js`; the `html:` escape hatch has zero call sites), no TODO/FIXME debt,
.gitignore/.gitattributes hygiene, vendored font licensing (OFL text shipped), VADER lexicon
licensing (MIT, non-redistributable lexicons deliberately not bundled).

## 5. Owner decisions needed — PLAN (blockers)

- **Approve/decline the P1 fixes** (temperature gating, pricing refresh) before any code change.
- **Release tags/CHANGELOG:** optional; the release-version policy is already resolved in §9.

## 6. Product ideas (optional — NOT bugs, NOT commitments) — PLAN

Grouped proposals from the 2026-07-11 premium-polish review live in that session's report.
Headlines only (each safe to defer): in-product "Load demo project" onboarding; provider setup
wizard with key validation; surface `noTemperature`/pricing badges for all providers in the
model picker; `<link rel="preload">` for the four body fonts; axe/keyboard a11y pass over
charts and tables; README upgrades (Docker section, CI badge, screenshots, `?fixtures=1`
gallery mode); CHANGELOG.md + git tags; ESLint/Prettier + CI lint job; vendor the SheetJS
tarball (install currently depends on cdn.sheetjs.com uptime); docker-compose healthcheck +
volume. None are prerequisites for local use.

## 7. Decisions ledger / ignore list — HISTORY (do not re-litigate without owner)

- **No frontend framework.** `public/app` is deliberate vanilla JS + hand-rolled router/DOM
  helpers; the Next.js layer exists only as shell/proxy. Do not propose React/Vue migrations.
- **No web framework on the server.** Plain `node:http` + custom router is intentional.
- **MockModel ships on purpose** and is the default demo path; it is clearly labelled mock.
- **`config/` and `projects/` are gitignored** except `.gitkeep`; keys live only in
  `config/keys.json`; exports must never include keys.
- **Single-writer bundle rule:** exactly one process writes a project bundle. The blind-coder
  listener is a role inside the same process on an ephemeral port — never a second process.
- **Coder LAN sharing (`0.0.0.0`) is an explicit researcher opt-in**; the Host-header guard
  bypass for that restricted listener is deliberate and documented in `server/index.js`.
- **`npm audit` in CI is report-only by design** (doesn't block merges).
- **SheetJS `xlsx` installs from the vendor CDN tarball** — that is SheetJS's official
  distribution channel, not an accident (vendoring it is an open idea, §6).
- **Line endings are LF everywhere** (`.gitattributes`), because CSV fixtures with embedded
  newlines are byte-exact; don't "fix" this.
- **`estimate: true` static pricing is an accepted design** — estimates are fine, stale
  numbers are not (see backlog #2).

## 8. Architecture boundaries & non-negotiables

- All provider adapters are constructed **only** via `getAdapter()`
  (`server/providers/registry.js`) so privacy modes (`open` / `no-training` / `strict`)
  cannot be bypassed; `strict` = local backends only; overrides must be ledgered.
- Every JSON route speaks the `{ok:true,data}|{ok:false,error:{code,message}}` envelope
  (`server/router.js`); SSE routes signal terminal failure via an `error` event.
- Route modules in `server/routes/*.js` default-export `[{method, pattern, handler}]` — the
  server auto-mounts them.
- Backend binds loopback only; the DNS-rebinding Host check in `server/router.js` must stay.
- Writes to bundles go through `server/core/store.js` (atomic write / locked NDJSON append);
  never `fs.writeFile` a `project.json` directly.
- Tests use `node --test`; new server behavior lands with a test in `tests/`.

## 9. Conventions

- **Commits:** message format `Commit vX.Y.Z: <Title>` on `main` (no tags so far; 5-commit
  history starts at the v2.1.0 re-baseline).
- **Versioning:** `package.json` is the source of truth. README and the `Commit vX.Y.Z:` prefix
  must match it; update the root lockfile version in the same change. Tags and a CHANGELOG are
  optional and require owner approval.
- **Docs:** README is `Readme.md` (that casing is tracked; keep it unless renamed on purpose).

## 10. Working rules for agents — non-negotiable

1. **Never commit or push unless the owner explicitly asks.** Working-tree edits are fine;
   `git commit`/`git push`/`git tag` require an explicit instruction naming what to commit.
2. Never reset, discard, amend, or force-push pre-existing work without confirmation.
3. Show the owner the exact file list + summary before any commit; run `npm test` (and
   `npm run build` if the shell/deps changed) first; keep unrelated changes out.
4. Ensure `CONTEXT.md` is included (tracked) in the next commit the owner approves.
5. **Update protocol:** after any working session, update §1 Snapshot, move finished items
   from §4/§5 into §3 History, and append one Session-log line. Keep this file a summary —
   details belong in reports/commits, not here.

## 11. Session log (newest first)

- **2026-07-11** — Fresh-eyes audit + context bootstrap (Claude Code). Ran full test suite
  (853/855 pass, 2 env-gated skips) and production build (green). Found: P1 temperature-param
  400s on current Claude models, P1 stale Opus 4.8 pricing (3x) + ctx values, version-identity
  split, unused `lint` dep, Docker niggles (§4). Added `"license": "MIT"` to package.json
  (matches LICENSE/README). Created this file. Owner later approved the v2.2.0 handoff commit:
  it tracks this file and aligns the release version, with requested commit timestamp 2026-07-13
  00:30 IST.
- **2026-07-09** — v2.1.1 → v2.1.4: audit-fix rounds + CI/lockfile repairs (see History).
- **2026-06-29** — v2.1.0 rebrand/build re-baseline; repo history starts here.
