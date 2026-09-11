# ADR-0003: All modifiers are Effects, resolved with full provenance

**Status:** Accepted — 2026-09-11

## Context
Pathfinder stacking rules are typed: two enhancement bonuses do not stack, two dodge bonuses
do. A naive implementation adds numbers at each call site and gets this wrong in ways that are
invisible until a player is mid-combat. Separately, the sheet must be able to explain any
number it displays.

## Decision
Races, classes, feats, items, spells, conditions and hand-typed player notes all emit uniform
`Effect` objects targeting a typed `StatKey`. A single `resolve()` function applies the
stacking rules.

`resolve()` returns `{ total, applied, suppressed }` — never a bare number.

## Consequences
- Stacking correctness lives in one heavily tested function instead of forty call sites
- "Why is my AC 18?" becomes a UI feature over data the engine already produces
- Toggleable buffs and conditions need no special handling — they are effects with `active`
- Adding new content sources requires no engine changes
- Cost: more verbose than adding numbers, and every stat must be a `StatKey`. Accepted.

## Rejected alternative
Returning bare numbers and reconstructing explanations later. Retrofitting provenance would
touch every derivation function; the cost is near zero now and large later.
