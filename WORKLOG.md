# WORKLOG — invoice-locally-deploy

Append-only work journal for the PC Idle Time programme (vacay-idle / m-FLASH supervisor).
Project = deployment wrapper (docker-compose.yml + readme.md) for the separately developed
`invoicer` application (../invoicer — read-only reference for this project's blocks).
Entries are dated; never rewrite past entries.

---

## 2026-10-11 — Rung-2 dedicated code review of whole project (2 provable readme defects fixed; propose-tier queue filed; drafts drafted)

Block: queue row 18 ("LIBRARY PASS"). Window ~09:57–10:55 AEDT. Claim dir:
`~/.hermes/data/power_coordinator/orchestrator/queue-claims/2026-10-11/invoice-locally-deploy`.
Rung reached: **2 → 4 → 5** (review, then drafts; stop rule applies — see handoff).

### Scope reviewed

Whole tracked surface (2 files): `docker-compose.yml`, `readme.md` (git `main`, 1 commit
`8edf4da` at block start, working tree clean). Cross-checked against the invoicer app
(READ-ONLY): `Dockerfile.release`, `backend/Dockerfile`, `backend/app/main.py`,
`backend/app/services/license_service.py`, `backend/.env.example`, `.github/workflows/*`,
`COMMERCIAL_BUILD.md`. Docker host inspected READ-ONLY (`docker ps`, `docker images`):
no container or image from this project is currently deployed on the host, so nothing live
was touched. Guardrails held: no data dirs/volumes read; no state-changing docker commands;
no pull; no push; vacay-idle/queue.md untouched.

### Cross-check wrapper ↔ app (alignment verified — no defects in these)

- Port: compose `8000:8000` = `EXPOSE 8000` (Dockerfile.release:69, backend/Dockerfile:38) = readme "Open http://localhost:8000". Consistent.
- Data path: compose `./invoicer:/app/data` = `DATA_DIR = Path("/app/data")` (license_service.py:20) = Dockerfile.release:55. Consistent.
- Env var: compose `INVOICE_LICENSE_KEY=${LICENSE_KEY}` = `os.getenv("INVOICE_LICENSE_KEY")` (license_service.py `__init__`). Consistent. No inline secrets in the tracked files (verified by history scan: 0 credential-pattern hits in all commits).
- Healthcheck: release image bakes in its own HEALTHCHECK (Dockerfile.release:76–77); compose correctly omits one. No action needed.
- App enforces license at startup (main.py:42–46 blocks startup when invalid), so the readme's key-setup step is load-bearing — which is why the defect below matters.

### Defects FIXED this block (test-provable, minimal fix, local commit `1897679`)

1. **readme step 3 referenced a placeholder that does not exist in docker-compose.yml.**
   Proof (before): `grep -n "LL-PASTE-KEY-HERE" docker-compose.yml` → no match; compose
   contains `INVOICE_LICENSE_KEY=${LICENSE_KEY}` only. The documented instruction
   ("Replace LL-PASTE-KEY-HERE with your actual key") was unexecutable, and a customer
   following it would deploy with an empty license key → app blocks at startup.
   Fix (minimal, aligns docs to the artifact's existing env-indirection design): step 3 now
   says to create a `.env` file next to the compose file containing `LICENSE_KEY=<key>`
   (compose auto-loads `.env` for interpolation), plus a keep-it-private/do-not-commit note.
   Alternative considered and rejected in the same pass: replacing `${LICENSE_KEY}` with an
   inline placeholder in compose — rejected because it invites users to commit keys into a
   tracked file (regresses the current design). Before/after evidence: before,
   `grep -c "LL-PASTE-KEY-HERE" readme.md` = 1 / compose = 0; after, readme = 0 and the
   readme references `LICENSE_KEY`, matching compose 1:1.
2. **readme.md ended inside an unclosed code fence.** Proof (before): `grep -c '\`\`\`' readme.md`
   = 3 (odd; `od -c` of final bytes shows file ends at `docker-compose up -d` with no fence).
   Fix: appended the closing fence. After: fence count 6 (even).

Commit: `1897679 "Fix readme install steps to match docker-compose.yml"` (local only;
repo convention matched — history uses short imperative subjects; NOT pushed, per rules).

### Findings NOT implemented (agent-added, needs sign-off)

1. **HIGH — GitHub PAT embedded in the git remote URL** (`git remote -v` → `origin` fetch
   and push both embed a `ghp_…` token in the https URL, stored plaintext in `.git/config`;
   value deliberately NOT quoted anywhere in this file). History scan is clean: the token
   appears in 0 commits and 0 tracked files — it lives only in `.git/config`.
   Needed from the human: (a) rotate/revoke the token at GitHub; (b) then re-point the
   remote without an embedded credential (SSH remote or credential helper). Not auto-fixed:
   stripping the URL would silently change push/pull auth behavior, and rotation is
   human-only anyway.
2. **MEDIUM — `ghcr.io/LBya/invoicer:latest` has no active build path.** The invoicer repo's
   `cd.yml` contains no docker push (deploy steps are echo placeholders);
   `build-commercial.yml.example` (disabled — `.example` suffix) is the only workflow that
   would publish that exact image (it builds `Dockerfile.release` and would tag
   `ghcr.io/LBya/invoicer` from the repo name); `COMMERCIAL_BUILD.md` documents a manual
   build but with a different example name (`yourname/local-invoicer`). So `docker-compose
   pull` depends on an image whose publication is manual/unreproducible from CI. Not
   provable as a wrapper defect (cannot pull per rules; image may exist from a manual push)
   — hence propose. Proposal: enable the commercial build workflow in the invoicer repo and/
   or document the manual push; cross-ref the invoicer repo's own 2026-10-11 rung-2 block,
   which reviewed that repo but did not flag image publication.
3. **MEDIUM — floating `:latest` tag.** Update flow (readme "Updates") is pull-latest;
   no version pinning or digest pinning anywhere. Proposal: publish versioned tags and pin
   one in compose (keep `latest` only if the pull-to-update flow is intended).
4. **LOW — port mapping binds all interfaces** (`"8000:8000"` = 0.0.0.0). For a
   single-user self-hosted app, propose `"127.0.0.1:8000:8000"` unless LAN sharing is intended.
5. **LOW — first-run permission hazard on Linux.** `./invoicer` does not exist in the repo;
   Docker auto-creates bind-mount sources root-owned, while the container runs as non-root
   `appuser` (Dockerfile.release:62–66) and must WRITE to `/app/data` (SQLite DB + license
   files per license_service.py:20–21). Likely write failure on first Linux-native run
   (Docker Desktop file sharing is unaffected). Proposal: readme step to pre-create and
   chown the dir, or switch to a named volume. (Static evidence only — no container was
   run to prove it; that is why this is propose-tier, not a fixed defect.)
6. **LOW — `docker-compose` (v1) commands in readme.** v1 is EOL; propose `docker compose`
   (plugin) wording, or note both.
7. **LOW — repo has no `.gitignore`.** Now that the fixed readme instructs creating `.env`
   with a real key, a `.gitignore` entry for `.env` (and a `.env.example`) would close the
   leak path for the owner. Proposal only (new file, not a defect fix).
8. **NOTE (cosmetic) —** service `invoicelocally` vs `container_name: invoiceLocally`
   casing mismatch; readme step 1 store URL is an intentional placeholder template link.
   No action proposed beyond awareness.

### Purpose/intent doc + ADRs + glossary (rungs 4–5, DRAFT ONLY)

Drafted, all headed "DRAFT — needs human sign-off; no force until signed":
- `docs/purpose.md` — purpose/intent/non-goals/invariants for the wrapper.
- `docs/adr/0001-prebuilt-ghcr-image.md`, `docs/adr/0002-license-key-env-indirection.md`,
  `docs/adr/0003-bind-mount-data-dir.md` — from review evidence above.
- `docs/glossary.md` — wrapper/app/store/data-dir terminology.

Stop rule respected: nothing born from these drafts is implementable in this block; the
enhancement brainstorm (rung 3) is NOT run against my own draft purpose doc (self-made
mandate) — it unlocks for a LATER block after sign-off.

### Handoff (done / blocked / next)

- **DONE:** full review of both tracked files + cross-check vs invoicer app (alignment
  verified for port/data-path/env-var/healthcheck); 2 test-provable readme defects fixed
  and committed locally (`1897679`, not pushed); 8-item sign-off queue filed above;
  purpose/ADR/glossary drafts written under `docs/` (draft-only).
- **BLOCKED (on human):** sign-off queue items 1–7 above — especially #1 token rotation
  (security) and #2 image build path (the wrapper cannot `pull` reproducibly until the
  invoicer repo publishes). Nothing else blocks.
- **NEXT (for a later block, after sign-off):** work the sign-off queue in priority order
  (#1, #2, then the LOW items); after the human signs the purpose doc, rung 3 (enhancement
  brainstorm) becomes available to a later block. This project used no continue yet today;
  per same-day rules it is eligible for at most one continue later today.
- Compliance: no push/publish; no state-changing docker commands; no data/volume contents
  read; queue.md untouched; secret values never quoted in this file.
