# 0001 — Registry-first direction; the discovery site becomes a view

- **Status:** Accepted
- **Date:** 2026-10-28
- **Related:** [Vision](../vision.md) · [Research: v0 review](../research/2026-10-28-prototype-v0-review.md) · [v0 prototype](../prototypes/v0-discovery-site.md)

## Context

v0 was designed as a Steam-style discovery site. Every game passed through a rules engine and every registration waited for a moderator before it had any value. Three independent reviews of v0 (indie developer and strategy; player discovery; trust and safety) found:

- A curated catalogue is too thin to attract players. A storefront has a cold-start problem we can't solve early.
- Mandatory human review of every registration doesn't scale. It also delays the core value for developers by weeks.
- The most valuable, unique thing is **pre-release, dated provenance**, which no storefront offers.
- Players want AI-usage information **where they already shop**.

## Options considered

1. **Keep the discovery site as the product** and improve it. Pros: the prototype exists, and it's a clear destination. Cons: cold start, moderation load, and thin catalogue.
2. **Registry-first.** Self-serve dated records with public permalinks and badges. AI data goes out through an overlay or API. The site is a thin view. Pros: value on day one, distribution through developers' own pages, small human layer. Cons: a bare record may seem weak without endorsement, and the extension depends on Steam's page structure.
3. **AI-disclosure-only product.** Pros: low legal exposure and a clear player hook. Cons: crowded space, self-reported data, and it drops the developer-protection mission.
4. **Private evidence vault only** (no public records). Pros: no liability. Cons: no network effect and weak public value.

## Decision

Option 2. The core product is a **free, self-serve provenance registry**, plus **structured AI-usage disclosure data**. It is distributed through permalinks and badges, a Steam overlay extension, and a public API. The discovery site remains as a view over the data. Human review covers disputes and an optional "Verified" check only.

## Consequences

- Registration produces a public dated record immediately. Moderation leaves the critical path.
- v0's rules engine and moderation tooling become the basis of the **dispute process**, not the main flow.
- We need a credible timestamping story (ADR-0003) and a dispute process (future record).
- Success is measured by developer adoption and badge placement, not site traffic.
- v0 stays in the repo as a reference. A v1 prototype will demonstrate this direction.
