# ADR-0001: Distribute the invoicer as a prebuilt GHCR image, not build-from-source

**RATIFIED 2026-10-12 — owner, fatten grill session (docs/fatten/invoice-locally-deploy-research.md).** 2026-10-11. Evidence source:
WORKLOG.md 2026-10-11 entry; `docker-compose.yml` line 3; invoicer repo
`.github/workflows/` (read-only cross-reference).

## Status

Proposed (observed practice: the compose file already references a prebuilt image; this
ADR records the implied decision and its gap).

## Context

The wrapper's entire job is to let a non-technical customer run the app without a
toolchain. Building the image on the customer's machine would require cloning the
invoicer repo, Node, Poetry, and a long build — incompatible with the two-file install
story.

## Decision (draft)

The wrapper consumes `ghcr.io/LBya/invoicer:latest`, built by the invoicer repo from
`Dockerfile.release` and published to GitHub Container Registry. The customer never
builds.

## Consequences

- Positive: install is two files and one command; customers never see the source repo.
- Gap found in review (needs sign-off, review item 2): no *active* CI publishes that
  image today — the invoicer repo's `cd.yml` has no docker push, and the only workflow
  that would publish it is disabled (`build-commercial.yml.example`). Until that is
  enabled (or a documented manual push exists), `docker compose pull` has no
  reproducible source. This ADR cannot be considered satisfied until the publication
  path exists.
- Open: whether `latest` (pull-to-update) or pinned version tags are used — review
  items 2–3, likely a separate ADR once decided.
