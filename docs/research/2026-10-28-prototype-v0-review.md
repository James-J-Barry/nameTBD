# Prototype v0 review: three lenses

- **Status:** Current
- **Date:** 2026-10-28
- **Related:** [v0 prototype](../prototypes/v0-discovery-site.md) · [ADR-0001](../decisions/0001-registry-first-direction.md) · [Vision](../vision.md)

## Question

The owner was unsure which direction to take the project. We wanted to know what independent reviewers, each with a different stake, would conclude after using the v0 prototype.

## Method

Three AI agents (Claude Sonnet) each reviewed `prototype.html` from a single lens. Each reported bugs, things that work, 3–5 direction ideas, and a top recommendation with a two-week plan. They used Claude in Chrome against a locally served copy.

| Lens | Coverage | Caveats |
|---|---|---|
| Indie developer & strategy | Registered a fictional pre-release game end to end; opened the claim flow and the moderation queue | Didn't open game pages or the Tests tab |
| Player discovery | Store, search, tags, all AI filters and presets, three game pages, the claim form | Mobile width not tested (window resize didn't apply); simulation bar not used |
| Trust & safety | Traced the rules-engine source by hand | **Didn't use the browser.** Repros are derived from code and weren't confirmed in the UI |

These are AI reviewers, not real users. Treat their output as hypotheses to test with people, not as evidence of demand.

## Findings: bugs and logic issues

| Issue | Raised by | Notes |
|---|---|---|
| Missing AI disclosure is treated as "None"; filters and cards then imply "no AI" | Player, T&S (also builder) | → ADR-0005 |
| A claim with a made-up earlier date and a "distinctive" overlap hides any non-verified game (QUEUED) | T&S | → ADR-0004 |
| Pending (unreviewed) registrations count as collision partners, so registering first can push others to HELD | T&S (also builder) | Wider than test case 19 |
| Two look-alike games can both become VERIFIED; there's no cross-check between approved registrations | T&S | |
| Steam page creation date counts as independent, so a cheap Steam page "predates" the real developer | T&S | → ADR-0003 |
| Review brigading can push a rival to HELD via copy-signal or naming | T&S | → ADR-0004 |
| Appeals with junk "new evidence" re-queue; appeals without it are silently closed | T&S | |
| The timeline sweep skips session-touched cases, so attack tests never show as failures | T&S | |
| Featured hero ignores active filters (shows an AI-using game under "No AI anywhere") | Player | Confirmed in UI |
| Hash deep links only work on a fresh load; the URL doesn't update on navigation; Back doesn't work | Player, Dev | Confirmed in UI |
| Idea timeline names a person and the overlaps; it reads as an accusation despite the caveat | Player | → ADR-0004 |
| AI chips on cards (●/○) have no legend | Player | |
| "Next" is disabled with no indication of required fields | Dev | |
| Registration confirmation shows internal jargon ("QUEUED priority 90"), with no receipt or expected wait | Dev | |
| "Claim an existing record" lists non-public imported records, which would leak unannounced titles | Dev | Needs a Steam ownership check |

**Corrections to reviewer reports:**
- The player reviewer reported the internal rules-engine panel on player pages. It only appears in *internal preview*, which is what `#game=` deep links open. Players arriving from the store don't see it. The deep-link design is still confusing (see roadmap).
- The developer reviewer cited 32–52 day queue waits as an operating-cost signal. Those waits come from scripted mock dates, not measurement. The scaling concern still stands, but the numbers aren't evidence.

## Findings: what works

- Showing the AI-disclosure **source** (developer vs Steam) with a quote was called the strongest trust feature.
- Labelling evidence as **independently dated vs self-reported** was seen as honest.
- The moderator evidence view works well: side-by-side timelines, similarity scores, the shared-asset-pack check, right of response, and the capacity cap.
- Neutral wording ("absence implies nothing") read as honest, not evasive.
- The test table and timeline sweep make rule regressions visible.

## Findings: direction ideas

| Idea | Dev | Player | T&S |
|---|---|---|---|
| Self-serve timestamped registry as the core | ✅ top pick | | ✅ top pick (crypto timestamps) |
| Embeddable badge / permalink for store pages | ✅ distribution | ✅ (API + badge) | |
| Browser extension overlaying Steam pages | ✅ | ✅ top pick | |
| Public API / due diligence for publishers & platforms | ✅ revenue | ✅ | |
| Private evidence vault | ✅ | | ✅ |
| Partner with jams / festivals / platforms for dated records | | | ✅ |
| Steam curator lists, Discord bot | | ✅ | |
| Moderation only for disputes; drop auto-hiding and idea claims | ✅ | | ✅ top pick |
| AI-disclosure-only product | | (AI as hook) | ✅ considered |
| Keep the standalone discovery site as the core | ❌ | ❌ (too thin) | ❌ |

**Where they converged:** the standalone site shouldn't be the core. The registry, or the data, is the product, delivered where people already are. Human review should shrink to disputes.

**Where they differed:** what the hook is.
- Player: AI usage for players.
- Developer: pre-release provenance for developers, with publishers paying.
- T&S: minimal liability first.

## Implications

- Direction set to registry-first → [ADR-0001](../decisions/0001-registry-first-direction.md).
- Supporting decisions proposed → ADR-0003, ADR-0004, ADR-0005.
- Bugs that carry into any direction → [roadmap](../roadmap.md#now-next-2-weeks).
- Next research must involve **real** developers and players (see the roadmap experiments).
