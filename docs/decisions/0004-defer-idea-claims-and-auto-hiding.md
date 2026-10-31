# 0004 — Defer public idea claims and automatic hiding

- **Status:** Proposed
- **Date:** 2026-10-28
- **Related:** [ADR-0001](0001-registry-first-direction.md) · [ADR-0002](0002-neutral-factual-language.md) · [v0 prototype — known issues](../prototypes/v0-discovery-site.md#known-issues-and-rule-ambiguities)

## Context

In v0, several inputs from *other parties* could hide a game automatically (HELD or QUEUED):
- an idea claim meeting the evidence bar,
- a pending, unreviewed registration of a similar game,
- reviews naming it in copy-signal terms.

The trust-and-safety review showed each one is an abuse vector. A made-up dated claim, a cheap registration, or a review brigade can hide a competitor's game. Also, the "named as the original" rule pushed genuine originals out of the listing unless they were verified. On top of that, a public idea timeline that names a person and the overlapping elements reads to players as an accusation, even with a disclaimer.

## Decision (proposed)

For v1:

- **Remove idea claims and idea timelines** from the public product. Disputes about ideas go through the general dispute process and are not published.
- **No automatic hiding triggered by another party.** Claims, registrations and reviews about game X may *notify* moderators and X's developer. They never change X's visibility on their own.
- Signals from look-alikes (similarity, review language) go to **internal tooling only**, as prompts for people, not routing outcomes.
- Visibility changes only for non-accusatory reasons:
  - removed from Steam or confirmed takedown,
  - not a game,
  - duplicate,
  - content policy,
  - a person's decision after right of response.

## Consequences

- The abuse surface shrinks sharply, and so does moderator load.
- Real cases of idea reuse get no public signal in v1. We accept this until we have a dispute process and real usage data to design against.
- The v0 rules engine's HELD and QUEUED logic is kept for reference, but isn't carried into v1's public path.
