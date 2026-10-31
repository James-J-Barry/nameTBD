# Roadmap

- **Status:** Current
- **Date:** 2026-10-30
- **Related:** [Vision](vision.md) · [ADR-0001](decisions/0001-registry-first-direction.md) · [v0 prototype](prototypes/v0-discovery-site.md)

Living document. Move items between sections as they start/finish; link the research note when an experiment reports back.

## Now (next ~2 weeks)

### 1. Fix v0 issues that carry into any direction
Findings from the [v0 review](research/2026-10-28-prototype-v0-review.md). Each still applies to the new direction.

- [ ] Add a **`Not disclosed`** AI level; stop treating missing data as `None`; filters gain "include undisclosed" (ADR-0005).
- [ ] **Pending (unreviewed) registrations must not affect other games** (remove them from the collision reference set).
- [ ] Prevent **two look-alike records both being "Verified"** without a cross-check.
- [ ] Claims and registrations must **not hide another party's game** (ADR-0004).
- [ ] Routing: update the URL on navigation and listen for `hashchange`; make `#game=` links public by default, with a separate explicit internal-preview link.
- [ ] Featured slot on the store must respect active filters.
- [ ] Registration confirmation: replace internal jargon (`QUEUED priority 90`) with a developer-facing receipt.
- [ ] Day-by-day sweep should not silently skip session-touched cases (or should report them separately).

### 2. Stand up MongoDB Atlas (dev)
See [atlas-setup](architecture/atlas-setup.md) and [data model](architecture/data-model.md).

- [ ] Create the `firstmark-dev` Atlas project and a free (M0) or Flex cluster, with an IP access list and per-service database users.
- [ ] `db/` folder: `$jsonSchema` validators, standard indexes, the `games_search` Atlas Search index, and the `db:apply` script.
- [ ] `db:seed`: convert v0 `MOCK_DATA` into `games`, `registrations`, `disputes`, `audit_events` and `review_snapshots` documents.
- [ ] Prove the AI-filter query (any combination of "None" categories, with and without "include undisclosed") on Atlas Search against the seed data.

### 3. Build prototype v1: registry-first
A second prototype alongside v0 so the two directions can be compared. Unlike v0, it uses a thin Node API over the dev Atlas cluster, so the data model gets exercised for real.

- [ ] Self-serve registration → record is public and dated immediately (mock timestamp proof shown).
- [ ] Public record page with permalink, QR code and embeddable "Registered [date]" badge.
- [ ] Mock Steam store page with an extension-style overlay (registry status + AI breakdown).
- [ ] Minimal dispute flow (file → counter-notice → moderator decision → dated factual outcome).
- [ ] Document it in `prototypes/v1-registry.md`.

### 4. Validation experiments
| Experiment | Who | Success signal | Write-up |
|---|---|---|---|
| Developer registration | ~10 indie devs (Reddit / Discord dev communities) | ≥ 6 complete a registration; ≥ 3 place the link/badge somewhere public | `research/YYYY-MM-DD-dev-registration-test.md` |
| Player overlay | 5–10 players from AI-averse communities | Players use the overlay unprompted on ≥ 2 shopping sessions | `research/YYYY-MM-DD-player-overlay-test.md` |
| Timestamping explainer | Same participants | Can explain "what the badge proves" in their own words | fold into the above |

## Next

- Worker service: change-stream consumer with persisted resume tokens; timestamp anchoring and upgrade jobs.
- Atlas Vector Search for look-alike prompts (pick text and image embedding models).
- Choose hosting for the API and worker, and an object storage provider for evidence files.
- Staging Atlas project (dedicated tier), with private networking and field-level encryption for contact data.
- Pick a third-party timestamping approach (OpenTimestamps vs RFC 3161 TSA vs both) → decision record.
- Define the dispute process and what gets published → decision record + spec.
- Read-only public API (records + AI disclosure).
- Explore game-jam / festival partnerships as sources of independently dated records.

## Later

- Paid due-diligence access for publishers / platforms.
- Optional moderated "Verified" review.
- Private evidence vault with auto-ingest (git pushes, devlogs, social posts).
- Revisit idea claims once there is a dispute process and real usage data (ADR-0004).
- Final product name.

## Done

- 2026-10-27 — v0 prototype (discovery site + rules engine + moderation) built. [doc](prototypes/v0-discovery-site.md)
- 2026-10-28 — Three-lens review of v0; direction set to registry-first. [research](research/2026-10-28-prototype-v0-review.md) · [ADR-0001](decisions/0001-registry-first-direction.md)
- 2026-10-30 — Datastore set to MongoDB Atlas; architecture, data model and Atlas setup drafted. [ADR-0006](decisions/0006-mongodb-atlas.md) · [architecture](architecture/README.md)
