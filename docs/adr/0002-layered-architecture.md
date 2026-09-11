# ADR-0002: Rules engine is framework-free and UI-independent

**Status:** Accepted — 2026-09-11

## Context
The Pathfinder rules layer is the part of this project with real intellectual value and the
part that must be provably correct. UI framework choices have a shelf life of a few years;
the rules of Pathfinder 1e do not change at all.

## Decision
`src/core/` contains no DOM access, no browser APIs, no framework imports and no CSS. It is a
pure TypeScript library. `src/ui/` may import from `src/core/`; the reverse is forbidden.

The UI framework decision is explicitly deferred to WP10.1, after the engine is complete.

## Consequences
- The engine can be tested in milliseconds with no browser
- A framework change costs a UI rewrite, not a rewrite of the valuable part
- The engine could later back a CLI, a Discord bot, or a different UI entirely
- Cost: some values must be passed explicitly rather than reached for globally
