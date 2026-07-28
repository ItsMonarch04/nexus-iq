# CONTEXT.md — Nexus IQ working handoff

Purpose: the single onboarding document for anyone (human or agent) picking up this repo on
another machine with zero prior context. Read this before touching anything.
Sections are labelled **FACT** (verified now), **HISTORY** (done, immutable), or **PLAN** (not done yet).

Last updated: 2026-07-19 (implementation + verification session, committed as the **v2.4.0 → v2.4.4** series; local-only, unpushed).

---

## 1. Snapshot — FACT (verify with `git status` before trusting)

| Item | State |
| --- | --- |
| Branch | `main`, tracks `origin/main` (https://github.com/ItsMonarch04/nexus-iq.git) |
| Last commit | `Commit v2.4.4: Version Coherence + Handoff /5-T6-F5` — final commit of the **v2.4.0 → v2.4.4** series on top of pushed `fe64689` (v2.3.7). **`main` is 5 commits ahead of `origin/main`; not pushed** — the owner pushes after review. |
| Commit series | Five logical commits, one version each, `/N-T6-F5` suffixed (N = 1..5): v2.4.0 non-category human coding; v2.4.1 analyst suggestions route + opt-in UI; v2.4.2 iframe route/title bridge + evidence-door focus; v2.4.3 atomic project-cap reservation; v2.4.4 version coherence + this handoff. (The earlier seven-commit v2.3.x `/N-S6-F5` series is described in §3/§11.) |
| Commit date caveat | Owner-directed post-dating continues: the v2.3.x series is dated 2026-07-17 (real work 2026-07-14) and the v2.4.x series is **post-dated to 2026-07-28 (00:30–02:30 IST, +30 min each)** against a real work date of 2026-07-19. Author/committer `ItsMonarch04 <sps.tensor@gmail.com>`. This is deliberate; do not "fix" the dates. |
| Working tree | **Clean.** The 2026-07-19 implementation sweep is fully committed in the v2.4.x series; nothing is staged and `git diff --check` passes. |
| Version identity | **Consistent at `v2.4.4`:** README `Release version`, `package.json`, and lockfile all read `2.3.7` (each commit bumped them together); the UI rail and `/api/health` read the package version. `devDependencies` is empty (unused `lint` removed). README carries a per-version Changelog; no git tags. |
| Tests | `npm test` → 866 tests, 864 pass, 2 conditionally skipped, 0 failures. Verified 2026-07-19 in a loopback-capable environment. Focused `gold-integrity` regression test: 10/10 pass. |
| Build / dependency audit | `npm run build` green on Next 16.2.10; live `npm audit --omit=dev` reports 0 known vulnerabilities. Verified 2026-07-19. |
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

- `96b4c78` v2.1.0 (2026-06-29) — rebranding + build overhaul; repo effectively re-baselined.
- `0a61327` v2.1.1 (2026-07-09) — first audit round of fixes.
- `7ed2941` v2.1.2 / `084a3c7` v2.1.3 (2026-07-09) — CI workflow fixes, lockfile + package.json sync.
- `c277cf1` v2.1.4 (2026-07-09) — second audit round ("TCS5").
- `d78f606` v2.2.0 — documentation/handoff release: package and lockfile version aligned, MIT package license recorded, README release version added, and this context tracked. It also replaced raw NUL source bytes with visible `\x00` escapes in `public/app/js/components/toast.js` and `server/providers/registry.js`. Current history has six commits.
- Previously completed defensive work (not proof that no gaps remain; see §4): atomic writes + torn-tail
  NDJSON healing (`server/core/store.js`), DNS-rebinding + path-traversal guards
  (`server/router.js`), process crash guards + orphaned-run healing (`server/index.js`),
  privacy-mode gates as the only adapter constructor path (`server/providers/registry.js`),
  per-attempt cost accounting incl. failed attempts (`server/providers/base.js`).
- 2026-07-11 session: fresh-eyes audit (findings in §4), `license` field added to
  package.json, this file created.
- 2026-07-13 worktree: README quick-start prerequisites/install steps added; handoff
  refreshed from live Git, tests, build, npm audit, and a new code audit (documentation only).
- **v2.3.0 → v2.3.6 (committed locally on `main`, unpushed; post-dated 2026-07-17):** audit
  findings cross-checked and the genuine, well-scoped ones FIXED with regression tests, split
  into seven `/N-S6-F5` commits — v2.3.0 release prep + drop `lint`, v2.3.1 provider fixes +
  strict-mode egress, v2.3.2 filesystem hardening, v2.3.3 CSRF, v2.3.4 budget cap, v2.3.5
  frontend/a11y, v2.3.6 Docker + handoff. Details below. Server security: store-boundary
  slug validation + resolved-path containment (`server/core/store.js`); strict mode rejects
  non-loopback Ollama endpoints (`server/providers/registry.js`); cross-origin CSRF guard on
  every mutation (`server/router.js`, relaxed for the opt-in shared coder listener). Budget:
  the engine now enforces the PROJECT cap every unit (worker + Director spend), not just at
  admission (`server/runs/engine.js`, `server/routes/runs.js`). Providers: Anthropic omits
  sampling params for models that reject them + one-shot 400 retry, and Opus 4.8 metadata
  corrected to $5/$25 · 1M ctx; OpenAI catalog corrected to `gpt-5-mini` and GPT-5.2 $1.75/$14
  (`server/providers/anthropic.js`, `openai.js`). Privacy-at-rest: 0700/0600 modes for bundles,
  keys, and the PII vault + a boot-time chmod migration and abandoned-import GC (`store.js`,
  `routes/_shared.js`, `settings.js`, `ingest/pii.js`, `routes/import.js`, `index.js`).
  Frontend: idempotent sheet `onClose`, ordered/inert inspector, project-navigation store race
  gated by route slug, theme `aria-pressed` seeded, iframe `title` (`screens/_shared.js`,
  `components/inspector.js`, `main.js`, `pages/index.js`). Ops: Docker volumes + prod-only
  install + port/version cleanup + healthcheck; unused `lint` devDependency removed.

- **v2.4.0 → v2.4.4 (committed locally on `main`, unpushed; post-dated 2026-07-28):** the
  2026-07-19 sweep that resolved every §4 flagged item, split into five `/N-T6-F5` commits —
  v2.4.0 non-category human coding (sprint + blind coder: continuous, multilabel,
  extraction), v2.4.1 analyst suggestions (guarded route + opt-in run-detail UI), v2.4.2
  iframe route/title bridge + keyboard evidence doors, v2.4.3 atomic project-cap
  reservation/settlement, v2.4.4 version coherence + this handoff. Version surfaces
  (`Readme.md` release marker + Changelog, `package.json`, lockfile) moved every commit.

## 4. Backlog — FACT (2026-07-19: prior flagged items resolved or deliberately classified)

The seven items that were previously waiting for an owner decision were revisited. The contained
product gaps are implemented and committed in the v2.4.0–v2.4.4 series; the one policy choice
is now explicit.

1. **Human coding for non-category constructs — RESOLVED.** Both the blind coder and the
   full-bleed sprint now support bounded numeric labels for `continuous`, true toggle-and-submit
   sets for `multilabel`, and one-verbatim-span-per-line submissions for `extraction` (including
   an intentional empty span array). The latter matches judge output shape while preserving
   legacy free-text data acceptance. `tests/server/gold-integrity.test.js` covers span arrays.
2. **Crash guard policy — DECIDED: retain log-and-continue.** Nexus IQ is a single long-lived
   local process whose background runs should survive an unexpected handler failure. The existing
   code/comments/lifecycle test consistently implement that policy. Exit-and-supervise is a
   different deployment architecture, not an incidental correctness fix.
3. **Analyst suggestions — RESOLVED.** A completed run can now request Director-backed analysis
   suggestions through a guarded route. The UI makes each proposed analysis opt-in: the researcher
   can dismiss it or explicitly create it; no suggestion auto-creates an artifact.
4. **Iframe route and title state — RESOLVED.** The Next host and same-origin app exchange a
   constrained route/title message. Parent URL/history carries the inner route, Back/Forward
   restores it, and an initial-load handshake closes the message-listener race. Browser-verified
   on 2026-07-19 for deep links, title sync, and Back navigation.
5. **Keyboard evidence doors — RESOLVED.** Quote evidence figures are focusable button-like
   controls with an accessible name and visible focus styling; existing SVG charts and native
   controls were checked for their own keyboard paths.
6. **Concurrent project-cap admission — RESOLVED.** Starts/resumes atomically reserve each run's
   estimated remaining headroom under the project lock; settlement releases that reservation and
   books actual spend in the same mutation. The focused metering regression coverage and full
   test suite pass.
7. **Browser/DOM automation — intentionally optional, not a known product defect.** No Playwright
   or axe dependency was added solely to turn a quality-investment idea into CI scope. The affected
   behaviors received static/server regression coverage where applicable plus a real browser
   verification this session. A dedicated accessibility acceptance suite remains a §6 investment
   for a public-web release, not a blocker for the local product.

Explicitly checked and still not issues: 866-test suite + production build health; `npm audit`
0 vulnerabilities; XSS-safe DOM call sites (`html:` escape hatch has zero users); NDJSON
torn-tail/concurrency protections; static-file containment; response error redaction; report
escaping; CSV formula hardening; `.gitignore`/`.gitattributes`; font and VADER licensing; no
TODO/FIXME debt.

## 5. Owner decisions needed — PLAN (blockers / next work)

1. **Review the v2.4.0 → v2.4.4 series and confirm the push.** The v2.3.x series is pushed;
   `main` is now 5 local commits ahead of `origin/main` with the v2.4.x series (`npm test`
   866/864 green, build green, `npm audit --omit=dev` 0 findings, all verified 2026-07-19
   pre-commit). Review `git log origin/main..HEAD` and complete the push when satisfied.
2. **No unresolved implementation blocker remains from §4.** The only non-code follow-up is to
   decide when a dedicated browser accessibility suite is justified for a public-web release.
3. Decide npm publication intent. `package.json` says `private: false`, but there is no entry
   point, package allowlist, or publish workflow. Do not flip the flag or invent policy.
4. Accept the recorded future-dated `d78f606` history or explicitly request a history action.
   Default: leave pushed history untouched and correct the clock for future commits.

## 6. Product ideas (optional — NOT bugs, NOT commitments) — PLAN

### Visual / interaction polish

- Responsive shell with collapsible rail/inspector overlay. **Value:** polished laptop/tablet
  use. **Scope:** medium CSS/layout. **Dependency:** supported viewport policy. **Defer:** yes if
  desktop-only is documented.
- Unified busy/progress language and a persistent operation center. **Value:** paid/slow work
  feels deliberate across navigation. **Scope:** medium-large. **Dependency:** centralized job
  state. **Defer:** yes for private beta.

### UX clarity and conversion

- One-click guided MockModel demo. **Value:** high first-run conversion with no keys. **Scope:**
  medium. **Dependency:** deterministic demo/reset path. **Defer:** after correctness blockers.
- Provider setup wizard with validation, price/capability freshness stamps, and privacy cues.
  **Value:** fewer failed/expensive first calls. **Scope:** medium. **Dependency:** provider
  backlog 6-7 resolved. **Defer:** safe for technical beta, not ideal for public launch.

### Performance / accessibility

- Route-level module loading. **Value:** reduce the roughly 724 KB uncompressed eager frontend.
  **Scope:** medium. **Dependency:** router support. **Defer:** yes for local desktop use.
- Browser accessibility acceptance suite and responsive table alternatives. **Value:** launch
  confidence for keyboard, screen-reader, zoom, and narrow layouts. **Scope:** medium.
  **Dependency:** Playwright/axe. **Defer:** not before broad public launch.

### Content / deployment / operational maturity

- In-product diagnostics/support bundle plus backup/restore guidance. **Value:** high for a
  local filesystem/provider product. **Scope:** medium. **Dependency:** redaction and bundle
  compatibility policy. **Defer:** until beta, not general availability.
- Release provenance and supply-chain automation: version-coherence CI, tags/CHANGELOG/releases,
  action permission/SHA hardening, Docker smoke test, and vendored/mirrored SheetJS tarball.
  **Value:** repeatable public releases. **Scope:** medium. **Dependency:** owner release policy
  and licensing/update cadence. **Defer:** only until the first public release.

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
  numbers are not (see backlog 6-7).
- **Do not rewrite the pushed v2.2.0 commit merely to repair its future date.** The anomaly is
  recorded in §1; any new history action requires explicit owner direction.

## 8. Architecture boundaries & non-negotiables

- All provider adapters are constructed **only** via `getAdapter()`
  (`server/providers/registry.js`). `strict` is required to mean on-machine backends only and
  overrides must be ledgered; backlog 2 is a current violation to close, not a new policy.
- Every JSON route speaks the `{ok:true,data}|{ok:false,error:{code,message}}` envelope
  (`server/router.js`); SSE routes signal terminal failure via an `error` event.
- Route modules in `server/routes/*.js` default-export `[{method, pattern, handler}]` — the
  server auto-mounts them.
- Backend binds loopback only; the DNS-rebinding Host check in `server/router.js` must stay.
- Every filesystem identifier must be validated at the lowest path-building boundary and the
  resolved path must remain inside its configured root; route-only validation is insufficient.
- Writes to bundles go through `server/core/store.js` (atomic write / locked NDJSON append);
  never `fs.writeFile` a `project.json` directly.
- Tests use `node --test`; new server behavior lands with a test in `tests/`.

## 9. Conventions

- **Commits:** existing format is `Commit vX.Y.Z: <Short title>` on `main`; the six-commit
  history starts at the v2.1.0 re-baseline. The owner prefers several small logical commits,
  compact titles joined with `+` or `/`, and may require a sequence suffix. Ask for the exact
  message/suffix and wait for approval before committing; do not infer it from inconsistent
  historical examples.
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
4. Ensure `CONTEXT.md` is updated before any approved commit and include only the documentation
   changes that belong to that commit; it is already tracked.
5. **Update protocol:** after any working session, update §1 Snapshot, move finished items
   from §4/§5 into §3 History, and append one Session-log line. Keep this file a summary —
   details belong in reports/commits, not here.

## 11. Session log (newest first)

- **2026-07-19** — v2.4.0 → v2.4.4 committed (Fable 5). Split the verified working tree into
  five logical commits (`/N-T6-F5`, post-dated 2026-07-28 00:30–02:30 IST, author
  `ItsMonarch04`): non-category coding; analyst suggestions; iframe bridge + evidence-door
  focus; atomic cap reservation; version coherence + handoff. The intermediate analyst commit
  carries a self-contained route/UI subset; the reservation commit completes the shared
  import/roll-up refactor. Full `npm test` re-run pre-commit: 866 total, 864 pass, 2 intended
  skips. Local only — not pushed.
- **2026-07-19** — Context implementation + verification (Codex). Left all changes deliberately
  uncommitted. Implemented continuous, multilabel, and extraction controls for both human-coding
  surfaces; wired opt-in post-run analysis suggestions; synchronized iframe route/title state
  with a same-origin initial-load handshake; made quote evidence doors keyboard reachable; and
  replaced snapshot-only project-cap admission with atomic per-run reservations plus atomic
  settlement. Retained the documented crash-guard policy after review. Verified `npm test`
  (866 total: 864 pass, 2 intended skips), focused gold-integrity coverage (10/10), production
  build, live dependency audit (0 vulnerabilities), and browser deep-link/title/Back behavior.
- **2026-07-17** — v2.3.7 Sync State Push (Claude Code). State-sync release: bumped the version
  marker to 2.3.7 across `Readme.md` (Release version + a new Changelog entry), `package.json`,
  and both root `package-lock.json` fields, and refreshed §1 / §5 / this log to the pushed,
  in-sync state. No code or functional change. One commit, `Sync State Push /O8`, post-dated
  2026-07-17 04:15 IST; left unpushed for the owner's own review/push.
- **2026-07-14** — Audit cross-check + fixes (Claude Code). Verified the 2026-07-13 audit's
  findings against the code, then FIXED the genuine, well-scoped ones with regression tests
  (all P1s + the scoped P2/P3s — see §3). Six items were deliberately NOT implemented and are
  flagged in §4 for owner decision (sprint controls for non-category construct types; the
  deliberate crash-guard behavior; `analyst.js`; iframe deep-link architecture; a browser test
  suite; the wider a11y pass). Corrected the README Docker section (volumes now persist data).
  `npm test` 862/864 pass (2 env-gated skips, 9 tests added), `npm run build` green, `npm audit`
  0 vulnerabilities, live boot smoke-tested (health/create/CSRF/strict-ollama).
  After owner review of the split, the fixes were committed on `main` as the **v2.3.0 → v2.3.6**
  series (7 commits, `/N-S6-F5` suffixed, post-dated 2026-07-17 01:00–04:00 IST, author
  `ItsMonarch04`). A first attempt that committed before showing the split was reset (`git reset
  --mixed`) and redone once approved. **Local `main` is 7 commits ahead of `origin/main`; not
  pushed.** Next step: owner review of the local series, then push when approved.
- **2026-07-13** — Fresh-eyes audit + handoff refresh (Codex). Audit started from a clean tree.
  Verified live `origin/main == d78f606`, version coherence, 855 tests (853 pass, 2 gated
  skips), green production build, and live npm audit with 0 known vulnerabilities. Found the
  P1 store traversal, strict-mode remote Ollama escape, CSRF gap, unenforced project hard cap,
  unsupported human-coding types, provider/catalog drift, and frontend races, plus the lower
  priorities in §4. Added README quick-start/Docker truth and refreshed this file only. No code
  fix, commit, tag, push, reset, amend, or history rewrite was performed.
- **2026-07-11** — Fresh-eyes audit + context bootstrap (Claude Code). Ran full test suite
  (853/855 pass, 2 env-gated skips) and production build (green). Found: P1 temperature-param
  400s on current Claude models, P1 stale Opus 4.8 pricing (3x) + ctx values, version-identity
  split, unused `lint` dep, Docker niggles (§4). Added `"license": "MIT"` to package.json
  (matches LICENSE/README). Created this file. Owner later approved the v2.2.0 handoff commit:
  it tracks this file and aligns the release version. Original `224ff670` used the requested
  2026-07-13 00:30 IST timestamp; it was later amended/replaced by current pushed `d78f606`,
  whose only extra delta normalizes two raw NUL separators and whose Git date is 2026-07-15.
- **2026-07-09** — v2.1.1 → v2.1.4: audit-fix rounds + CI/lockfile repairs (see History).
- **2026-06-29** — v2.1.0 rebrand/build re-baseline; repo history starts here.

---

## Owner action items — outstanding (as of 2026-07-22)

Repo state confirmed: `main` (`6c91da1a`) matches `origin/main` at **v2.4.4**. (§1 and the 2026-07-19 session log entries still say "5 commits ahead of `origin/main`; not pushed" — that is now stale; v2.4.0–v2.4.4 are pushed and in sync.) Every §4 flagged item is resolved and committed. Nothing implementation-side remains; every item below is a policy call only you can make:

1. **npm publication intent.** `package.json` says `private: false`, but there is no entry point, `files` allowlist, or publish workflow. Either flip to `private: true` (matches actual behaviour) or write the publish policy — do not leave the mismatch.
2. **Future-dated `d78f606` (v2.2.0) history.** Recorded, pushed, uncorrected. Either accept as-is or explicitly request a history action. Default recommendation from §5: leave pushed history untouched and keep the clock correct for future commits.
3. **Dedicated browser accessibility acceptance suite.** Not needed for the local product; decide when it is justified for a public-web release (§6 investment; §4 item 7 flagged it as intentionally optional).
