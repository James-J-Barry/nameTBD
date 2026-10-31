# 0006 — Use MongoDB Atlas as the primary datastore

- **Status:** Accepted
- **Date:** 2026-10-30
- **Related:** [Architecture](../architecture/README.md) · [Data model](../architecture/data-model.md) · [Atlas setup](../architecture/atlas-setup.md) · [ADR-0001](0001-registry-first-direction.md) · [ADR-0003](0003-third-party-timestamps.md)

## Context

The registry-first direction (ADR-0001) needs a datastore for:

- **Variable-shape records:**
  - games imported from Steam or registered by hand
  - evidence items of several types
  - AI disclosure across 10 categories, each with a source and a quote
- **Faceted search:** text, tags, and any combination of "None in category X" filters, for the browse view, the Steam overlay and the API.
- **Similarity detection:** description, screenshot and title similarity, used only as internal prompts for people (ADR-0004).
- **An append-only audit trail:** for registrations, disputes and moderator decisions.
- **Time-varying signals:** review counts and positive % over time.
- **Event-driven work:**
  - re-evaluating a record when its evidence changes
  - upgrading pending timestamp proofs
  - opening dispute response windows

The owner has chosen MongoDB Atlas.

## Options considered

1. **MongoDB Atlas.** Documents fit nested evidence and AI data naturally. Atlas Search covers text search and facets. Atlas Vector Search covers similarity. Time series collections cover review history. Change streams drive event-driven jobs. It's a managed service with backups and encryption. Cons: relational-style integrity (e.g. one-to-many disputes) has to be enforced in the application or with schema validators; vendor lock-in for the Search and Vector Search features.
2. **Postgres + pgvector + a search engine.** Strong integrity, but three systems to run, or heavier extensions.
3. **Firebase / Firestore.** Weak at ad-hoc faceted queries and vector search.

## Decision

Use **MongoDB Atlas** as the single primary datastore. Specifically:

- **Collections and document shapes** as defined in [data-model.md](../architecture/data-model.md), enforced with `$jsonSchema` validators.
- **Atlas Search** for browse, search and AI-filter facets.
- **Atlas Vector Search** for look-alike detection. Results go to internal tooling only.
- **Time series collections** for review snapshots.
- **Change streams**, consumed by our own worker service, for event-driven jobs.
- **Binary evidence files are not stored in Atlas.** They go to object storage, addressed by SHA-256. Atlas stores hashes, metadata and timestamp proofs.
- **Clients never talk to Atlas directly.** The browser extension, the site and partners all go through our API service, using the official MongoDB driver. This also means we don't depend on Atlas's deprecated Data API.

## Consequences

- The data model must be designed around access patterns. Where it's read-heavy we denormalise, e.g. the AI summary is embedded on the game document.
- Rules that span collections must live in application code and be covered by tests:
  - a registration is immutable once filed
  - only one active verification per game
  - decisions are append-only

  Database roles back the append-only rules up: the API user gets insert-only access to the audit collections.
- Atlas Search and Vector Search indexes are part of the codebase (JSON definitions in the repo) and are applied per environment.
- Free (M0) and Flex tiers are fine for prototypes. Production needs a dedicated tier for backups, private networking and predictable Search performance. Re-check current tier limits when provisioning.
- The v0 `MOCK_DATA` maps onto these collections, so it can become a seed script for v1.
