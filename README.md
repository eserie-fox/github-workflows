# Shared GitHub Workflows

This repository contains small reusable GitHub Actions implementations for public Python
projects. It currently provides:

- `.github/workflows/python-ci.yml`: dependency installation, Ruff format/lint checks, and
  pytest on Python 3.11;
- `.github/workflows/python-build.yml`: standard sdist and wheel builds uploaded as the
  `python-package-distributions` artifact.

Consumers may commit an up-to-date `uv.lock` or ignore it. CI enforces `--locked` when the
lockfile is present and resolves dependencies normally when it is absent. Consumers expose Ruff
and pytest through a `dev` optional dependency, keep their Ruff and pytest settings in
`pyproject.toml`, and support Python 3.11.

A minimal caller job is:

```yaml
jobs:
  ci:
    uses: eserie-fox/github-workflows/.github/workflows/python-ci.yml@main
```

The build workflow is called the same way, using `python-build.yml`. During bootstrap,
consumers use `@main`; changing `main` therefore changes every consumer, so `main` should be
protected and reviewed.

This repository stores no project secrets, variables, environments, or publishing identity.
PyPI Trusted Publishing cannot currently trust a reusable workflow directly, so each project
keeps a thin `publish.yml` whose OIDC-enabled publish job downloads the shared build artifact
and invokes the PyPI publishing action locally.

The workflows intentionally support only these current conventions and do not establish a
broad compatibility or customization framework.
