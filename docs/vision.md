# Vision

- **Status:** Current
- **Date:** 2026-10-28
- **Related:** [ADR-0001](decisions/0001-registry-first-direction.md) · [Research: v0 review](research/2026-10-28-prototype-v0-review.md) · [Roadmap](roadmap.md)

> Working name: **Firstmark** (placeholder; the repo is `nameTBD`).

## The problem

Generative AI has made it cheap to produce a look-alike of a game within days of its first public showing — sometimes before the original has even launched. Indie developers have no easy, credible way to prove *"we showed this first"*, and players have no reliable way to know how much of a game was made with AI. Today that evidence is scattered across devlogs, social posts, git histories and store pages, and nobody stitches it together until a dispute is already happening.

## What we are building

**A free, self-serve provenance registry for games, plus structured AI-usage disclosure data — delivered where developers and players already are.**

The product has four layers, in priority order:

| # | Layer | What it is | Who it serves |
|---|---|---|---|
| 1 | **Dated registry** (the core) | A developer submits build hashes, concept art, teaser links, devlogs, repo history — before or after release. The record is **public and dated the moment it is submitted**, with dates anchored by third-party timestamps (e.g. OpenTimestamps / RFC 3161) rather than our say-so. Each record gets a permanent public page. | Indie developers, especially **pre-release** — the moment they are most exposed and the thing no storefront offers |
| 2 | **AI-usage disclosure data** | Ten categories (art, music, SFX, voice, writing, design, code, translation, runtime generation, marketing) × `None` / `Assisted` / `Generated` / `Not disclosed`, with the source (developer-stated vs parsed from Steam) and a quoted statement. | Players who want to avoid (or seek out) specific kinds of AI use |
| 3 | **Distribution surfaces** | An embeddable *"Registered [date]"* badge and permalink for store pages and press kits; a browser extension that overlays registry + AI data on Steam store pages; a read-only public API. A thin browse/search site exists but is a **view of the data, not the product**. | Developers (social proof), players (info at the point of purchase), press/curators |
| 4 | **Human layer** (deliberately small) | People handle **disputes** and an **optional, slower "Verified" review**. "Verified" means *the dated record was checked*, never *this game is the original*. | Developers in a dispute; later, publishers/platforms doing due diligence |

**How it's built:** a Node API and worker over **MongoDB Atlas**. Atlas Search powers browse and the AI filters, Atlas Vector Search powers the internal look-alike prompts, and change streams drive timestamping and syncing. Evidence files live in object storage, keyed by hash. See [architecture](architecture/README.md) and [ADR-0006](decisions/0006-mongodb-atlas.md).

### Why this shape

Three independent reviews of the v0 prototype ([research note](research/2026-10-28-prototype-v0-review.md)) converged on the same points:

- **A standalone storefront has a cold-start problem.** A curated catalogue of a few dozen games gives players no reason to visit. Data delivered on Steam pages, badges and APIs does not depend on our traffic.
- **Human review of every registration does not scale** and makes developers wait for the core value. Self-serve, timestamped records give value on day one.
- **Automatic hiding and public idea-claims carry the biggest abuse and legal risk.** Claims and pending registrations could be used to hide competitors' games. Removing these from the default path removes that attack surface.
- **The hook differs by audience.** Players care most about "does this use AI art or voice?"; developers care most about "can I prove we were first?". The registry serves developers; the AI data serves players; each makes the other more credible.

## Principles (non-negotiable)

1. **Record originals' evidence; never label anything a copy.** No page, badge, API field or UI string says a game is a "copy", "clone" or "fake". We state dated facts. → [ADR-0002](decisions/0002-neutral-factual-language.md)
2. **Absence is neutral.** Not being registered, or not appearing, implies nothing about a game. We never imply completeness.
3. **Dates come from third parties wherever possible.** Self-reported dates are always labelled as self-reported. → [ADR-0003](decisions/0003-third-party-timestamps.md)
4. **Automation collects and routes facts; people make judgments.** No automated verdicts, and no automatic hiding of another party's game based on unverified input. → [ADR-0004](decisions/0004-defer-idea-claims-and-auto-hiding.md)
5. **"Unknown" is not "none".** Missing AI disclosure is shown as *Not disclosed*, never as *None*. → [ADR-0005](decisions/0005-ai-disclosure-not-disclosed-level.md)
6. **We don't sell games.** Every purchase link goes to Steam or the developer's own store.

## What we are *not* doing (for now)

- Running a full storefront-style catalogue as the main product.
- Public idea-theft claims / idea timelines (designed in v0; deferred — see ADR-0004).
- Automatic HELD/QUEUED hiding of games triggered by other parties' claims, registrations or reviews.
- Judging originality. We publish dated records and disclosures; disputes go to people, and outcomes are stated as facts.

## Business model (hypotheses to test, not commitments)

- **Free for developers**, always, for the core registry. Their records are the dataset.
- **Paid due-diligence access** for publishers, funds, festivals and platforms: "before you sign or feature this game, who showed it first and what does the record say?" (API, bulk checks, alerts).
- **Optional paid "Verified" review** or pro features (auto-ingest of git pushes/devlogs into a private, exportable evidence vault) — only if developers ask for them.

## Open questions

- Will developers put the badge on their store pages? (Is the record socially valuable without platform endorsement?)
- Will players install an extension, or do they only want a curated list (e.g. a Steam curator page)?
- Which third-party timestamping approach is credible to non-technical people, and how do we explain it in one line?
- Can game-jam hosts and festivals supply independently dated records directly (partnerships)?
- What is the dispute process exactly (counter-notice flow, response windows, what gets published)?
- Final product name.

## How we will know it is working

See [roadmap.md](roadmap.md) for the current experiments. Headline signals:

- Developers: % of invited devs who complete a registration; % who place the badge/link on their store page or press kit.
- Players: % who keep the extension installed after two weeks; filter usage on the AI categories.
- Operations: disputes per 100 registrations; moderator minutes per dispute.
