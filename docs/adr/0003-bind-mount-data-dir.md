# ADR-0003: Customer data persists via host bind mount `./invoicer` → `/app/data`

**DRAFT — needs human sign-off; no force until signed.** 2026-10-11. Evidence source:
WORKLOG.md 2026-10-11 entry; `docker-compose.yml` line 8; invoicer `Dockerfile.release`
(55–66) and `license_service.py` (20–21), read-only cross-reference.

## Status

Proposed (observed practice; a known hazard is filed for sign-off — review item 5).

## Context

The app writes everything durable to `/app/data`: the SQLite database, the license
cache (`license.json`), and the hardware-bound instance ID (`instance_id`). The
container is disposable (`restart: unless-stopped`, `pull` + `up -d` update flow), so
data must live outside the container layer. The wrapper binds the host directory
`./invoicer` (relative to the compose file) to `/app/data`.

## Decision (draft)

Customer data lives on the host under `./invoicer` and survives every container
replacement, update, and restart. The container never stores durable state.

## Consequences

- Positive: update flow (pull + up -d) cannot lose customer invoices; backup = copy
  one folder.
- Hazard found in review (needs sign-off, review item 5): on Linux-native Docker, the
  auto-created host folder is root-owned while the container runs as non-root
  `appuser`, so first-run writes (DB, instance ID) may fail; Docker Desktop file
  sharing is unaffected. Options: readme pre-create/chown step, or a named volume.
  Decision needed before wide distribution.
- Cosmetic (review item 8): the folder name `./invoicer` matches the app repo's name,
  which may confuse (it holds data, not source). Renaming is cheap now, breaking later.
