# CLAUDE.md

Provenance registry + AI-usage disclosure for games (working name Firstmark). Read `docs/vision.md` first for the current direction.

## Documentation system

Docs live in `docs/`. Conventions are in `docs/README.md`. Keep them current as part of any change:

- When a decision is made or reversed, add `docs/decisions/NNNN-slug.md` from `docs/templates/decision.md` and update `docs/decisions/README.md`. Don't edit the substance of accepted records; supersede them instead.
- When something is learned (reviews, tests with users, experiments), add `docs/research/YYYY-MM-DD-slug.md` and update `docs/research/README.md`.
- When a prototype is added or changed, update its doc in `docs/prototypes/`, including its known issues.
- Keep `docs/vision.md` and `docs/roadmap.md` describing the current state. Move roadmap items when work starts or finishes.
- Add new terms to `docs/glossary.md`.
- Use absolute dates (YYYY-MM-DD).

## Product rules that apply to code, UI copy and docs

- Never state or imply that a specific game is a "copy", "clone" or "fake" (ADR-0002). Internal terms are "look-alike" and "non-original".
- Absence is neutral. Never imply the catalogue is complete.
- Missing AI disclosure is "Not disclosed", never "None" (ADR-0005).
- All example data is fictional: no real games, studios or people.

## Data layer: MongoDB Atlas (ADR-0006)

- The source of truth is `docs/architecture/data-model.md`. Change it in the same change as any collection, field or index. Search, Vector Search and role changes also go in `docs/architecture/atlas-setup.md`.
- Clients never connect to Atlas directly. All access goes through the API or worker, using the official driver.
- `registrations` and `audit_events` are append-only. State changes are written in the same transaction as their audit event.
- Fields under `internal.*` and the `similarity_prompts` collection never leave the API (public projection).
- Never commit connection strings or credentials.

## Prototypes

Single-file HTML (React + Tailwind via CDN), with mock data and logic in separate, commented `<script>` blocks. Keep the logic DOM-free so it can be tested in Node.
