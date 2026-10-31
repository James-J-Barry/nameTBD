# MongoDB Atlas setup

- **Status:** Draft
- **Date:** 2026-10-30
- **Related:** [ADR-0006](../decisions/0006-mongodb-atlas.md) · [Data model](data-model.md) · [Architecture](README.md)

How to provision and configure Atlas for this project. Atlas tiers, limits and feature availability change over time. **Check the current Atlas documentation when provisioning** and update this page if anything differs.

## Environments

| Environment | Atlas project | Cluster tier | Data |
|---|---|---|---|
| `dev` | `firstmark-dev` | Free (M0) or Flex | Fictional seed data (from v0 `MOCK_DATA`) |
| `staging` | `firstmark-staging` | Small dedicated (e.g. M10) | Fictional seed data + anonymised fixtures |
| `prod` | `firstmark-prod` | Dedicated, sized from load tests | Real |

- Use one Atlas **project** per environment, so network rules, users and alerts are isolated.
- The database name is `firstmark` in every environment.
- Pick a region close to the API hosting. If data residency matters for EU developers' contact data, choose the region deliberately before launch. Moving later is costly.
- Free and Flex tiers lack features that production needs (backups, private endpoints, dedicated Search resources). Treat them as dev-only.

## Repository layout (planned)

```
db/
  schemas/            $jsonSchema validator per collection (games.json, registrations.json, …)
  indexes/            standard index definitions per collection
  search/             Atlas Search + Vector Search index definitions (JSON)
  migrations/         numbered, idempotent scripts; bump schemaVersion
  seed/               converts v0 MOCK_DATA into documents for dev/staging
```

A `db:apply` script, run from CI or locally with the environment's connection string, does the following:
1. Creates the collections, including `review_snapshots` as a **time series** collection.
2. Applies `$jsonSchema` validators (`validationLevel: "strict"`, `validationAction: "error"`).
3. Creates the standard indexes.
4. Creates or updates the Atlas Search and Vector Search indexes, through the driver's search index helpers or the Atlas Admin API / CLI.

It must be idempotent.

## Search indexes

**`games_search`** (Atlas Search on `games`)

```json
{
  "mappings": {
    "dynamic": false,
    "fields": {
      "title":       [{ "type": "string" }, { "type": "autocomplete" }],
      "description": { "type": "string" },
      "tags":        [{ "type": "string" }, { "type": "token" }],
      "visibility":  { "type": "token" },
      "verification": { "type": "document", "fields": { "state": { "type": "token" } } },
      "steam": { "type": "document", "fields": { "releasedAt": { "type": "date" } } },
      "reviewSummary": { "type": "document", "fields": { "count": { "type": "number" } } },
      "ai": { "type": "document", "fields": {
        "art":         { "type": "document", "fields": { "level": { "type": "token" } } },
        "music":       { "type": "document", "fields": { "level": { "type": "token" } } },
        "sfx":         { "type": "document", "fields": { "level": { "type": "token" } } },
        "voice":       { "type": "document", "fields": { "level": { "type": "token" } } },
        "writing":     { "type": "document", "fields": { "level": { "type": "token" } } },
        "design":      { "type": "document", "fields": { "level": { "type": "token" } } },
        "code":        { "type": "document", "fields": { "level": { "type": "token" } } },
        "translation": { "type": "document", "fields": { "level": { "type": "token" } } },
        "runtime":     { "type": "document", "fields": { "level": { "type": "token" } } },
        "marketing":   { "type": "document", "fields": { "level": { "type": "token" } } }
      } }
    }
  }
}
```

The mapping is static (`dynamic: false`), so `internal.*` and `embeddings` are never indexed for public search.

**`games_description_vec`** and **`media_image_vec`** (Atlas Vector Search)

```json
{ "fields": [
  { "type": "vector", "path": "embeddings.description", "numDimensions": 1024, "similarity": "cosine" },
  { "type": "filter", "path": "visibility" }
] }
```

Set `numDimensions` to whatever the chosen embedding model outputs. `media_image_vec` is the same on `media.embedding`, with a filter on `gameId`.

## Database users and roles

Create **custom roles** scoped to the `firstmark` database. No application uses `atlasAdmin` or `readWriteAnyDatabase`.

| Role | Granted to | Privileges |
|---|---|---|
| `app_api` | API service | `find/insert/update` on `games`, `studios`, `registrations`, `disputes`, `media`, `api_clients`. **`find/insert` only** on `audit_events`. `find` on `rule_settings`, `similarity_prompts`, `review_snapshots` |
| `app_worker` | Worker service | `find/update` on `games`, `registrations` (timestamp fields enforced in code), `disputes`. `find/insert/update` on `similarity_prompts`, `media`. `insert` on `review_snapshots`. **`find/insert` only** on `audit_events`. `changeStream` + `find` on watched collections |
| `app_migrate` | CI migration job | Collection/index/validator management. Used only by `db:apply` |
| `readonly_analyst` | People, ad hoc | `find` on everything except `studios.contact` (use a view that projects it out) |

- Give each service its own database user, with credentials in the host's secret manager. Never commit connection strings. `.env` files are git-ignored.
- People get Atlas UI access through Atlas organisation/project roles with MFA, not shared database users.

## Network and encryption

- **dev:** IP access list limited to developer IPs. Never use `0.0.0.0/0`.
- **staging/prod:** private endpoint or VPC peering to the API/worker hosting, where the host supports it. Otherwise use a tight IP access list of the host's egress IPs.
- TLS is always on (Atlas default). Encryption at rest is on by default. Consider customer-managed keys for prod.
- **Contact PII** (`studios.contact`) uses field-level encryption: Queryable Encryption if we need equality lookups by email, CSFLE otherwise. That way database operators and backups never see plaintext emails.

## Backups, retention and monitoring

- **prod:** continuous cloud backup with point-in-time recovery. Run a restore drill before launch and record it as a research note.
- `audit_events` and `registrations` are kept indefinitely: they *are* the product's evidence. Consider Online Archive for old `review_snapshots` and `audit_events` if storage costs grow.
- Atlas alerts: replication lag, connections, disk, Search index build failures, and slow queries via Performance Advisor. Route them to the team's alert channel.
- Track Search and Vector Search query latency separately from CRUD.

## Change streams (worker)

The worker opens change streams on `registrations` (inserts), `games` (inserts, and updates to `description`/`media`), and `disputes` (state changes).

- It persists the **resume token** after each processed event (in a `worker_state` collection), so restarts don't miss or double-process events.
- Handlers must be idempotent, keyed by `(collection, documentId, eventType, clusterTime)`.
- Scheduled jobs (proof upgrades, Steam sync, dispute deadlines) run on the worker's scheduler. They query the partial indexes listed in the data model.

## Local development

- Point the API at the `dev` Atlas cluster. For offline work, use a local MongoDB via the Atlas CLI local deployment (`atlas deployments setup --type local`), which also supports Atlas Search and Vector Search locally.
- `npm run db:apply && npm run db:seed` gives a fresh database with the fictional v0 scenarios.
