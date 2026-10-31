# Data model (MongoDB Atlas)

- **Status:** Draft
- **Date:** 2026-10-30
- **Related:** [ADR-0006](../decisions/0006-mongodb-atlas.md) · [Architecture](README.md) · [Atlas setup](atlas-setup.md) · [Glossary](../glossary.md)

Database: `firstmark` (one per environment, see [atlas-setup](atlas-setup.md#environments)). All examples use fictional data.

## Conventions

- `_id`: `ObjectId`, except where noted. Public references use a human-readable `slug`.
- Dates are BSON `Date` (UTC). Fields that are a calendar date rather than an instant (e.g. a release date) are stored as `Date` at 00:00 UTC.
- Every document has `createdAt`, `updatedAt` and `schemaVersion` (int). Migrations bump `schemaVersion` and are idempotent scripts in the repo.
- Enum values are lowercase snake_case.
- Fields under `internal` are **never** included in the public projection.
- Banned vocabulary in field names and enum values: copy, clone, fake (ADR-0002).

## Collections at a glance

| Collection | Purpose | Write pattern | Public? |
|---|---|---|---|
| `games` | One per game: Steam-imported or registered | Upsert (import), update (AI disclosure, verification) | Public projection |
| `studios` | Developer accounts / organisations | Insert/update | Name only |
| `registrations` | Provenance records with evidence and timestamp proofs | **Insert-only** (+ worker updates to `timestamp` sub-docs) | Yes (permalink) |
| `disputes` | Dispute cases between parties | State-machine updates via API | Outcome summary only |
| `audit_events` | Every decision and state change | **Insert-only**, enforced by role | No |
| `similarity_prompts` | Look-alike candidates for moderators | Worker upsert | No |
| `media` | Screenshot / capsule metadata + image embeddings | Worker insert/update | URLs only |
| `review_snapshots` | Time series of review counts / positive % | Worker insert | Aggregates only |
| `rule_settings` | Versioned thresholds & priorities (successor to v0 Settings) | Insert new version | No |
| `api_clients` | Partner API keys (hashed), quotas | Insert/update | No |

## `games`

```js
{
  _id: ObjectId("…"),
  slug: "tidewright",
  title: "Tidewright",
  studioId: ObjectId("…"),                // → studios
  kind: "game",                            // game | dlc | soundtrack | tool
  parentGameId: null,                      // set for dlc/soundtrack
  tags: ["Roguelite", "Sailing", "Crafting"],
  description: "Build a ship plank by plank…",
  links: { steam: "https://store.steampowered.com/app/9100201/", developer: "https://saltglass.example/tidewright" },
  steam: {                                 // absent for pre-release games with no Steam page
    appId: 9100201,
    pageCreatedAt: ISODate("2025-05-14"),
    releasedAt: ISODate("2025-11-02"),
    earlyAccess: false,
    removedAt: null, removalReason: null,
    lastSyncedAt: ISODate("2026-10-30T04:00:00Z")
  },
  firstPublicRecordAt: ISODate("2024-12-01"),   // earliest independently dated public record
  visibility: "public",                    // public | hidden ; hidden only for non-accusatory reasons (ADR-0004)
  hiddenReason: null,                      // removed_from_steam | not_a_game | duplicate | content_policy | moderator_decision
  duplicateOfGameId: null,
  registry: {                              // denormalised from registrations for fast reads
    registered: true,
    latestRegistrationId: ObjectId("…"),
    firstRegisteredAt: ISODate("2026-10-01")
  },
  verification: {                          // optional human check (ADR-0001); never means "original"
    state: "verified",                     // none | requested | verified | declined
    decidedAt: ISODate("2026-10-14"),
    auditEventId: ObjectId("…")
  },
  ai: {                                    // all 10 categories always present
    art:         { level: "none",          source: "developer", quote: "…", updatedAt: ISODate("…") },
    music:       { level: "not_disclosed", source: null,        quote: null, updatedAt: ISODate("…") },
    sfx:         { level: "not_disclosed", source: null,        quote: null, updatedAt: ISODate("…") },
    voice:       { level: "none",          source: "developer", quote: "…", updatedAt: ISODate("…") },
    writing:     { level: "none",          source: "developer", quote: "…", updatedAt: ISODate("…") },
    design:      { level: "none",          source: "developer", quote: "…", updatedAt: ISODate("…") },
    code:        { level: "assisted",      source: "developer", quote: "…", updatedAt: ISODate("…") },
    translation: { level: "assisted",      source: "developer", quote: "…", updatedAt: ISODate("…") },
    runtime:     { level: "none",          source: "developer", quote: "…", updatedAt: ISODate("…") },
    marketing:   { level: "none",          source: "developer", quote: "…", updatedAt: ISODate("…") }
  },                                       // level: none | assisted | generated | not_disclosed (ADR-0005)
                                           // source: developer | steam_disclosure | null
  reviewSummary: { count: 1934, positivePct: 89, asOf: ISODate("2026-10-30") },  // latest snapshot, denormalised
  embeddings: { description: [/* float32 × N */], model: "text-embed-vX", computedAt: ISODate("…") },
  internal: {                              // never public
    notes: [],
    signals: { reviewLookalikeShare: 0.02, genreComparisonShare: 0.2, asOf: ISODate("…") }
  },
  createdAt: ISODate("…"), updatedAt: ISODate("…"), schemaVersion: 1
}
```

**Indexes**

| Index | Why |
|---|---|
| `{ slug: 1 }` unique | Permalinks |
| `{ "steam.appId": 1 }` unique, partial (`steam.appId` exists) | Overlay lookup, import upsert |
| `{ studioId: 1 }` | Studio dashboard |
| `{ parentGameId: 1 }` partial | DLC on parent pages |
| Atlas Search index `games_search` | Browse, text search, facets, AI filters (see below) |
| Atlas Vector Search index `games_description_vec` on `embeddings.description` | Look-alike prompts (internal) |

**Why Atlas Search for AI filters.** Users can require `none` in any subset of the 10 categories, which gives 1,024 combinations. Compound B-tree indexes can't cover that. An Atlas Search index with `ai.<category>.level` mapped as `token` handles any combination as `equals` clauses inside a `compound.filter`, plus facet counts through `$searchMeta`.

Example: "No AI in art or voice, include undisclosed = false, verified only":

```js
db.games.aggregate([
  { $search: {
      index: "games_search",
      compound: {
        filter: [
          { equals: { path: "visibility", value: "public" } },
          { equals: { path: "verification.state", value: "verified" } },
          { equals: { path: "ai.art.level", value: "none" } },
          { equals: { path: "ai.voice.level", value: "none" } }
        ],
        should: [{ text: { query: "sailing", path: ["title", "description", "tags"] } }]
      }
  } },
  { $project: PUBLIC_GAME_PROJECTION },
  { $limit: 24 }
])
```

With "include undisclosed" ticked, each category clause becomes `in: { path: "ai.art.level", value: ["none", "not_disclosed"] }`.

## `studios`

```js
{
  _id: ObjectId("…"), slug: "saltglass-studio", name: "Saltglass Studio",
  members: [{ userId: "auth-provider-id", role: "owner" }],
  contact: { email: BinData(6, "…") },     // encrypted (Queryable Encryption / CSFLE), see atlas-setup
  steamOwnership: [{ appId: 9100201, verifiedAt: ISODate("…"), method: "steamworks_key" }],
  createdAt, updatedAt, schemaVersion: 1
}
```

Indexes: `{ slug: 1 }` unique, `{ "members.userId": 1 }`, `{ "steamOwnership.appId": 1 }`.

Before a studio can attach a registration to an imported game, it has to prove it owns the Steam app (`steamOwnership`). This fixes the v0 issue where the picker leaked unannounced titles.

## `registrations`

```js
{
  _id: ObjectId("…"),
  publicId: "rec_7Hq2Kd",                  // permalink: /r/rec_7Hq2Kd
  gameId: ObjectId("…"),
  studioId: ObjectId("…"),
  filedAt: ISODate("2026-10-01T15:22:09Z"),
  prerelease: false,
  supersedesId: null,                      // corrections create a new registration
  state: "filed",                          // filed | withdrawn
  evidence: [
    {
      itemId: "ev1",
      type: "build_hash",                  // build_hash | concept_art | teaser | devlog | git | document
      label: "Release build v1.0.0",
      sha256: "3b8f…c201",                 // hex; required for file-based types
      url: null,                           // required for teaser/devlog
      claimedDate: ISODate("2025-11-02"),  // what the developer says
      dateSource: "timestamped",           // timestamped | publicly_posted | self_reported (ADR-0003)
      storage: { key: "evidence/3b8f…c201", public: false },   // object storage; null if not uploaded
      timestamp: {
        method: "opentimestamps",          // opentimestamps | rfc3161
        state: "anchored",                 // pending | anchored | failed
        proof: BinData(0, "…"),            // .ots / TSR bytes (small; kept inline)
        submittedAt: ISODate("2026-10-01T15:22:10Z"),
        anchoredAt: ISODate("2026-10-01T17:05:00Z"),
        lastCheckedAt: ISODate("…")
      }
    }
  ],
  createdAt, updatedAt, schemaVersion: 1
}
```

**Rules:**
- The API never updates `evidence[*]` fields except `timestamp.*`, which only the worker writes.
- `filedAt` is set by the server, never by the client.

**Indexes**

| Index | Why |
|---|---|
| `{ publicId: 1 }` unique | Permalinks |
| `{ gameId: 1, filedAt: 1 }` | Record history on game page |
| `{ "evidence.sha256": 1 }` | Detect the same file registered by different parties → `similarity_prompts` (v0 case 19) |
| `{ "evidence.timestamp.state": 1, "evidence.timestamp.submittedAt": 1 }` partial (`state: "pending"`) | Worker job: upgrade pending proofs |

## `disputes`

Replaces v0's claims, queue and appeals (ADR-0004: disputes are not public by default).

```js
{
  _id: ObjectId("…"),
  publicId: "dsp_K29xQ",
  type: "registration_conflict",           // registration_conflict | evidence_ownership | ai_disclosure_accuracy | record_correction
  subject: { gameId: ObjectId("…"), registrationId: ObjectId("…") },
  parties: [
    { role: "filer",      studioId: ObjectId("…") },
    { role: "respondent", studioId: ObjectId("…"), notifiedAt: ISODate("…"), respondBy: ISODate("…") }
  ],
  state: "awaiting_response",              // opened | awaiting_response | under_review | resolved | withdrawn
  priority: 90,                            // computed from rule_settings at open time
  evidence: [ /* same item shape as registrations.evidence, minus timestamp */ ],
  outcome: null,                           // { decidedAt, publicSummary (dated facts only), auditEventId }
  internal: { assignee: "mod_ines", notes: [] },
  openedAt: ISODate("…"), createdAt, updatedAt, schemaVersion: 1
}
```

Indexes: `{ state: 1, priority: -1, openedAt: 1 }` (the moderator queue, sorted the same way as v0), `{ "parties.studioId": 1, state: 1 }`, and `{ "parties.respondBy": 1 }` partial (state `awaiting_response`) for deadline jobs.

## `audit_events` (append-only)

```js
{
  _id: ObjectId("…"),
  at: ISODate("…"),
  actor: { type: "moderator", id: "mod_ines" },     // moderator | developer | system | worker
  action: "verification_granted",                    // registration_filed | timestamp_anchored | dispute_opened | dispute_resolved | visibility_changed | …
  subject: { collection: "games", id: ObjectId("…") },
  before: { "verification.state": "requested" },
  after:  { "verification.state": "verified" },
  reason: "Devlog and build hash consistent.",      // internal; never public
  requestId: "req_…"
}
```

- Indexes: `{ "subject.id": 1, at: 1 }`, `{ "actor.id": 1, at: -1 }`.
- The `app_api` and `app_worker` roles have `insert` + `find` only, with no `update` or `remove` ([atlas-setup](atlas-setup.md#database-users-and-roles)).
- State changes to `games`, `registrations` and `disputes` are written **in the same multi-document transaction** as their audit event.

## `similarity_prompts` (internal)

```js
{
  _id: ObjectId("…"),
  pair: [ObjectId("a…"), ObjectId("b…")],   // sorted, so (a,b) is unique
  scores: { title: 0.58, description: 0.81, screenshots: 0.88, keywords: 0.77, max: 0.88 },
  sameEvidenceHash: false,
  models: { text: "text-embed-vX", image: "img-embed-vY" },
  computedAt: ISODate("…"),
  reviewState: "unreviewed"                  // unreviewed | dismissed | linked_to_dispute
}
```

- Indexes: `{ pair: 1 }` unique, `{ reviewState: 1, "scores.max": -1 }`.
- **Read only by moderator tools.** These prompts never feed visibility (ADR-0004).

## `media`

```js
{ _id, gameId, kind: "screenshot", url: "https://cdn.example/…", sha256: "…", embedding: [/* float32 × N */], model: "img-embed-vY", createdAt }
```

Indexes: `{ gameId: 1 }`, plus an Atlas Vector Search index `media_image_vec` on `embedding` with a filter field `gameId`.

## `review_snapshots` (time series)

```js
// created with: db.createCollection("review_snapshots", { timeseries: { timeField: "ts", metaField: "gameId", granularity: "hours" } })
{ ts: ISODate("2026-10-30T04:00:00Z"), gameId: ObjectId("…"), count: 1934, positivePct: 89, lookalikeShare: 0.02, genreShare: 0.2 }
```

The latest point is copied to `games.reviewSummary`. `lookalikeShare` is the successor to v0's "copy-signal score" and is internal only.

## `rule_settings`

Versioned successor to v0's Settings panel: `{ version: 7, activeFrom, values: { SIMILARITY_PROMPT_THRESHOLD: 0.75, PRIORITY_…: … }, createdBy }`. Only the highest `activeFrom` ≤ now applies. Each dispute stores the version it was computed with.

## Mapping from v0 `MOCK_DATA`

| v0 field | Atlas location |
|---|---|
| `games[].dates.steamPageCreated / release`, `earlyAccess`, `removed` | `games.steam.*` |
| `games[].dates.firstPublicRecord`, `publicRecords[]` | `games.firstPublicRecordAt` + `registrations.evidence` (`dateSource: "publicly_posted"`) |
| `games[].ai`, `aiSource`, `aiQuote` | `games.ai.<category>.{level, source, quote}` (missing → `not_disclosed`) |
| `games[].registration` | `registrations` document |
| `games[].reviews`, `copySignal`, `genreSignal` | `review_snapshots` + `games.reviewSummary` / `games.internal.signals` |
| `games[].assetMatch`, `similarity[]` | `similarity_prompts` |
| `claims[]`, `appeals[]` | `disputes` |
| `moderationLog[]` | `audit_events` |
| `settings` | `rule_settings` |
| `games[].expected`, `truth`, `case` | Test fixtures only. Never stored in Atlas |
