# CLAUDE.md

Project context for Claude Code. This file is version-controlled, so it travels with the
repository to any machine — unlike Claude's per-machine memory.

## What this is

A web-based, editable Pathfinder 1st Edition **player** character sheet, styled to look like a
printed book, which computes every derived value the rules allow and can explain each one.
Player-facing only: no Game Master features of any kind.

Full planning set is in [`docs/`](docs/). Read [`docs/01-architecture.md`](docs/01-architecture.md)
and [`docs/02-rules-engine.md`](docs/02-rules-engine.md) before writing code in `src/core/`.

## Who I'm working with

John is learning web development from a standing start — no prior JavaScript, TypeScript or
framework experience as of September 2026. He is technically capable and thinks in terms of
maintainability and system design, so explain the web/TypeScript specifics without explaining
what a data model or a dependency is.

Deliberate learning order, inverted from the usual: TypeScript and testing first against the
headless rules engine, then HTML and CSS, then React last.

## Decisions already taken — do not relitigate without cause

See [`docs/adr/`](docs/adr/) for the reasoning.

- **TypeScript + Vite + Vitest.** The UI framework choice is deliberately deferred to WP10.1.
- **`src/core/` is framework-free.** No DOM, no browser APIs, no framework imports. `src/ui/`
  may import from `src/core/`; the reverse is forbidden.
- **Everything is an `Effect`** — a typed bonus aimed at a typed `StatKey`. Races, classes,
  feats, items, spells, conditions and hand-typed player notes all emit the same shape.
- **`resolve()` returns `{total, applied, suppressed}`**, never a bare number. This is what
  makes "why is my AC 18?" a feature rather than a refactor.
- **Character files store choices only, never derived values.** No AC, no attack bonus, no
  skill totals in a saved character.
- **Every stat accepts a custom modifier with a note.** No engine covers all of Pathfinder;
  the escape hatch is designed in, not bolted on.
- **Engine first, content later.** Races, classes and feats are JSON data grown over time.
  Adding a race must require zero changes under `src/core/derive/`.
- **Warnings, never blocks.** The engine never refuses to compute.
- **Browser-local persistence + JSON import/export.** No backend, no accounts.
- **No string formula interpreter.** Scaling effects use a fixed set of named functions.

## Working conventions

- Work is referenced by work-package number from [`docs/04-roadmap.md`](docs/04-roadmap.md),
  e.g. "let's do WP2.2". Progress is tracked in the roadmap artifact:
  https://claude.ai/code/artifact/a54b3d88-f3c7-4c67-9d6d-294c0cd99a20
- Conventional commits (`feat:`, `fix:`, `docs:`), small and single-purpose, on branches.
- New significant decisions get an ADR in `docs/adr/`, numbered sequentially.
- Tests accompany rules code. The golden fixtures in `tests/fixtures/` are the regression net
  for correctness — treat a fixture change as a claim that needs justifying against the
  rulebook.

## Environment

Developed across a Windows machine and a Fedora laptop. Keep everything path-agnostic: no
hardcoded absolute paths, no OS-specific shell in npm scripts, LF line endings enforced by
`.gitattributes`.
