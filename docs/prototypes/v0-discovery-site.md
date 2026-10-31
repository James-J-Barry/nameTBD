# v0 — Discovery site + rules engine

- **Status:** Archived (reference). Direction superseded by [ADR-0001](../decisions/0001-registry-first-direction.md)
- **Date:** 2026-10-27
- **File:** [`prototype.html`](../../prototype.html)
- **Related:** [Research: v0 review](../research/2026-10-28-prototype-v0-review.md) · [Glossary](../glossary.md#v0-rules-engine-terms)

## What it demonstrates

A clickable, single-file prototype of a Steam-style discovery site that sells nothing and links out to stores. It shows:
- a rules engine that routes every game to one status,
- developer registration (including pre-release),
- idea claims,
- a moderator dashboard,
- live-editable thresholds,
- a date simulation,
- a test panel with expected statuses for 21 scenarios plus extras.

What is still useful under the new direction:
- **Moderation tooling.** The evidence summary, side-by-side timelines, similarity table and shared-pack check become the dispute tools.
- **Scenario catalogue.** The mock data is a ready-made adversarial test set.
- **Language discipline**, per ADR-0002.
- **Seed data.** `MOCK_DATA` maps field by field onto the Atlas collections (see the [mapping table](../architecture/data-model.md#mapping-from-v0-mock_data)). The planned `db:seed` script converts it.

## How to run

The file loads React, ReactDOM, Babel standalone and Tailwind from CDNs, so it needs internet access.

- Open `prototype.html` directly in a browser, or
- serve it with `python3 -m http.server 8765` and open `http://127.0.0.1:8765/prototype.html`.

| Deep link | Opens |
|---|---|
| `#moderation`, `#tests`, `#settings`, `#register`, `#claim` | That view |
| `#game=<id>` | Internal preview of a record, including non-public ones and the engine panel |

Deep links are read on page load only. See the known issues below.

## Structure

The file has three clearly separated script blocks:

1. **`<script id="mock-data">`** defines `MOCK_DATA`:
   - simulation range,
   - default settings (thresholds, priorities, capacity),
   - AI categories,
   - 32 records (each with `case`, `scenario`, `truth`, and an `expected` status timeline),
   - similarity pairs, claims, appeals, and scripted moderator decisions.
2. **`<script id="rules-engine">`** is a pure `Engine` module with no DOM. It's importable in Node via `module.exports`. Evaluation order:
   1. not known yet → `null`
   2. non-accusatory automatic outcomes → `ATTACHED` / `REJECTED`
   3. prior moderator rejection (appeal with new evidence → `QUEUED`)
   4. approved registration → `VERIFIED`
   5. a stakeholder waiting → `QUEUED`
   6. fact checks → `HELD` / `NOT_YET_ELIGIBLE` / `LISTED`

   Similarity collisions are applied afterwards, against a snapshotted reference set.
3. **`<script type="text/babel">`** holds the React UI: store, game page, registration, claim form, moderation, settings, simulation bar and tests (including a whole-timeline sweep).

## Test status at build time

- All 32 status tests and 3 claim-bar tests pass at the default date (2026-10-27).
- A day-by-day sweep (2026-09-01 → 2027-02-28) shows one intentional worst-case window: case 12b, 2026-09-01 → 2026-10-02 (see issue 3 below).

## Known issues and rule ambiguities

Builder's list, from writing the rules:

1. **"Named as the original" pushes genuine originals out.** An original named in a look-alike's reviews drops to HELD unless VERIFIED. Cases 3a and 10a only pass because they are VERIFIED. Disabling the check lists 12b, which is a worst case.
2. **"Collision with a listed/registered game" is circular.** Resolved with a snapshotted reference set: registered, verified, or listable apart from collisions.
3. **Look-alike launches first.** It is LISTED until the original has a footprint on the site (case 12b).
4. **DLC vs "exactly one status".** Added `ATTACHED`.
5. **HELD is silent, yet held developers can appeal.** They have no way to know they are held.
6. **Pending registrations count for collisions.** Registering first can hide other games (case 19).
7. **"Open claim".** Only bar-meeting, undecided claims count.
8. **Effect of an upheld idea claim is unspecified.** The game is re-evaluated normally.
9. **VERIFIED is decided before triggers.** New claims or matches against a verified game don't queue it.
10. **Approve/Reject mean different things per trigger type.**
11. **Priority order:** asset match 100 > pre-release registration 90 (+5 if a look-alike is live) > claim 70 > appeal 60 > registration 50. Ties go to the oldest.
12. **Similarity uses the max of four signals,** not a weighted blend.
13. **No "Not disclosed" AI level** (→ ADR-0005).
14. **Assorted:**
    - Early Access age counts from the EA date.
    - The capacity cap warns but doesn't block.
    - Automatic rejections can't be appealed.
    - Claims can only target public games.

Added by the [v0 review](../research/2026-10-28-prototype-v0-review.md):

- Claims with made-up dates hide targets.
- Two look-alikes can both be VERIFIED.
- Steam page dates are treated as independent.
- Review brigading can push a game to HELD.
- The sweep skips session-touched cases.
- The featured hero ignores filters.
- No `hashchange` handling and no URL updates.
- The registration receipt shows jargon.
- The existing-record picker leaks non-public records.
