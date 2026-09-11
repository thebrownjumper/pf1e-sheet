# 04 — Roadmap

Sixteen milestones, each broken into work packages. A work package is sized to be finishable
in one or two sittings and has a definition of done you can check without judgement.

Reference a package by its number when you want to start it, e.g. "let's do WP2.2".

---

## M0 — Groundwork  *(no Pathfinder content yet)*

| # | Work package | Done when |
|---|---|---|
| WP0.0 | **Throwaway spike.** One HTML file, hardcoded stats, a Dex box that updates an AC box. Delete it afterwards. | You have felt the shape of the problem |
| WP0.1 | Install Node, create a Vite + TypeScript project, run the dev server | `npm run dev` serves a page |
| WP0.2 | Git repo, `.gitignore`, first commit, branch-per-feature habit | `git log` shows a sensible first commit |
| WP0.3 | Add Vitest. Write a deliberately failing test, then make it pass | `npm test` runs in watch mode |
| WP0.4 | ESLint + Prettier, format-on-save | Saving a file reformats it |
| WP0.5 | Create the folder structure from doc 01; write ADR-0002 recording the layering rule | Folders exist, ADR committed |

**Learn first:** what a terminal is, what npm and `package.json` are, what a module is,
TypeScript basic types. *~1–2 weeks.*

---

## M1 — The character document

| # | Work package | Done when |
|---|---|---|
| WP1.1 | Define the `Character` type. Choices only, no derived values | Types compile |
| WP1.2 | Zod schema mirroring the type; `parseCharacter()` | Bad JSON is rejected with a readable error |
| WP1.3 | `schemaVersion` plus a migration runner | A v1 file loads into a v2 app in a test |
| WP1.4 | `exportCharacter()` / `importCharacter()` as JSON strings | Round-trip test passes |
| WP1.5 | Three fixture characters in `tests/fixtures/` | They parse |

**Learn:** TS interfaces vs types, unions, `Record`, optional properties. *~2–3 weeks.*

---

## M2 — The modifier engine — the heart of the project

| # | Work package | Done when |
|---|---|---|
| WP2.1 | `BonusType`, `Effect`, `StatKey`, `Resolution` types | Compiles |
| WP2.2 | `resolve(base, effects)` implementing the stacking rules | Returns `{total, applied, suppressed}` |
| WP2.3 | `EffectSource` interface; collect effects from race, class, feat, item, custom | Context gathers from all sources |
| WP2.4 | Exhaustive stacking tests: same-type suppression, dodge/circumstance/untyped stacking, penalties, same-source penalties | ~30 tests green |

Do not move on until WP2.4 is genuinely exhaustive. Everything downstream inherits its bugs.

**Learn:** discriminated unions, array methods (`filter`, `reduce`, grouping), test-first
development. *~2–3 weeks.*

---

## M3 — Abilities and progression

| # | Work package | Done when |
|---|---|---|
| WP3.1 | `deriveCharacter()` pipeline skeleton and context object | Empty stages run in order |
| WP3.2 | Ability scores and modifiers, including damage, drain and penalties | Fixtures assert correct modifiers |
| WP3.3 | Class progression tables in JSON; BAB and base saves, multiclass-aware | A fighter 3 / wizard 2 computes correctly |
| WP3.4 | Hit points: HD, Con, favoured class, Toughness | Matches a hand-worked example |

*~3–4 weeks.*

---

## M4 — Defences

| # | Work package | Done when |
|---|---|---|
| WP4.1 | AC, Touch AC, Flat-footed AC with size and Dex caps | All three correct for the armoured fixture |
| WP4.2 | Fortitude, Reflex, Will | Multiclass fixture correct |
| WP4.3 | CMD and flat-footed CMD | Size table applied the right way round |
| WP4.4 | Golden test: the five-buff cleric, deliberately colliding bonus types | The suppression list is right |

*~2–3 weeks.*

---

## M5 — Offence

| # | Work package | Done when |
|---|---|---|
| WP5.1 | Initiative and CMB | Correct for all fixtures |
| WP5.2 | Attack bonuses, iteratives at +6/+11/+16, two-weapon penalties | A BAB 11 fighter shows +11/+6/+1 |
| WP5.3 | Per-weapon attack lines: damage dice, Str scaling (x1.5 two-handed, x0.5 off-hand), enhancement, crit range and multiplier | Weapon table renders correct strings |
| WP5.4 | Power Attack and similar opt-in trade-offs as toggles | Toggling changes attack and damage together |

*~3–4 weeks.*

---

## M6 — Skills

| # | Work package | Done when |
|---|---|---|
| WP6.1 | `skills.json`: ability, trained-only, ACP-affected flags | Loads and validates |
| WP6.2 | Totals: ranks + ability + class-skill 3 + effects − ACP | Fixtures correct |
| WP6.3 | Max-ranks validation producing warnings, never blocks | Over-spend warns |
| WP6.4 | Size and racial effects into skills (Stealth, Fly) | Halfling fixture correct |

*~2 weeks.*

---

## M7 — Equipment and encumbrance

| # | Work package | Done when |
|---|---|---|
| WP7.1 | Item schema, equipment slots, equipped vs carried | Validates |
| WP7.2 | Armour: AC bonus, max Dex, ACP, spell failure, speed | Full plate fixture correct |
| WP7.3 | Carrying capacity, load bands, size multipliers, load penalties | Heavy-load fixture correct |
| WP7.4 | Wealth and coin weight | Sums correctly |

