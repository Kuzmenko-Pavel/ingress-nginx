# Contributing

This is a personal, self-maintained distribution of ingress-nginx (see [`README.md`](./README.md)).
There is no CLA, no prow bot, and no `/lgtm` workflow — those belonged to the upstream
Kubernetes project and do not apply here.

Contributions are welcome via pull request, but the maintainer may accept, modify, or decline
any change at their discretion.

## Workflow

- `main` is the source of truth. Do all work on a feature branch and open a PR **into `main`
  within this repository**. Do not open PRs against `kubernetes/ingress-nginx`.
- Keep changes focused and describe what and why in the PR.
- Before pushing, run the relevant checks locally:

  ```bash
  make test          # Go unit tests
  make lua-test      # Lua unit tests (if you touched rootfs/etc/nginx/lua)
  make helm-test     # chart unit tests (if you touched charts/)
  make verify-docs   # if you changed annotations/docs generation
  ```

  CI (`.github/workflows/ci.yaml`) runs lint, unit, build, chart and kind e2e on PRs.

- Do not hand-edit generated files: the chart `README.md` is generated from
  `README.md.gotmpl` (helm-docs); annotation docs via `make verify-docs`.
- See [`AGENTS.md`](./AGENTS.md) for the codebase map, build/release model, and hard rules
  (e.g. do not reintroduce upstream-only CI bindings, do not commit registry/digest edits to
  `charts/ingress-nginx/values.yaml`).

## Releasing

Releases are cut by pushing a plain SemVer tag (`vX.Y.Z`); the `release` workflow builds and
publishes images and the OCI Helm chart to GHCR. See
[`docs/maintained-distribution-release.md`](docs/maintained-distribution-release.md).

## Conduct

Be respectful and constructive. See [`code-of-conduct.md`](./code-of-conduct.md).
