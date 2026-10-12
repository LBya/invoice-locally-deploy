# Purpose & Intent — invoice-locally-deploy

**RATIFIED 2026-10-12 — owner, fatten grill session (docs/fatten/invoice-locally-deploy-research.md).**
Drafted 2026-10-11 from code-review evidence (see WORKLOG.md, 2026-10-11 entry).
Nothing in this document is implementable in the block that drafted it.

## Purpose

Give a non-technical customer (a musician or freelancer who bought a license through the
Whop store) a two-file, one-command way to run the `invoicer` application on their own
machine: download `docker-compose.yml`, paste their license key into a `.env` file, run
`docker-compose up -d`, and open http://localhost:8000.

## What this project IS

- A **deployment wrapper** only. It contains no application code. The application is the
  separate `invoicer` project, consumed as the prebuilt image `ghcr.io/LBya/invoicer:latest`.
- The **customer-facing install surface** of the commercial edition: its readme is the
  entire setup documentation a customer sees.
- A **data-location contract**: it binds `./invoicer` (host) to `/app/data` (container),
  which is where the app keeps its SQLite database, license cache, and hardware ID.

## What this project is NOT (non-goals)

- Not an application repo — no backend/frontend code, no tests of app behavior live here.
- Not a build pipeline for the image (that belongs to the invoicer repo; ADR-0001).
- Not a multi-host or clustered deployment — single machine, single container, local port.
- Not a licensing authority — validation happens inside the app against the Whop proxy;
  the wrapper only carries the key into the container's environment (ADR-0002).

## Invariants (draft)

1. No secret value is ever committed to this repository (key reaches the container only
   via a local `.env` file that is never committed — ADR-0002).
2. Customer data (SQLite DB, license files) survives container replacement: it lives on
   the host bind mount, not in the container layer (ADR-0003).
3. Documented steps must exactly match the compose file — every placeholder named in the
   readme must exist in the artifact, and vice versa (2026-10-11 defect #1 violated this).
4. The wrapper tracks the app's real contract: port 8000, `/app/data`,
   `INVOICE_LICENSE_KEY` (verified against `Dockerfile.release` and
   `license_service.py` on 2026-10-11).

## Open decisions for the human (from the review sign-off queue)

- Bind port to loopback only or all interfaces? (review item 4)
- Named volume vs host bind mount for `/app/data`? (review item 5 — affects first-run
  permissions on Linux)
- Pin image versions vs keep `latest` pull-to-update flow? (review items 2–3)
