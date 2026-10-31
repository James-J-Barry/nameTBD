# Documentation

This folder is the source of truth for **what we are building, why, and what we have learned**. Code says *how*; these docs say *why* and *what next*.

## Map

| Doc | What it answers | Changes when… |
|---|---|---|
| [vision.md](vision.md) | What is the product, who is it for, what principles constrain it? | A decision record changes the direction |
| [roadmap.md](roadmap.md) | What are we doing now, next, later — and how will we know it worked? | Work starts or finishes, or an experiment reports back |
| [glossary.md](glossary.md) | What do our terms mean exactly? | A new term appears in code, UI, or another doc |
| [architecture/](architecture/) | How is the system built — components, MongoDB Atlas data model, environment setup? | A technical decision record is accepted, or the data model changes |
| [decisions/](decisions/) | Why did we choose X over Y? (one record per decision) | A meaningful choice is made — never edited after acceptance, only superseded |
| [research/](research/) | What did we observe? (reviews, user interviews, experiments) | We learn something — one dated note per study |
| [prototypes/](prototypes/) | How does each prototype work, how to run it, what are its known issues? | A prototype is built, changed, or retired |
| [templates/](templates/) | Starting points for new docs | Rarely |

## How the pieces fit

```
research note  ──►  decision record  ──►  vision / roadmap updated
 (what we saw)       (what we chose)        (current state of the plan)
                           │
                           └──►  prototype or spec  ──►  new research note
```

1. **Learn** something → write a research note in `research/` (dated, never rewritten later).
2. **Decide** something because of it → add a decision record in `decisions/` that links to the research.
3. **Update** `vision.md` and `roadmap.md` so they describe the *current* plan and link to the decision.
4. **Build** → document the prototype in `prototypes/`; test it → back to step 1.

`vision.md` and `roadmap.md` are living documents (always current). Research notes and accepted decision records are historical (append-only; supersede rather than edit).

## Conventions

- **Front matter.** Every doc starts with a small header block: `Status`, `Date` (YYYY-MM-DD, absolute — never "last week"), and `Related` links.
  - Decision status: `Proposed` → `Accepted` → (`Superseded by NNNN` | `Deprecated`).
  - Other docs: `Draft` | `Current` | `Archived`.
- **Naming.**
  - Decisions: `decisions/NNNN-short-slug.md`, numbered sequentially. Never reuse a number.
  - Research: `research/YYYY-MM-DD-short-slug.md`.
  - Prototypes: `prototypes/vN-short-slug.md`, matching the prototype file.
- **Language rule applies to docs too.** The product never says a game is a "copy", "clone" or "fake". In docs, prefer *look-alike*, *non-original*, or *later record* so the habit carries into UI copy. Quoting a user review verbatim is fine.
- **Fictional data only** in examples. Do not name real games, studios, or people as examples of anything.
- **Link, don't repeat.** If something is explained elsewhere, link to it.
- **Short beats complete.** A one-page decision record that exists is better than a perfect one that doesn't.

## Adding a doc

| You want to… | Do this |
|---|---|
| Record a decision | Copy `templates/decision.md` to `decisions/NNNN-slug.md` (next free number), fill it in, add it to the index in `decisions/README.md`, update `vision.md` / `roadmap.md` if affected |
| Record what you learned | Copy `templates/research-note.md` to `research/YYYY-MM-DD-slug.md`, add it to `research/README.md` |
| Document a new prototype | Copy `templates/prototype.md` to `prototypes/vN-slug.md`, add it to `prototypes/README.md` |
| Introduce a new term | Add it to `glossary.md` in the same change |
| Add/change a collection, field or index | Update `architecture/data-model.md` (and `atlas-setup.md` for Search/Vector indexes or roles) in the same change as the code |
