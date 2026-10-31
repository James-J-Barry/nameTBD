# 0005 — Add a "Not disclosed" AI-usage level

- **Status:** Proposed
- **Date:** 2026-10-28
- **Related:** [Glossary — AI-usage disclosure](../glossary.md#ai-usage-disclosure) · [Research: v0 review](../research/2026-10-28-prototype-v0-review.md)

## Context

v0 had three levels: None, Assisted and Generated. A missing disclosure counted as None. Steam disclosures are often sparse, so many games would pass a "No AI art" filter only because nobody said anything. The cards then read "none disclosed", which players take as "clean". Two of the three reviewers flagged this as the biggest trust risk for the AI filters. It is also a legal risk, because we would be positively asserting something we don't know.

## Decision (proposed)

- Add a fourth level, **Not disclosed**, used whenever no source states the category.
- Filters that require "None" **exclude Not disclosed by default**, with an "include undisclosed" toggle.
- Cards and pages show "Not disclosed" explicitly. They never collapse it into None.
- Every non-default value shows its source (developer-stated or parsed from Steam) and a quote.

## Consequences

- Filtered results get much smaller at first. That honestly reflects the data and gives developers a reason to disclose.
- Storage (ADR-0006): every `games` document always carries all 10 `ai.<category>` sub-documents. `level` is one of `none | assisted | generated | not_disclosed`, and the default is `not_disclosed`. Filters run on Atlas Search `token` fields. See [data model](../architecture/data-model.md#games).
- Steam disclosure parsing has to distinguish "says no AI" from "says nothing" per category.
