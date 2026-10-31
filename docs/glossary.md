# Glossary

- **Status:** Current
- **Date:** 2026-10-30

Terms used across docs, code and UI. Add new terms in the same change that introduces them.

## Core concepts

| Term | Meaning |
|---|---|
| **Provenance record** | A dated, public record of evidence a developer submitted for a game (hashes, links, art, repo history). Created at submission in the registry-first direction. |
| **Registration** | The act of submitting a provenance record. Can happen **pre-release** (before a Steam page exists). |
| **Independently dated** | Evidence whose date a third party can confirm: a public post/video/jam entry, a store page, or a cryptographic timestamp. Shown as such. |
| **Self-reported date** | A date only the submitter vouches for (git history, local files, concept art). Always labelled. |
| **Third-party timestamp** | Cryptographic proof that a given hash existed at a given time (e.g. OpenTimestamps, RFC 3161). Proves *existence by a date*, not originality. |
| **Verified** | (Registry-first meaning.) A person checked that a record's dates and evidence are consistent. It does **not** mean "this game is the original". In v0 this was "Verified Provenance", granted only by a moderator. |
| **Look-alike** | A game whose title, description, screenshots or keywords score highly similar to another record. Neutral, internal term. Never used publicly about a specific game. |
| **Non-original** | Test/ground-truth label only (mock data). Never shown publicly. |
| **Absence is neutral** | Principle: not being registered or listed says nothing about a game. |

## AI-usage disclosure

| Term | Meaning |
|---|---|
| **Categories** | Art & visuals, Music, Sound effects, Voice, Writing & narrative, Game design & mechanics, Code, Translation, Live/runtime generation, Marketing materials. |
| **None** | The developer or the Steam disclosure positively states no AI use in that category. |
| **Assisted** | AI used, output human-edited. |
| **Generated** | AI output used substantially as-is. |
| **Not disclosed** | No information. Distinct from None (see ADR-0005). |
| **Source** | `developer-stated` or `parsed from Steam disclosure`, shown with a short quote. |

## Data & infrastructure

| Term | Meaning |
|---|---|
| **MongoDB Atlas** | Managed MongoDB service; our primary datastore (ADR-0006). |
| **Atlas Search** | Full-text and faceted search built into Atlas (`$search`, `$searchMeta`). Powers browse and the AI filters. |
| **Atlas Vector Search** | Nearest-neighbour search over embeddings (`$vectorSearch`). Powers internal look-alike prompts only. |
| **Embedding** | Numeric vector representing a description or image, used for similarity. |
| **Similarity prompt** | An internal `similarity_prompts` document suggesting two records a moderator may want to look at. Never affects visibility. |
| **Change stream** | Atlas feed of inserts/updates that the worker consumes to run jobs. |
| **Resume token** | Position in a change stream, persisted so the worker never misses or repeats events. |
| **Time series collection** | Atlas collection type optimised for timestamped measurements (`review_snapshots`). |
| **Public projection** | The single server-side definition of which fields may leave the API. |
| **`publicId`** | Short permanent id used in record and dispute permalinks (e.g. `rec_7Hq2Kd`). |
| **Date source** | Per evidence item: `timestamped`, `publicly_posted` or `self_reported` (ADR-0003). |
| **Lookalike share** | Internal successor to v0's copy-signal score, stored in `review_snapshots`. |

## v0 rules-engine terms

These describe the [v0 prototype](prototypes/v0-discovery-site.md). Several are deprecated by ADR-0004.

| Term | Meaning |
|---|---|
| **LISTED** | Public, no badge. |
| **VERIFIED** | Public, with badge; requires a moderator decision. |
| **NOT_YET_ELIGIBLE** | Hidden; too new or too few reviews; re-checked automatically. |
| **HELD** | Hidden and silent; ambiguous signals, nobody waiting. |
| **QUEUED** | Hidden; a stakeholder is waiting on a moderator. |
| **REJECTED** | Hidden; internal reason; appealable. |
| **ATTACHED** | v0 addition: DLC shown on its parent's page, not evaluated alone. |
| **Copy-signal score** | Share of reviews claiming a game reproduces a *specific* other game. Genre comparisons ("like X") are excluded. Metric name only. |
| **Similarity collision** | Max of title/description/screenshot/keyword similarity ≥ threshold against a game in the reference set. |
| **Reference set (R)** | Games that count as collision partners: registered, verified, or otherwise listable. |
| **Stakeholder waiting** | A registration, claim meeting the evidence bar, asset/code match against a registered build, or appeal. Routes to QUEUED. |
| **Idea claim** | "This game used my publicly shown concept." Lower-certainty layer, shown only as a dated timeline. Deferred by ADR-0004. |
| **Evidence bar** | A claim needs a dated source link, a source date earlier than the target's first public record, and a specific distinctive overlap. |
| **Right of response** | Notifying the accused party, with a response window, before a decision. |
