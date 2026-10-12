# ADR-0002: License key reaches the container via env indirection from an uncommitted .env file

**RATIFIED 2026-10-12 — owner, fatten grill session (docs/fatten/invoice-locally-deploy-research.md).** 2026-10-11. Evidence source:
WORKLOG.md 2026-10-11 entry, fixed defect #1; `docker-compose.yml` line 10; invoicer
`license_service.py` (`os.getenv("INVOICE_LICENSE_KEY")`), read-only cross-reference.

## Status

Proposed (the compose file already implements this; the readme was corrected on
2026-10-11, commit `1897679`, to document it accurately).

## Context

The app blocks startup without a valid `INVOICE_LICENSE_KEY` (main.py:42–46), so every
customer must supply a key. The compose file references it as
`INVOICE_LICENSE_KEY=${LICENSE_KEY}` — compose interpolates from the environment or an
auto-loaded `.env` file next to the compose file. Putting the key inline in the compose
file would work but invites customers to commit or reshare a tracked file containing a
live secret.

## Decision (draft)

- The key is stored by the customer in a local `.env` file (`LICENSE_KEY=…`) that is
  never committed to version control and never shared.
- The compose file contains only the indirection `${LICENSE_KEY}`, never a literal key.
- Documentation must describe the `.env` mechanism (current readme step 3, fixed
  2026-10-11).

## Consequences

- Positive: no secret ever enters this repo's history (verified: 0 credential patterns
  in all commits, 2026-10-11 scan).
- Cost: one more file for the customer to create; mitigated by exact readme wording.
- Follow-up needing sign-off (review item 7): add `.gitignore` with `.env` and a
  `.env.example`, to protect the owner from committing their own key.
- Unrelated but discovered during this review (review item 1): the repo's git remote
  URL embeds a GitHub PAT in `.git/config` — rotate and re-point; that token is a
  repo-access credential, not part of this ADR's mechanism.
