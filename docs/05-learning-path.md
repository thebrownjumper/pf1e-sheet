# 05 — Learning Path

You are starting with no web development experience. This maps what to learn onto when you
actually need it, so you are never learning something abstractly.

## The governing idea

You will learn **TypeScript before HTML and CSS**, which is the reverse of how web development
is usually taught. That is deliberate. The first eight milestones of this project have no user
interface at all — they are arithmetic and data structures — and learning a language against a
test suite that goes red and green in half a second is a far better feedback loop than learning
it against a browser where nothing happens and you don't know why.

It also means the visual work lands when you have something worth looking at.

## Before M0 — roughly a week

Just enough to not be lost:

- **The terminal.** Changing directory, listing files, running a command. An hour, once.
- **What Node, npm and `package.json` are.** You do not need depth. You need to know that npm
  installs libraries into `node_modules/` and that `package.json` records which ones.
- **Git basics:** `add`, `commit`, `status`, `log`, `branch`, `checkout`. Ignore everything
  else for now; add `merge` and `rebase` when you first need them.

## Before M1–M2 — the big one, three to four weeks

**JavaScript fundamentals, then TypeScript on top.** Do not skip straight to TS.

JavaScript, in order of usefulness to this project:
1. Variables, `const` vs `let`, functions, arrow functions
2. Objects and arrays; destructuring; spread (`...`)
3. `map`, `filter`, `reduce`, `find`, `some`, `every` — you will use these constantly, because
   the modifier engine is essentially a filter-and-reduce over effect lists
4. Modules: `import` / `export`
5. Immutability as a habit — return new objects, never mutate inputs

Then TypeScript:
1. Basic annotations, `interface` vs `type`
2. **Union types and discriminated unions** — this is the concept that makes the `Effect` and
   `StatKey` design work, and it is the highest-value idea in the whole language for you
3. Generics, lightly. Enough to read them. Depth can wait.
4. `Record<K, V>`, `Partial`, `Pick`, `Omit`

**Suggested resources:** javascript.info for JavaScript (best free structured course there is),
then the official TypeScript Handbook. MDN for reference rather than learning.

**Also learn here:** how to write a test with Vitest. `describe`, `it`, `expect`. It is a small
API and you will use it for months.

## During M3–M8 — learning by doing

No new large topic. You are consolidating TypeScript against real problems. Things you will
pick up naturally as you need them: JSON handling, Zod schemas, module organisation, debugging
with breakpoints in VS Code, and reading a stack trace without panic.

If you want one deliberate study topic in this stretch, make it **pure functions and testing
strategy** — it is what keeps the engine trustworthy.

## Before M9 — four to six weeks

**HTML and CSS, properly.** Resist the urge to reach for Bootstrap or Tailwind; you want to
understand the underlying model, and the book aesthetic is bespoke enough that a utility
framework would fight you.

1. **HTML semantics** — headings, lists, tables, forms, labels. Which element means what, and
   why the accessibility tree cares.
2. **The box model**, then **flexbox**, then **grid**. In that order. Grid does the two-page
   spread; flexbox does everything inside it.
3. **Custom properties** (CSS variables) — the design-token system depends entirely on these.
4. **Media queries** for responsive and print.
5. Transforms and transitions, for the page turn.

**Suggested resources:** web.dev's *Learn CSS*, MDN's HTML element reference, and Josh Comeau's
CSS writing for the parts that are genuinely counter-intuitive (stacking contexts, centring,
flexbox sizing).

## Before M10 — two to three weeks

**React** (assuming WP10.1 lands there). The official React documentation is genuinely good and
is the only thing you need.

Focus on: components and props, `useState`, `useEffect` (and when *not* to use it), lists and
keys, controlled form inputs, and lifting state up. Skip context, reducers, suspense and
performance optimisation until something forces you to care.

One warning specific to this project: React tutorials will encourage you to scatter state
across components. Don't. You have exactly one piece of state — the `Character` — and
everything else is derived from it by a pure function you already wrote and tested. That is a
much simpler application than most React tutorials describe, and it stays simple only if you
refuse to drift from it.

## Beyond

Pick up as needed, not in advance: GitHub Actions for CI (M2), IndexedDB if localStorage gets
tight (M14), accessibility testing tools (M15), and static hosting (M15).

## A note on pace

Two evenings a week will get you there in about a year. Four evenings will roughly halve it.
The failure mode is not slowness — it is stopping for six weeks and losing the thread. Small,
frequent, committed work beats heroic weekends, and the ADRs exist so that a six-week gap is
survivable when it happens anyway.
