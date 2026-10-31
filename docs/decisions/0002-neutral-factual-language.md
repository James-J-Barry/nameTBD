# 0002 — Neutral, factual language: label originals' evidence, never copies

- **Status:** Accepted
- **Date:** 2026-10-28
- **Related:** [Vision — principles](../vision.md#principles-non-negotiable) · [Glossary](../glossary.md)

## Context

Calling a game a "copy", "clone" or "fake" is an accusation. It carries defamation risk if it's wrong, and it invites abuse: competitors would try to get that label put on rivals. Automated systems will sometimes be wrong (shared asset packs, game-jam parallel development, reverse confusion where the original launches later).

## Decision

- No page, badge, API field, notification or UI string states or implies that a specific game is a copy, clone or fake.
- We publish **dated facts** only. Example: "Steam page created Oct 9, 2026", or "Record registered Sep 20, 2026; teaser publicly posted Jun 18, 2026".
- **Absence is neutral.** We never imply completeness or that unregistered games are suspect.
- Quoted user content (e.g. reviews) may contain such words. It is shown only in internal tools, labelled as unverified.
- Internal and test terminology uses *look-alike* and *non-original*. These terms are never shown publicly about a specific game.

## Consequences

- UI copy reviews must check for the banned words. The v0 prototype was checked this way.
- Dispute outcomes are published as dated timelines, not verdicts.
- Some players will find the neutral wording less satisfying than a "clone" label. We accept that trade.
