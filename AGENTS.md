# Dyscolab workspace (u2602)

## What this is

This is a development workspace for the **Dyscolab** scientific Python
ecosystem. Four libraries are checked out as git submodules, and one root
pixi environment installs **all of them from source**, so changes in one
library are immediately visible to the others.

| Submodule | Role | Depends on |
| --- | --- | --- |
| [symbolite/](symbolite/) | Minimal symbolic expressions, translated to numerical, code and LaTeX backends | — |
| [poincare/](poincare/) | Declarative ODE systems (`System`, `Variable`, `Parameter`, equations), compilation, simulation, units, reactions, analysis | symbolite |
| [simbio/](simbio/) | Chemical reaction networks: compartments and volumes, reaction motifs, SBML/MathML I/O | poincare |
| [jablonski/](jablonski/) | Photophysical state/transition models, spectra, quantum yields, excitation | poincare |

Dependency direction: `symbolite → poincare → {simbio, jablonski}`. simbio and
jablonski do not depend on each other.

## Before changing a submodule

**Read that submodule's `AGENTS.md` before you modify anything in it**
([symbolite](symbolite/AGENTS.md), [poincare](poincare/AGENTS.md),
[simbio](simbio/AGENTS.md), [jablonski](jablonski/AGENTS.md)). Each one
covers the architecture, invariants, commands and downstream consumers of
its repository. If a change spans several submodules, read all the
affected ones.

The submodule files are written for **standalone** checkouts and say
nothing about this workspace. Inside this workspace, apply them like this:

| Submodule AGENTS.md says | In this workspace, do |
| --- | --- |
| `pixi run test` (in the submodule) | `pixi run test-<name>` from the workspace root (uses development versions of all four) |
| Run downstream suites against a source install of this checkout | `pixi run test-<downstream>` for each downstream library, from the workspace root |
| Test with source installs of upstream libraries | Already the case. The root environment installs every library from source |
| Search downstream sources (`rg ... <repo>/src`) | `rg ... poincare/src simbio/src jablonski/src` |
| Lint commands (`pixi run -e lint ...`, `pixi run ruff ...`) | Run them from inside the submodule directory with the submodule's environment, because the root environment has no lint tools |

## Python environment

- Run everything from the **top-level** pixi environment (`.pixi/` at the
  workspace root). Its manifest is [pixi.toml](pixi.toml), and it installs
  `symbolite`, `poincare`, `simbio` and `jablonski` from the sibling
  directories.
- When you inspect, reference, import or test `poincare`, `symbolite`,
  `simbio` or `jablonski`, always use the **development sources** in the
  sibling submodule directories (`<name>/src/<name>/`). Never use the
  published copies installed in a submodule's own `.pixi/` environment.
  Submodule environments pull *published* versions of their sibling
  dependencies, so they can disagree with the workspace.
- Tests: `pixi run test-symbolite`, `pixi run test-poincare`, `pixi run test-simbio`,
  `pixi run test-jablonski`. Each runs `pytest --doctest-modules` inside the
  submodule against the development versions. Pass extra pytest arguments
  after the task name, for example `pixi run test-poincare src/poincare/tests/test_units.py -x`.
- The root environment is not pinned to a Python version and currently
  runs 3.14. Submodule CI matrices cover 3.12–3.14 (poincare only
  3.12–3.13). Avoid syntax or APIs newer than 3.12.
- The root environment has **no lint tools** (ruff, pre-commit and prek are
  missing). Lint from inside the submodule with its own environment, as
  described in each submodule's AGENTS.md. Run ruff only on the files you
  changed, because every submodule has pre-existing lint and format debt.

## Cross-repository changes

- Make the change in the lowest affected library first (symbolite, then
  poincare, then simbio or jablonski).
- After **any** change to an upstream library, run the tests of that library
  **and of every library that depends on it**. This includes internal
  refactors and bug fixes, not only public API changes, because downstream
  code can break in unexpected ways:
  - symbolite → `pixi run test-symbolite`, `test-poincare`, `test-simbio`, `test-jablonski`
  - poincare → `pixi run test-poincare`, `test-simbio`, `test-jablonski`
  - simbio or jablonski → their own task only
- Downstream packages import many "private" upstream modules
  (`symbolite.core.*`, `symbolite.impl._lang_value_utils`, `poincare._node`,
  `poincare.reactions.reactions`). Grep the downstream `src/` directories
  before renaming or changing them.
- Bump a lower bound in a downstream `pyproject.toml` (for example
  `poincare >= 1.2.0`) only when asked. Report it when the new code needs a
  newer upstream release.

## Source control

- Each submodule is its own git repository with its own remote
  (`github.com/dyscolab/<name>`). Commit inside the submodule. The root
  repository only records submodule pointers (root commits are titled
  "submodule changes"), plus the root `pixi.toml`/`pixi.lock`.
- Use conventional commits (`feat:`, `fix(core):`, `docs(README):`, …) in
  all repositories.
- Do not add `Co-Authored-By:` trailers or any other AI or tool attribution
  lines to commit messages, in any repository.
- `temp/` is gitignored in the root repository.
  `temp/` holds personal scratch notebooks and notes, so do not treat it as
  project documentation or depend on it.