**Note the ordering trap:** WP7.3 must run before M4's AC, because load caps Dex. Revisit the
pipeline order here. *~3 weeks.*

---

## M8 — Content data layer

| # | Work package | Done when |
|---|---|---|
| WP8.1 | Zod schemas for race, class, feat, item | Validate |
| WP8.2 | Loader plus a frozen typed registry keyed by id | An unknown id fails loudly |
| WP8.3 | Seed content: 2 races, 2 classes, ~20 feats, core skills, ~30 items | Fixtures build from real content |
| WP8.4 | `npm run validate:content` CLI and an authoring guide in `docs/` | A malformed JSON file fails CI |

This is the milestone that makes content growth a data task rather than a code task. **Adding
a race must require zero changes under `src/core/derive/`.** If it doesn't, the design is
wrong — fix it here, not later. *~2–3 weeks.*

---

### Checkpoint: a complete, tested, headless rules engine

At this point you have no user interface and a genuinely valuable asset. Everything above is
learnable without ever touching CSS.

---

## M9 — UI foundations *(now you learn HTML and CSS)*

| # | Work package | Done when |
|---|---|---|
| WP9.1 | Semantic HTML for one sheet page, zero styling | Reads correctly as a document |
| WP9.2 | Design tokens and type scale from doc 03 | All colours are variables |
| WP9.3 | Parchment substrate, two-page spread, gutter, paper block | Looks like a book, static |
| WP9.4 | Page navigation and the flip transition | Respects `prefers-reduced-motion` |
| WP9.5 | Print stylesheet | Print preview is clean and flat |
| WP9.6 | High-contrast theme toggle and accessibility pass | AA contrast in one theme; full keyboard nav |

**Learn:** HTML semantics, the box model, flexbox, grid, custom properties, media queries.
*~4–6 weeks.*

---

## M10 — Wiring the engine to the screen

| # | Work package | Done when |
|---|---|---|
| WP10.1 | **Choose the UI framework** (React recommended) and write ADR-0004 | Decision recorded with reasoning |
| WP10.2 | A store holding one `Character`; derived state via `deriveCharacter` | Editing Dex moves AC on screen |
| WP10.3 | Reusable editable-field components bound to character paths | Fields save and restore |
| WP10.4 | **"Why is this number?" provenance popover** driven by `Resolution` | Clicking AC lists every applied and suppressed modifier |
| WP10.5 | Custom-modifier and override UI — the escape hatch made real | Any stat accepts a user modifier with a note |
| WP10.6 | Warning display | Over-spent ranks show a non-blocking marker |

WP10.4 is the moment the whole architecture pays off, and the feature no other sheet does well.

*~4–6 weeks.*

---

## M11 — The sheet pages

One work package each, in this order: **WP11.1** identity and portrait · **WP11.2** abilities
and defences · **WP11.3** offence and weapons · **WP11.4** skills · **WP11.5** feats, traits
and class features · **WP11.6** equipment and encumbrance · **WP11.7** notes and background.

*~4–6 weeks.*

---

## M12 — Effects and conditions panel

| # | Work package | Done when |
|---|---|---|
| WP12.1 | Condition content data (shaken, fatigued, sickened, prone) | Loads |
| WP12.2 | Buff and condition toggle panel writing to `activeEffectIds` | Toggling rage changes ten numbers at once |
| WP12.3 | Active-effects summary strip on every page | Always visible what is switched on |

A pure payoff milestone — the engine already does the work. *~2 weeks.*

---

## M13 — Spellcasting

| # | Work package | Done when |
|---|---|---|
| WP13.1 | Caster progression data; slots per day plus bonus slots | Cleric 7 correct |
| WP13.2 | Prepared and spontaneous models | Both render |
| WP13.3 | Save DCs and concentration | Correct per spell level |
| WP13.4 | Spellbook page; used and remaining slot tracking | Usable at the table |

*~3–4 weeks.*

---

## M14 — Persistence and output

| # | Work package | Done when |
|---|---|---|
| WP14.1 | localStorage repository behind an interface, autosave | Reload restores state |
| WP14.2 | Character list, switcher, duplicate, delete | Multiple characters |
| WP14.3 | Export and import `.json` via download and file picker | Round-trips a real character |
| WP14.4 | Print and PDF output polish | A printed sheet is playable |

*~2–3 weeks.*

---

## M15 — Release

**WP15.1** responsive and mobile layout · **WP15.2** performance pass · **WP15.3** full
accessibility audit · **WP15.4** error boundaries and crash-safe loading · **WP15.5** README,
licence notices, content guide · **WP15.6** deploy to static hosting.

*~3–4 weeks.*

---

## Cross-cutting practices

Adopt at M0 and keep throughout:

- **ADRs** in `docs/adr/` — one short file per significant decision. Future-you will ask "why
  on earth did I do that"; this answers it.
- **Conventional commits** (`feat:`, `fix:`, `docs:`) — small, single-purpose, on branches.
- **CI from M2** — GitHub Actions running `npm test` and `npm run validate:content` on every
  push. Cheap to set up, and it is what stops rot.
- **Never store a derived value.** If you catch yourself writing AC into the character file,
  stop and reread doc 01.
