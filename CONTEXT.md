# CONTEXT.md — Nexus IQ working handoff

## §6 product maturity — committed as v2.5.1 → v2.5.8 (`/N-O8-G4`)

**Series:** eight commits on `main`, post-dated **2026-08-31** (00:30–02:00 IST) then
**2026-09-01** (06:00–07:30 IST). Author/committer `ItsMonarch04 <sps.tensor@gmail.com>`.

**What landed:** Every former §6 product idea — responsive shell + viewport policy,
operation center, guided MockModel demo, catalog freshness + provider wizard, lazy
screens (shell boot), Playwright/axe a11y + table cards, diagnostics/support bundle,
release provenance (version CI, SHA-pinned Actions, Docker smoke, vendored SheetJS).
Release notes live in **README.md Changelog** (no separate CHANGELOG file).

**Version state:** `v2.5.8` agrees across `package.json`, lockfile, and README Release.

---

Purpose: the single onboarding document for anyone (human or agent) picking up this repo on
another machine with zero prior context. Read this before touching anything.
Sections are labelled **FACT** (verified now), **HISTORY** (done, immutable), or **PLAN** (not done yet).

Last updated: 2026-09-01 (§6 series v2.5.1 → v2.5.8).

---

## 1. Snapshot — FACT (verify with `git status` before trusting)

