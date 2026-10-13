# Glossary — invoice-locally-deploy

**RATIFIED 2026-10-12 — owner, fatten grill session (docs/fatten/invoice-locally-deploy-research.md).** 2026-10-11. Drafted from
code-review evidence (WORKLOG.md, 2026-10-11 entry).

| Term | Meaning in this project |
|---|---|
| **Wrapper** | This repository: the two tracked files (`docker-compose.yml`, `readme.md`) that deploy the app. Contains no application code. |
| **The app / invoicer** | The separate `invoicer` repository — the commercial invoicing application (FastAPI backend + SvelteKit frontend) consumed as a prebuilt image. Read-only reference for wrapper blocks. |
| **Release image** | The monolithic container image built from the invoicer repo's `Dockerfile.release` (frontend + backend in one container), published as `ghcr.io/LBya/invoicer`; the wrapper tracks its pinned `v1` major tag (ADR-0004). |
| **License key** | The customer's purchased Whop license. Passed as env var `INVOICE_LICENSE_KEY`; without a valid key the app blocks startup. |
| **`.env` file** | Uncommitted file next to the compose file holding `LICENSE_KEY=…`; compose auto-loads it for `${LICENSE_KEY}` interpolation. Never committed (ADR-0002). |
| **Data dir** | Host folder `./invoicer` bound to `/app/data` in the container; holds the SQLite database, `license.json`, and `instance_id` (hardware binding). All durable state lives here (ADR-0003). |
| **Instance ID** | UUID stored at `/app/data/instance_id`; binds the license to one installation so keys cannot be shared. |
| **License proxy** | Cloudflare Worker (`LICENSE_PROXY_URL` in the app) that validates Whop licenses without embedding secrets in the image. Owned by the invoicer repo, not this one. |
| **Whop store** | The sales channel; customers buy the license there, then install with this wrapper. Store URL in the readme is a template placeholder. |
| **Compose project** | The directory containing `docker-compose.yml`; its `.env` and relative bind-mount paths resolve from here. |
| **Pull-to-update** | The documented update flow (`docker compose pull` + `docker compose up -d`); delivers new builds within the pinned major tag `v1`, never a silent major bump (ADR-0004). |

**Cross-references:** purpose and non-goals in `docs/purpose.md`; distribution decision
ADR-0001; secret handling ADR-0002; data persistence ADR-0003. Findings and sign-off
queue in `WORKLOG.md` (2026-10-11 entry).
