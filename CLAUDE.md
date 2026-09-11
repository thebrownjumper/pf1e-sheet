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
- **Nothing is committed straight to `main`.** Every change goes on a branch and reaches
  `main` through a pull request. See *Division of labour* below.
- New significant decisions get an ADR in `docs/adr/`, numbered sequentially.
- Tests accompany rules code. The golden fixtures in `tests/fixtures/` are the regression net
  for correctness — treat a fixture change as a claim that needs justifying against the
  rulebook.

## Division of labour

Two machines, each running Claude Code against this same repository. They share no state
except what is pushed to `origin`, so the remote is the only channel between them — work that
has not been pushed does not exist to the other side.

**The invariant: whoever did not write the change reviews it.** Roles follow the change, not
the machine. Either machine may author; the other one reviews. This is the whole point of the
split — a reviewer who wrote the code re-reads their own reasoning and finds nothing.

- The authoring machine creates the branch, makes the commits, pushes, and opens the pull
  request.
- The reviewing machine pulls the branch, runs the tests, exercises the behaviour, and reviews
  on the pull request. It says what is wrong; it does not silently rewrite the change.
- Fixes arising from a review go back to the authoring machine, on the same branch, as new
  commits.
- The pull request is merged only once review has passed. Branches are deleted after merge.

Two things follow from the machines being different, and they are tendencies rather than
rules. Anything whose correctness depends on the platform — npm scripts, path handling, line
endings — is only really reviewed once the Windows machine has run it, so a cross-platform
change wants the Fedora side to author. And because review is the scarcer, more valuable pass,
the machine John is sitting at is usually the one that should author.

Both machines push as the same GitHub account, so GitHub will not accept an *approve* or
*request changes* review from either — it refuses those on your own pull request. Reviews land
as comments, and the verdict is stated in the text. The rule is enforced by convention, not by
the platform.

This is a working discipline, not a security boundary — John owns both machines and can
override it whenever he says so explicitly.

### What the authoring machine owes the reviewer

Written from the reviewing side. Each of these exists because without it a review either
cannot run or cannot reach a verdict.

- **Keep every npm script cross-platform.** The reviewer runs Windows, where npm executes
  scripts through `cmd.exe`. Verified on that machine: `&&` chaining works and is safe to use;
  Unix filesystem commands (`rm`, `cp`, `mv`, `touch`), command substitution (`$(...)`) and
  single-quoted arguments do not. The one that bites most often is an inline environment
  variable — `"test:ci": "NODE_ENV=test vitest run"` runs on Fedora, and on Windows exits 1
  with `'NODE_ENV' is not recognized as an internal or external command`. Use `cross-env` for
  those, Node for anything a script must do to the filesystem, and bare tool invocations
  otherwise.

  A trap when checking this yourself: Git for Windows ships its own `rm.exe` and `cp.exe` and
  puts them on `PATH`, so a script using them passes when npm is launched from Git Bash and
  still fails from PowerShell or `cmd`. Verify from PowerShell, not Git Bash.
- **One work package per pull request**, with the number in the title (`WP2.2: stacking
  resolver`). The description states which *done when* criterion from
  [`docs/04-roadmap.md`](docs/04-roadmap.md) it satisfies, so review is checked against an
  agreed target rather than taste.
- **Tests ship in the same pull request as the code they cover.** A PR that adds rules logic
  without tests cannot be reviewed for correctness, only for style.
- **Never force-push a branch that is under review.** Corrections go on as new commits;
  history is tidied at merge if at all. A rewritten branch discards the review already done
  and forces a full re-read.
- **Justify any change to a golden fixture in the PR description** — which rule changed, and
  where in the rulebook it says so. A fixture whose expected numbers move is either a fix or a
  regression, and the diff alone cannot tell the reviewer which.

## Environment

Developed across a Windows machine and a Fedora laptop. Keep everything path-agnostic: no
hardcoded absolute paths, no OS-specific shell in npm scripts, LF line endings enforced by
`.gitattributes`.