| Item | State |
| --- | --- |
| Branch | `main`, tracks `origin/main` (https://github.com/ItsMonarch04/nexus-iq.git) |
| Last commit | `Commit v2.5.8: Release Provenance + Vendored SheetJS + Handoff /8-O8-G4` |
| Commit series | v2.5.1 → v2.5.8 (`/N-O8-G4`). Aug 31: shell, ops, demo, freshness. Sep 1 06:00+: wizard UI, a11y, diagnostics, provenance+handoff. |
| Working tree | **Clean** after the series (verify). |
| Version identity | **Consistent at `v2.5.8`:** Readme Release, `package.json`, lockfile. `npm run check:version` enforces these plus vendored SheetJS `file:` dep. |
| Tests | `npm test` → 880 tests, 878 pass, 2 conditionally skipped, 0 failures. Roots: `tests/{unit,server,e2e,integration,sim}`. A11y: `npm run test:a11y`. |
| Build / dependency audit | `npm run build` green on Next 16.2.10. `npm audit --omit=dev` may report ambient `sharp`/Next findings; CI audit stays report-only (§7). Do not force-downgrade Next. |
| Node | Requires >= 20.10 (`engines`); CI matrix 20.x + 22.x |
| CI | SHA-pinned actions; version-coherence → build → test → audit (report-only); docker-smoke job. Release workflow on `v*` tags (notes from Readme Changelog). |

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
(gitignored). Demo: Projects → “Guided MockModel demo”, or import `demo/techcorp-exit-survey.csv`.

## 3. Confirmed completed work — HISTORY

- `96b4c78` v2.1.0 through `d78f606` v2.2.0, then v2.3.x / v2.4.x / v2.5.0 PF-A1 (see prior logs).
- **v2.5.1 → v2.5.8 (`/N-O8-G4`, 2026-08-31 / 2026-09-01):** §6 product maturity —
  viewport policy + collapsible rail/inspector; jobs + Operation Center; guided MockModel
  demo (`POST /api/demo/reset`); catalog `pricingVerifiedAt` + freshness envelope; provider
  setup wizard UI; Playwright/axe a11y + table cards; diagnostics/support bundle (keys/vault
  never included) + backup docs; vendored SheetJS, SHA-pinned CI/release, version-coherence
  check, Docker smoke, CONTEXT handoff. Policies under `docs/`.

## 4. Backlog — FACT

No open product defects tracked. §4 item 7 (browser a11y suite) closed by v2.5.6.

## 5. Owner decisions needed — PLAN

1. **Accept the recorded future-dated `d78f606` history or explicitly request a history action.**
   Default: leave pushed history untouched; keep the clock correct going forward.
2. **Push the v2.5.1 → v2.5.8 series** when satisfied (`git log origin/main..HEAD`).
3. **Optional:** watch for an upstream Next patch for the ambient `sharp` advisory (do not
   downgrade to Next 14).

## 6. Product ideas — HISTORY (built in v2.5.1 → v2.5.8)

All eight former §6 ideas are shipped. See §3.

## 7. Decisions ledger / ignore list — HISTORY (do not re-litigate without owner)

- **No frontend framework.** Vanilla JS + hand-rolled router in `public/app`.
- **No web framework on the server.** Plain `node:http` + custom router.
- **MockModel ships on purpose**; default demo path; clearly labelled mock.
- **`config/` and `projects/` gitignored** except `.gitkeep`; keys only in `config/keys.json`;
  exports / support bundles must never include keys.
- **Single-writer bundle rule.**
- **Coder LAN sharing (`0.0.0.0`) is opt-in**; Host-header bypass for that listener is deliberate.
- **`npm audit` in CI is report-only.**
- **SheetJS is vendored** at `vendor/xlsx-0.20.3.tgz` (`file:` dep); CDN is upstream for re-vendoring.
- **LF line endings** (`.gitattributes`) — CSV fixtures are byte-exact.
- **`estimate: true` static pricing is accepted** — estimates ok, stale verified dates are not.
- **Do not rewrite pushed `d78f606` merely to fix its future date.**
- **Crash-guard policy: log-and-continue.**
- **`package.json` `private: true`.**
- **Release notes live in README.md Changelog** — no separate CHANGELOG file; the release
  workflow extracts the matching `### vX.Y.Z` section from the README.

## 8. Architecture boundaries & non-negotiables

- Adapters only via `getAdapter()` (`server/providers/registry.js`).
- JSON envelope `{ok:true,data}|{ok:false,error:{code,message}}`; SSE terminal failure via `error` event.
- Route modules default-export `[{method, pattern, handler}]` and auto-mount.
- Backend loopback-only; DNS-rebinding Host check stays.
- Filesystem ids validated at lowest path-building boundary with resolved-path containment.
- Bundle writes through `server/core/store.js` only.
- Tests: `node --test` under `tests/{unit,server,e2e,integration,sim}`; a11y via `npm run test:a11y`.

## 9. Conventions

- **Commits:** `Commit vX.Y.Z: <Short title> /<suffix>` on `main`. Ask before committing.
- **Versioning:** `package.json` is source of truth; Readme Release + commit prefix must match;
  bump lockfile root in the same change. `npm run check:version` gates CI. Tags optional;
  GitHub Releases use Readme Changelog body.
- **Docs:** README is `README.md`. Policies in `docs/` (viewport, support-bundle, backup-restore).

## 10. Working rules for agents — non-negotiable

1. Never commit or push unless the owner explicitly asks.
2. Never reset/discard/amend/force-push pre-existing work without confirmation.
3. Show file list + summary before commit; run `npm test` (and `npm run build` if needed) first.
4. Update `CONTEXT.md` before an approved commit that includes it; keep it a summary.
5. After a session: refresh §1, move finished work into §3 History, append one §11 line.

## 11. Session log (newest first)

- **2026-09-01** — v2.5.1 → v2.5.8 committed (`/N-O8-G4`). Aug 31 00:30–02:00 IST: responsive
  shell; ops center; guided demo; catalog freshness. Sep 1 06:00–07:30 IST: provider wizard
  UI; a11y + table cards; diagnostics bundle; release provenance + vendored SheetJS + this
  handoff. Deleted untracked CHANGELOG (notes stay in Readme). Verified pre-series:
  880/878 tests, build green.
- **2026-07-23** — §6 implementation (working tree) before the split commit series.
- **2026-07-19** — v2.4.0 → v2.4.4 (`/N-T6-F5`).
- **2026-07-17** — v2.3.7 Sync State Push.
- **2026-07-14** — Audit fixes → v2.3.0 → v2.3.6.
- **2026-07-13 / 07-11 / 07-09 / 06-29** — earlier audits and v2.1.0 re-baseline.

---

## Owner action items — outstanding

1. **Future-dated `d78f606` (v2.2.0) history.** Accept as-is (default) or request a history action.
2. **Push** the local v2.5.1 → v2.5.8 series when ready.
