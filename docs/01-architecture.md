# 01 — Architecture

## The one rule

```
        ui/  ───depends on───▶  core/  ◀───depends on───  data/
                                  │
                            depends on NOTHING
```

`core/` contains no DOM, no browser APIs, no framework imports, no `window`, no CSS, no
`fetch`. It is a pure function library. Given a character's *choices* and a body of game
content, it returns every derived number.

`ui/` may import from `core/`. `core/` may never import from `ui/`. If you enforce nothing
else in this project, enforce that arrow.

Why it matters here specifically: the rules engine is the part with real intellectual value
and the part that must be provably correct. Keeping it framework-free means it can be tested
in milliseconds without a browser, reused in a CLI or a Discord bot, and survive you changing
your mind about React in eighteen months.

## Project layout

```
pf1e-sheet/
├── docs/                      # this planning set; ADRs live in docs/adr/
├── content/                   # game data as JSON — the growable part
│   ├── races/*.json
│   ├── classes/*.json
│   ├── feats/*.json
│   ├── items/*.json
│   └── skills.json
├── src/
│   ├── core/                  # PURE TYPESCRIPT. No browser. No framework.
│   │   ├── types/             # Character, Effect, StatKey, BonusType…
│   │   ├── schema/            # Zod validators + version migrations
│   │   ├── engine/            # the modifier resolver
│   │   ├── derive/            # the derivation pipeline, one file per stage
│   │   └── content/           # content loading, validation, typed registry
│   ├── ui/                    # everything visual (from Milestone 9)
│   │   ├── styles/            # design tokens, book chrome, print
│   │   ├── pages/             # one module per sheet page
│   │   └── components/
│   └── storage/               # localStorage, import/export
└── tests/
    ├── unit/
    └── fixtures/              # hand-verified characters — the safety net
```

## The two data shapes

Everything hinges on keeping these separate and never confusing them.

### 1. `Character` — the saved document

Contains **only choices a player made**. Race, class levels, ability score purchases,
skill ranks allocated, feats selected, items owned, items equipped, notes.

It contains **no derived values**. Not AC, not attack bonus, not skill totals. Ever.

The moment you store a derived value in the saved document, you have two sources of truth
and they will drift. This is the single most common way homebrew character sheets rot.

```ts
type Character = {
  schemaVersion: number;          // for migrations — see §Migrations
  id: string;
  identity: { name: string; player?: string; alignment?: Alignment; deity?: string; /* … */ };
  raceId: ContentId;
  classes: { classId: ContentId; level: number; archetypeIds: ContentId[]; favoredClass: boolean }[];
  abilities: Record<AbilityKey, {
    base: number;                 // point buy / rolled
    levelIncreases: number;       // the 4th/8th/12th… bumps
    inherent: number;             // tomes, wishes
  }>;
  skillRanks: Record<SkillId, number>;
  featIds: ContentId[];
  traitIds: ContentId[];
  inventory: InventoryEntry[];
  customEffects: Effect[];        // THE ESCAPE HATCH
  overrides: Partial<Record<StatKey, { value: number; note: string }>>;
  activeEffectIds: string[];      // which buffs/conditions are switched on right now
  notes: Record<string, string>;
};
```

### 2. `DerivedSheet` — computed, never saved

```ts
type DerivedSheet = {
  abilities: Record<AbilityKey, { score: Resolution; modifier: number }>;
  ac: { normal: Resolution; touch: Resolution; flatFooted: Resolution };
  saves: Record<'fort'|'ref'|'will', Resolution>;
  // …and so on for every number on the sheet
  warnings: Warning[];            // "skill ranks exceed maximum", "feat prereq unmet"
};
```

Recomputed from scratch on every change. It is fast enough — this is arithmetic over a few
hundred values, not a physics simulation. Do not cache it prematurely.

## Derivation is an ordered pipeline, not a reactive graph

The tempting design is a dependency graph that recomputes what changed. Resist it. Pathfinder
has genuine ordering dependencies (encumbrance depends on Strength; the Dex cap on AC depends
on encumbrance *and* on armour), and an explicit pipeline makes that order visible and
debuggable, where a graph makes it implicit and mysterious.

```ts
function deriveCharacter(char: Character, content: ContentRegistry): DerivedSheet {
  const ctx = createContext(char, content);   // collects every Effect from every source
  deriveAbilities(ctx);      // 1. scores, modifiers, damage/drain/penalties
  deriveProgression(ctx);    // 2. HD, BAB, base saves from class tables, feat slots
  deriveEquipment(ctx);      // 3. loads, carrying capacity, armour caps, ACP, speed
  deriveDefenses(ctx);       // 4. AC family, saves, CMD family, HP
  deriveOffense(ctx);        // 5. initiative, attack lines, CMB, damage
  deriveSkills(ctx);         // 6. per-skill totals
  deriveSpellcasting(ctx);   // 7. slots, DCs, concentration
  return finalise(ctx);
}
```

Each `derive*` is a pure-ish function over the context, in its own file, with its own test
file. Each stage may only read values produced by earlier stages. That constraint is the
whole design.

## Migrations

Character files will outlive your schema. Bake this in at Milestone 1, not later.

```ts
const migrations: Record<number, (doc: any) => any> = {
  1: d => ({ ...d, traitIds: d.traitIds ?? [] }),
  2: d => ({ ...d, overrides: {} }),
};
```

Load: validate → if `schemaVersion < CURRENT`, run each migration in order → validate again.
Never mutate a character file in place on disk without the user asking; migrate on load.

## Content registry

Content JSON is loaded once, validated with Zod, and frozen into a typed registry keyed by
id. Code never hardcodes a race or class. Adding the Alchemist should be a new JSON file and
a test, with zero changes to `src/core/derive/`.

Where that proves impossible — and it will, for genuinely novel mechanics like the Magus's
arcane pool — the class JSON gains a named `hook` string, and the engine holds a small,
documented registry of hook implementations. Keep that list short and make adding to it feel
expensive; it's the pressure valve that stops the data format becoming a programming language.

## Testing strategy

The engine's correctness is the product. Three layers:

1. **Unit tests** per derivation stage, testing rules in isolation
2. **Stacking property tests** — the bonus resolver gets exhaustive coverage; it is the
   component most likely to be subtly wrong and most expensive to get wrong
3. **Golden fixtures** — 6–10 complete characters, each with every derived value worked out
   by hand from the rulebook and asserted. Suggested spread:
   - L1 human fighter, full plate, no magic — the baseline
   - L1 halfling rogue — size modifiers everywhere
   - L5 fighter/wizard multiclass — BAB and save stacking across classes
   - L8 cleric with five buffs active — bonus type collisions on purpose
   - L12 barbarian raging — temporary ability scores changing downstream values
   - A character carrying a heavy load in medium armour — two Dex caps competing

Those fixtures are the regression net. Write them early; they will catch more real bugs than
any other single practice in this project.
