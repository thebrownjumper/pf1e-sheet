# 00 — Vision and Scope

## What this is

A web-based, editable Pathfinder 1st Edition **player** character sheet that looks and
behaves like a printed book on screen, and which computes every derived value the rules
allow it to compute — showing its working.

Three commitments, in priority order:

1. **Correct.** If the sheet says Touch AC 14, that number is defensible against the
   rulebook, and you can click it to see exactly which modifiers produced it.
2. **Maintainable.** Game content lives in data files, not code. Rules logic is separate
   from presentation. Everything in the rules layer is unit tested.
3. **Beautiful.** Parchment, letterpress typography, a two-page spread, page turns.

In that order. A gorgeous sheet that quietly gets flat-footed AC wrong is worse than no
sheet at all, because a player will trust it at the table.

## Explicitly in scope

- Player-facing character state only
- Multiclass characters, all ability-score sources, size changes
- Full defensive stat block: AC / touch / flat-footed, saves, CMD / flat-footed CMD
- Full offensive stat block: initiative, BAB with iteratives, CMB, per-weapon attack lines
- Skills, including class-skill bonus, armour check penalty, max-ranks validation
- Equipment, encumbrance, carrying capacity, load effects on Dex cap / ACP / speed
- Spellcasting: slots per day, bonus slots, save DCs, concentration, prepared vs spontaneous
- Toggleable temporary effects (rage, bless, haste, conditions) that flow through the maths
- Local persistence, JSON import/export, print/PDF output

## Explicitly out of scope

- Anything Game Master facing: encounters, initiative tracking for a party, NPCs, loot
  generation, maps, monster stat blocks
- Dice rolling (deliberate — the sheet reports numbers, the table rolls them). Revisit only
  after v1 ships.
- Multiplayer, real-time sync, accounts, a server of any kind
- A homebrew content *editor* with a UI. Content is authored as JSON files by hand, against
  a documented schema. A UI for this is a v2 conversation.
- Automatic levelling / build advice / legality enforcement. The sheet assists; it does not
  police. Warnings, never hard blocks.

## The escape hatch (non-negotiable)

No engine will ever cover all of Pathfinder 1e. Splatbooks, archetypes, GM rulings and
sheer volume guarantee it. Therefore **every derived value must accept a user-supplied
custom modifier with a free-text note, and an optional hard override.**

This is designed in from the first line of the engine, not bolted on later. It is the
difference between a tool people use and a tool people abandon the first time it can't
express their character.

## Decisions taken

| Decision | Choice | Rationale |
|---|---|---|
| Language | TypeScript | The domain is ~200 interlocking values; a type checker is the cheapest bug-prevention available |
| Build tool | Vite | Fastest path from nothing to a running project; minimal config |
| Tests | Vitest | Same config as Vite, watch mode, near-zero setup |
| UI framework | **Deferred to Milestone 10** | Learn the language and CSS first; the engine must not depend on this choice anyway |
| Rules content | Engine-first, seed data only, content grown over time | Decouples the hard part (code) from the endless part (data entry) |
| Persistence | Browser-local + JSON import/export | No backend, no accounts, no privacy surface, deploys as a static site |
| Formula evaluation | Numbers and a fixed set of named scaling functions only | A string-formula interpreter is a security and complexity trap; refuse it in v1 |

## Honest sizing

Working evenings and weekends from a standing start with no web experience:

| Stage | Elapsed |
|---|---|
| Toolchain and language basics (M0–M1) | 4–6 weeks |
| A tested, correct, headless rules engine (M2–M8) | 4–6 months |
| An ugly but fully working sheet on screen (M9–M11) | +3–4 months |
| The book aesthetic, spells, polish, deploy (M12–M15) | +3–4 months |

Call it **9–14 months** to something you'd hand to a friend. The first genuinely useful
artefact — a sheet you could actually play from — lands around month 6–7.

If that is too long, the single best scope cut is **one class, one race, no spellcasting**
for v1. That reaches a playable sheet in roughly three months and every later class is then
data entry rather than engineering.

## Licensing note (flagging, not advising)

Pathfinder 1e mechanics are Open Game Content under the OGL 1.0a; if you distribute rules
text you need the licence text and a Section 15 declaration. "Pathfinder", Golarion, the
iconic characters and much of the art and flavour are Product Identity and trademarks, and
are **not** covered. Practical implications for a public release:

- Don't name the product "Pathfinder Character Sheet"; give it its own name and describe it
  as compatible
- Use your own test characters, not the iconics, in fixtures and screenshots
- Read Paizo's Community Use Policy before publishing

None of this affects a project kept private, which is where you'll be for most of a year.
I'm not a lawyer — this is a flag to check, not a legal opinion.
