## Development

After cloning the repository,
run:
```
git submodule update --init --recursive
```
inside the repository.

- Pixi tasks run from the repository root
will use the `pixi.toml` provided there,
which defines an environment with development versions of all submodules.
- Pixi tasks run in each of the submodules
will use the submodule's manifest (`pyproject.toml`),
which defines an environment with a published (non-development) version the other submodules.

### Example from submodule:

```
cd poincare
pixi run test
```

will run tests with the development version of `poincare`,
but with a published version of `symbolite`.

### Example from root:

```
pixi run test-poincare
```

will run tests with development versions of both `poincare` and `symbolite`.
