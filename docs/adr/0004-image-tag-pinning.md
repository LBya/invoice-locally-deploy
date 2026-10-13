# ADR-0004: Pin the release image to a moving major tag (`v1`), not `latest`

**Accepted 2026-10-13 — implements owner ruling of 2026-10-12 (WORKLOG sign-off
queue item 3: "pin a versioned tag … implementer picks the pinned default and
documents it").** Parameters chosen by the implementing agent within that mandate.
Supersedes the open question left in ADR-0001 ("whether `latest` … or pinned
version tags are used").

## Context

ADR-0001 distributes the app as the prebuilt image `ghcr.io/lbya/invoicer` (lowercase;
see Decision note 3 below — earlier documents spelled the path `LBya`, which Docker
rejects at reference-parse time). Until now the wrapper referenced the floating
`:latest` tag, so any upstream build — including a breaking one — would be pulled onto
every customer's machine by the documented update flow (`docker compose pull` +
`docker compose up -d`). The readme's "Updates" section is the customer's only update
path; a non-technical customer cannot roll back or diagnose a broken pull.

## Decision

- `docker-compose.yml` tracks `ghcr.io/lbya/invoicer:v1` — a **moving major tag**.
- The invoicer release workflow (its `build-commercial.yml`, enabled the same day)
  publishes every release under its full semver tags, its `v<major>` tag, and
  `latest`; a v1.x release re-cuts `v1`, so `docker compose pull` keeps delivering
  updates **within major version 1** without any change to this wrapper.
- **Lowercase path (recorded 2026-10-13, rung-2 review):** the image path must be
  lowercase. Proof: `docker image inspect ghcr.io/LBya/invoicer:v1` (read-only parse
  check) → `invalid reference format: repository name (LBya/invoicer) must be
  lowercase`, while the lowercase form parses; GHCR normalizes package names to
  lowercase, so this is the only pullable spelling. Earlier ratified docs (ADR-0001,
  the 2026-10-11 review) spelled it `LBya` — those records keep their historical
  spelling; every living artifact (compose, readme, purpose, glossary) uses lowercase.
- `latest` remains published upstream for the owner's convenience but the wrapper
  deliberately does not track it. Pull-to-update therefore stays the customer flow
  (as the ruling contemplated), but only for compatible (same-major) releases.
- When a new major is released (v2), this wrapper's compose file is updated to the
  new pin and customers re-download the file — an explicit, documented step, never
  a silent upgrade.

## Consequences

- Positive: a breaking upstream release can never reach an existing customer
  machine; updates are still one familiar command for v1.x fixes.
- Positive: reproducibility at the major-version level — any wrapper copy pulls
  the same major line; full reproducibility (exact digest) was weighed and left
  out: it would kill the pull-to-update flow the project relies on, requiring a
  wrapper commit for every patch release (ADR-0001's two-file, no-toolchain story
  includes customers who downloaded the file once).
- Cost: the wrapper needs one commit + customer re-download per major bump —
  accepted and documented in the readme "Updates" section.
- Follow-through: the pinned tag only exists once the invoicer repo pushes a
  `v1.*.*` tag after enabling `build-commercial.yml` (pushes are owner-side; see
  WORKLOG 2026-10-13). Until that first publish, `docker compose pull` against
  `v1` will 404 exactly as it did against an unpublished `latest`.
