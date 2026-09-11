# Pathfinder 1e Player Character Sheet

A web-based, editable Pathfinder 1st Edition player character sheet, styled as a book, that
computes every derived value the rules allow and shows its working.

**Status:** planning. No code yet.

## Planning documents

| Doc | What it covers |
|---|---|
| [00 — Vision and Scope](docs/00-vision-and-scope.md) | What this is, what it isn't, decisions taken, honest sizing |
| [01 — Architecture](docs/01-architecture.md) | Layering, project layout, the two data shapes, testing strategy |
| [02 — Rules Engine](docs/02-rules-engine.md) | The Effect model, stacking rules, every formula |
| [03 — Design Language](docs/03-design-language.md) | The book aesthetic: tokens, parchment, typography, print, a11y |
| [04 — Roadmap](docs/04-roadmap.md) | 16 milestones broken into numbered work packages |
| [05 — Learning Path](docs/05-learning-path.md) | What to learn, and when, from a standing start |

Architecture decisions are recorded in [docs/adr/](docs/adr/).

## The three rules

1. **`src/core/` never imports from `src/ui/`.**
2. **Never store a derived value in a character file.**
3. **Every stat accepts a custom modifier with a note.** No engine covers all of Pathfinder.

## Next step

WP0.0 — the throwaway spike. See [the roadmap](docs/04-roadmap.md).
