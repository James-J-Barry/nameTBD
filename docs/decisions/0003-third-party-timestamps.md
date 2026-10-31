# 0003 — Third-party timestamps as the basis of a record's dates

- **Status:** Proposed
- **Date:** 2026-10-28
- **Related:** [ADR-0001](0001-registry-first-direction.md) · [Research: v0 review](../research/2026-10-28-prototype-v0-review.md)

## Context

In v0 the badge's credibility depended on a moderator judging evidence that was mostly self-reported. Git history can be rewritten, file dates can be forged, and a cheap Steam page with someone else's art "predates" the real developer's page. The trust-and-safety review flagged self-reported dates as the weakest point of the system.

## Options considered

1. **Moderator judgment** (v0). Slow, doesn't scale, and makes us liable for calls we can't truly verify.
2. **Third-party cryptographic timestamps.** The developer's file or commit hashes are anchored with OpenTimestamps and/or an RFC 3161 timestamp authority at submission. Anyone can check the result independently.
3. **Partner-attested dates.** Game-jam hosts, festivals and platforms vouch for their own submission dates.

## Decision (proposed)

Use option 2 as the default for every record, and add option 3 as partnerships allow.

Every evidence item is shown with one of three date sources:
- **timestamped** (third-party proof),
- **publicly posted** (a dated public URL),
- **self-reported**.

The record proves *existence by a date*. It does not prove originality, and the UI says so in one line.

## Consequences

- Registration is instant and independently checkable. Moderators are not needed to establish dates.
- Timestamps only help from the moment of registration. That makes pre-release registration the key behaviour to encourage.
- We need a plain-language explainer for non-technical users (part of the roadmap experiments).
- Storage (ADR-0006): proofs are small and kept inline on each evidence item, in `registrations.evidence[].timestamp` (`method`, `state`, `proof` as BinData, `anchoredAt`). A worker job uses a partial index on `timestamp.state: "pending"` to upgrade OpenTimestamps proofs once they're anchored. See [data model](../architecture/data-model.md#registrations).
- To decide before accepting: OpenTimestamps vs RFC 3161 vs both; how to hash large builds (per-file manifest?); and how to verify a timestamp from a public URL.
