# Architecture

- **Status:** Draft
- **Date:** 2026-10-30
- **Related:** [ADR-0006 MongoDB Atlas](../decisions/0006-mongodb-atlas.md) · [Data model](data-model.md) · [Atlas setup](atlas-setup.md) · [Vision](../vision.md)

This describes how the registry-first product ([ADR-0001](../decisions/0001-registry-first-direction.md)) is built on MongoDB Atlas. It's a design target: nothing beyond the v0 single-file prototype exists yet.

## System overview

```
                 ┌──────────────────────────┐
 Developers ───► │  Web app (register,      │
 Players    ───► │  record pages, browse)   │──┐
                 └──────────────────────────┘  │
 Steam overlay (browser extension) ────────────┤  HTTPS (public read API,
 Partners / publishers (API keys) ─────────────┤  authenticated write API)
                                               ▼
                                   ┌──────────────────────┐        ┌───────────────────┐
                                   │  API service (Node)  │──────► │  Object storage    │
                                   │  official MongoDB    │        │  evidence files    │
                                   │  driver; rules logic │        │  keyed by sha256   │
                                   └──────────┬───────────┘        └───────────────────┘
                                              │ driver (TLS, private endpoint in prod)
                                              ▼
                ┌───────────────────────────────────────────────────────────┐
                │                    MongoDB Atlas cluster                   │
                │  collections · $jsonSchema validators · Atlas Search ·     │
                │  Atlas Vector Search · time series · change streams       │
                └───────────────────────────┬───────────────────────────────┘
                                            │ change streams
                                            ▼
                                ┌──────────────────────────┐      ┌──────────────────────┐
                                │  Worker service          │────► │  Timestamp services   │
                                │  timestamps, Steam sync, │      │  OpenTimestamps /     │
                                │  embeddings, re-eval,    │      │  RFC 3161 TSA         │
                                │  dispute deadlines       │      └──────────────────────┘
                                └──────────────────────────┘
```

| Component | Responsibility | Talks to Atlas as |
|---|---|---|
| **API service** | All reads/writes from clients. Validates input, applies business rules (registration immutability, dispute state machine), shapes public responses (never exposes internal fields). | `app_api` role (see [atlas-setup](atlas-setup.md#database-users-and-roles)) |
| **Worker service** | Consumes change streams and runs scheduled jobs: anchors and upgrades timestamp proofs, imports Steam metadata and AI disclosures, computes embeddings, writes similarity prompts, enforces dispute deadlines. | `app_worker` role |
| **Moderator tools** | Internal UI (descendant of v0's moderation dashboard) for disputes and optional verification. Goes through the API with moderator auth. | via API only |
| **Object storage** | Evidence files (builds, art, documents) addressed by SHA-256. Private by default. Atlas holds only metadata and hashes. | — |

## Key flows

### 1. Self-serve registration (core)
1. The developer uploads files or pastes links. The client computes SHA-256 hashes locally. Files are uploaded to object storage only if the developer opts in.
2. The API inserts a `registrations` document (`state: "filed"`) holding the evidence metadata and hashes. In the same transaction it inserts an `audit_events` entry.
3. The worker sees the insert on a change stream. It submits each hash to the timestamp services and saves the proof on the evidence item (`timestamp.state: "pending"`).
4. A scheduled job upgrades pending OpenTimestamps proofs once they're anchored (`"anchored"`). The public record page shows each item's date source.
5. The record page is live from step 2. Moderation is **not** on this path (ADR-0001).

### 2. Steam import and AI disclosure
- The worker fetches Steam store data on a schedule. It upserts `games` by `steam.appId` and parses the AI disclosure text into `ai.<category>`. A category Steam says nothing about stays `not_disclosed` (ADR-0005).
- Review counts are appended to the `review_snapshots` time series collection.

### 3. Browse, overlay and API reads
- The public site and the API use **Atlas Search** (`$search` / `$searchMeta`) on `games`: text, tag facets, `verification.state`, and any combination of `ai.<category>.level` filters.
- The Steam overlay calls `GET /v1/games/by-steam/:appId` (a point read on a unique index). It returns the public projection only: registry status, record permalink and AI breakdown.

### 4. Look-alike prompts (internal only)
- When a game or record is added, the worker computes text and image embeddings. It stores them on `games.embeddings` and in `media`.
- `$vectorSearch` finds near neighbours, and the results are written to `similarity_prompts`, which only moderator tools read.
- Per ADR-0004, these prompts never change public visibility. A matching `evidence.sha256` registered by another party gets flagged the same way. This is the detection for v0 test case 19.

### 5. Disputes
- `disputes` holds a state machine: `opened → awaiting_response → under_review → resolved`. Every transition is an `audit_events` insert.
- The worker watches `respondBy` deadlines. Outcomes are published only as dated facts (ADR-0002).

## Cross-cutting rules

- **Public projection.** One server-side projection decides what leaves the API. Internal fields (`internal.*`, `similarity_prompts`, moderator notes, contact details) never reach clients.
- **Immutability.** Filed registrations aren't edited. A correction is a new registration with `supersedes`. Audit events are insert-only, enforced by role privileges.
- **Language.** No stored field, enum value or API response uses "copy", "clone" or "fake" (ADR-0002). Use `lookalike`, and only inside internal collections.
- **Fictional seed data** only, outside production.

## Open questions

- Hosting for the API and worker (any Node host works; we're not committed).
- Embedding models for text and images, and how often to recompute.
- Choice of object storage provider, and retention for evidence files the developer didn't make public.
- Whether to use Atlas Triggers instead of a self-hosted worker for some jobs. The default is our own worker consuming change streams, which keeps the logic in the repo and testable.
