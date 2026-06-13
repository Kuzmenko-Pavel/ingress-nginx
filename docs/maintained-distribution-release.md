# Maintained ingress-nginx Distribution

This repository is a self-maintained distribution based on the ingress-nginx codebase.
It is developed and released independently, with its own runtime artifacts published to
GitHub Container Registry (GHCR). It is not a contribution fork: pull requests are not
sent to `kubernetes/ingress-nginx`, and the project is free to evolve its Go code, Lua
code, nginx templates, Helm chart, Dockerfiles, GitHub Actions, and documentation.

## Ownership model

- This is an independently maintained codebase, not a release-only mirror.
- `main` in this repository is the single source of truth.
- Feature work happens on branches and is merged into `main` via pull requests **within
  this repository**. Pull requests are never opened against `kubernetes/ingress-nginx`.
- The maintainer owns all runtime code, chart, build logic, and release process.

## Release model

A release is produced by pushing a Git tag, then GitHub Actions builds and publishes the
artifacts. Tags use plain SemVer:

```bash
git tag v1.15.1
git push origin v1.15.1
```

The workflow `.github/workflows/release.yaml` will:

1. Determine versions from the Git tag (release tag) and repository files (`certgen`, `chart`)
2. Build and push the controller and controller-chroot images using the project's native
   build logic: `make release REGISTRY=ghcr.io/kuzmenko-pavel/ingress-nginx` (multi-arch
   `linux/amd64,linux/arm,linux/arm64` via Docker Buildx)
3. Build and push `kube-webhook-certgen` natively: `cd images && make push NAME=kube-webhook-certgen REGISTRY=...`
4. Resolve image digests and patch `charts/ingress-nginx/values.yaml` with GHCR registry,
   tags (and digests when available) for packaging
5. Run `helm lint` and `helm template` validation
6. Package and push the Helm chart as an OCI artifact (`oci://ghcr.io/kuzmenko-pavel/charts`)
7. Create a GitHub Release with a provenance block, image digest table, and chart attachment

Auxiliary images (`images/**`) and the `nginx` base image are built and published natively
by `.github/workflows/images.yaml` on changes under `images/`, pushed to the same GHCR
namespace via `make`. There is no custom build-push wrapper — all image builds go through
the repository `Makefile` / `images/Makefile`.

### Manual trigger

The workflow can also be run from the GitHub Actions UI (`workflow_dispatch`) with an
explicit `tag` input and an optional `upstream_base` input (a base upstream tag/commit to
record in the release provenance).

## Versioning policy

- Releases use plain SemVer tags, e.g. `v1.15.1`, `v1.15.2`, `v1.16.0` — no vendor suffixes.
- The Helm chart version uses plain SemVer, e.g. `4.15.1`, set in
  `charts/ingress-nginx/Chart.yaml` `.version`.
- Image tags always match the release tag.
- When the base upstream commit/tag is known, record it in the release notes (via the
  `upstream_base` input or an `UPSTREAM_BASE` file at the repository root).

## Provenance

Provenance is captured in two places:

- OCI image labels: `org.opencontainers.image.source`, `.revision`, `.version`.
- GitHub Release notes: repository, commit SHA, and the optional upstream base reference.

## Optional upstream reference

The original `kubernetes/ingress-nginx` remote is kept only as a read-only historical
reference, renamed locally to `upstream-archive`. It is **not** a mandatory base and is not
rebased onto as part of any standard process. It may be used manually to cherry-pick a
specific fix if that is ever desired:

```bash
git fetch upstream-archive --tags
git cherry-pick <commit-sha>
```

## GHCR artifacts

| Artifact | Location |
|----------|----------|
| Controller image | `ghcr.io/kuzmenko-pavel/ingress-nginx/controller` |
| Controller-chroot image | `ghcr.io/kuzmenko-pavel/ingress-nginx/controller-chroot` |
| kube-webhook-certgen | `ghcr.io/kuzmenko-pavel/ingress-nginx/kube-webhook-certgen` |
| Helm chart (OCI) | `ghcr.io/kuzmenko-pavel/charts/ingress-nginx` |

Install via the OCI Helm chart:

```bash
helm install ingress-nginx \
  oci://ghcr.io/kuzmenko-pavel/charts/ingress-nginx \
  --version 4.15.1 \
  --namespace ingress-nginx \
  --create-namespace
```

Verify an image digest:

```bash
docker buildx imagetools inspect \
  ghcr.io/kuzmenko-pavel/ingress-nginx/controller:v1.15.1
```

## Local development workflow

```bash
git checkout -b feature/my-change
# edit runtime code, chart, build logic, docs as needed
# build/test locally, then open a PR into main within this repository
git push origin feature/my-change
```

- `main` is the source of truth; changes land via PR into `main`.
- Committed chart values (`charts/ingress-nginx/values.yaml`) may be changed in the future
  when the maintained distribution requires it; the release workflow may additionally patch
  them at runtime for packaging convenience.
