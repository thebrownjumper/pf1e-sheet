# 02 — The Rules Engine

This is the heart of the project. Read this before writing any code in `src/core/`.

## The central idea: everything is an Effect

A race, a class feature, a feat, a magic ring, a spell, a condition and a manual note the
player typed are all the same kind of thing to the engine: a **source that emits effects**.

```ts
type BonusType =
  | 'enhancement' | 'deflection' | 'natural' | 'armor' | 'shield'
  | 'dodge' | 'luck' | 'morale' | 'insight' | 'sacred' | 'profane'
  | 'competence' | 'resistance' | 'alchemical' | 'size' | 'racial'
  | 'circumstance' | 'trait' | 'untyped';

type Effect = {
  id: string;
  source: string;        // "Ring of Protection +1" — shown to the user, so write it nicely
  sourceKind: 'race' | 'class' | 'feat' | 'item' | 'spell' | 'condition' | 'custom';
  target: StatKey;       // 'ac' | 'save.fort' | 'skill.stealth' | 'ability.str' | …
  type: BonusType;
  value: number;
  active: boolean;       // buffs and conditions toggle this
  appliesWhen?: string;  // "vs. evil outsiders" — INFORMATIONAL ONLY, never auto-applied
  note?: string;
};
```

`appliesWhen` deserves emphasis. Conditional bonuses ("+2 vs. fear", "+1 against goblinoids")
are pervasive in Pathfinder and modelling them properly means modelling the entire combat
context, which you are not going to do. So: carry the text, display it next to the number,
and let the player apply their own judgement. This is the correct trade, not a cop-out.

## StatKey

A string union, not free-form strings. This buys autocomplete and compile-time protection
against typos that would otherwise silently drop a bonus.

```ts
type StatKey =
  | `ability.${AbilityKey}`
  | 'hp' | 'initiative' | 'speed'
  | 'ac' | 'ac.touch' | 'ac.flatFooted' | 'ac.naturalArmor'
  | `save.${'fort'|'ref'|'will'}`
  | 'cmb' | 'cmd'
  | 'attack.melee' | 'attack.ranged' | 'damage.melee'
  | `skill.${SkillId}`
  | 'spell.dc' | 'concentration'
  | 'acp' | 'maxDex' | 'spellFailure';
```

## Stacking resolution — get this exactly right

Pathfinder's stacking rules, as the engine must implement them:

- **Bonuses of the same type do not stack.** Apply the largest; suppress the rest.
- **These types DO stack with themselves:** `dodge`, `circumstance`, `untyped`.
- **Penalties (negative values) stack**, regardless of type — except two penalties from the
  same source, which do not.
- `size` is computed once as a single value, not accumulated from multiple sources.

```ts
type Resolution = {
  total: number;
  applied:   Effect[];
  suppressed: { effect: Effect; reason: string }[];   // "overridden by Cloak of Resistance +3"
};

function resolve(base: number, effects: Effect[]): Resolution
```

**Return `Resolution`, never a bare number.** This one decision is what makes "why is my AC
18?" a two-line UI feature instead of a retrofit that touches every file. Design it in on day
one; it is nearly free then and brutal later.

## The formulae

Verify each of these against the rulebook as you implement it, and encode each as a test.

### Abilities
```
modifier = floor((score - 10) / 2)
effective score = base + racial + levelIncreases + inherent + enhancement
                  + temporary − damage − penalties;  drain reduces base directly
```

### Progression (per class, summed across classes for multiclass)
```
BAB:  full = level          (fighter, barbarian, paladin, ranger…)
      3/4  = floor(3×level/4)  (cleric, druid, monk, rogue, bard…)
      1/2  = floor(level/2)    (wizard, sorcerer, witch)

base save: good = 2 + floor(level/2)      poor = floor(level/3)
```
Multiclass: compute each class's contribution separately and **then** sum. Do not sum the
levels first. (This is exactly why multiclassing inflates saves — two good Fort saves each
grant the +2 base.)

Iterative attacks at BAB +6 / +11 / +16, each at −5 from the previous.

### Size modifiers — two different tables, don't conflate them
```
                    Fine  Dim  Tiny  Sml  Med  Lrg  Huge  Garg  Col
AC and attack rolls  +8   +4    +2   +1    0   −1    −2    −4   −8
CMB and CMD          −8   −4    −2   −1    0   +1    +2    +4   +8
```

### Armour Class
```
AC          = 10 + armor + shield + min(Dex, maxDexCap) + size + natural
                 + deflection + dodge + misc
Touch AC    = 10 + min(Dex, maxDexCap) + size + deflection + dodge + misc
              (excludes armor, shield, natural armor)
Flat-footed = AC minus Dex (if positive) minus dodge
```
A Dex *penalty* still applies to touch and flat-footed AC. `maxDexCap` is the lower of the
armour's limit and the encumbrance limit.

### CMB / CMD
```
CMB = BAB + Str mod + size(CMB table)
CMD = 10 + BAB + Str mod + Dex mod + size(CMB table) + deflection + dodge + misc
flat-footed CMD = CMD − Dex − dodge
```

### Skills
```
total = ranks + ability mod + (ranks ≥ 1 && class skill ? 3 : 0) + effects − ACP(if Str/Dex based)
max ranks per skill = total character level (Hit Dice)
```

### Encumbrance
```
capacity from Str table;  ×2 Large, ×4 Huge …  ×3/4 Small, ×1/2 Tiny
light  ≤ ⅓ max      no penalty
medium ≤ ⅔ max      maxDex +3, ACP −3, speed 30→20 / 20→15
heavy  ≤ max        maxDex +1, ACP −6, speed 30→20 / 20→15
```

### Spellcasting
```
save DC       = 10 + spell level + casting ability modifier
concentration = caster level + casting ability modifier
bonus slots   from the ability-score table; requires score ≥ 10 + spell level to cast at all
```

## Order of operations that people get wrong

1. Encumbrance must be computed **before** AC, because it can cap Dex.
2. Temporary ability changes (rage, bull's strength) must land **before** anything reading
   that modifier — a raging barbarian's Fort save, CMB, CMD and damage all shift together.
3. Ability *damage* and *drain* differ: damage is temporary and reduces the effective score;
   drain reduces the score itself. Model both.
4. Size changes cascade into attack, AC, CMB, CMD, skills (Stealth, Fly) and carrying
   capacity. A single `size` value must fan out to all of them.

## Scope guards for the engine

- **No string formula interpreter.** Some effects scale ("+1 per four caster levels"). Model
  those with a small fixed set of named scaling functions referenced by name from JSON. Never
  `eval`, never a mini-language. This holds the line against the data format quietly becoming
  an untested programming language.
- **Warnings, never blocks.** Over-spent skill ranks, unmet feat prerequisites and illegal
  ability arrays produce `Warning` objects the UI displays. The engine never refuses to
  compute. Players have GM permission for things your engine has never heard of.
