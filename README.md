# Shared GitHub Workflows

This repository provides small reusable GitHub Actions workflows for public Python projects:

- `.github/workflows/python-ci.yml` runs Ruff formatting and lint checks plus pytest;
- `.github/workflows/python-build.yml` builds an sdist and wheel and uploads the
  `python-package-distributions` artifact.

Every CI caller explicitly supplies `test-targets`, a nonempty JSON array of exact runner and
Python-version pairs. JSON is used because reusable-workflow inputs have no native array or
object type. All declared targets are treated equally and remain visible as individual matrix
jobs for diagnostics. `typecheck: true` also runs `uv run mypy src tests` in `format-lint`.
Callers retain the stable required gates `ci / format-lint` and `ci / tests`.

A cross-platform typed project calls CI like this:

```yaml
jobs:
  ci:
    name: ci
    uses: eserie-fox/github-workflows/.github/workflows/python-ci.yml@main
    with:
      test-targets: >-
        [
          {"runner":"ubuntu-latest","python":"3.11"},
          {"runner":"ubuntu-latest","python":"3.13"},
          {"runner":"windows-latest","python":"3.11"},
          {"runner":"macos-latest","python":"3.11"}
        ]
      typecheck: true
```

A Linux-only project calls CI like this:

```yaml
jobs:
  ci:
    name: ci
    uses: eserie-fox/github-workflows/.github/workflows/python-ci.yml@main
    with:
      test-targets: >-
        [
          {"runner":"ubuntu-latest","python":"3.11"},
          {"runner":"ubuntu-latest","python":"3.13"}
        ]
```

A committed `uv.lock` is honored with `uv sync --locked`; a lockfile is not required. Consumers
provide the `dev` extra and keep tool configuration in `pyproject.toml`.

The shared build is consumed from a thin project-local publication workflow. Pull requests and
ordinary `main` pushes run CI only. A manual `workflow_dispatch` builds once and publishes to
TestPyPI; a newly created `v*` tag builds once and publishes to PyPI. Deleted tags do not build or
publish.

Consumers intentionally reference `@main` so shared policy evolves continuously. A change to
shared `main` reaches every consumer and therefore requires careful review. This repository
stores no project secrets, variables, environments, or publishing identity.
