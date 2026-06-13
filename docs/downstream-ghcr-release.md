# Downstream GHCR Release

This fork publishes its own runtime artifacts to GitHub Container Registry (GHCR)
without modifying any ingress-nginx runtime code, chart templates, or controller behavior.

## Published artifacts

| Artifact | Location |
|----------|----------|
| Controller image | `ghcr.io/kuzmenko-pavel/ingress-nginx/controller` |
| Controller-chroot image | `ghcr.io/kuzmenko-pavel/ingress-nginx/controller-chroot` |
| kube-webhook-certgen | `ghcr.io/kuzmenko-pavel/ingress-nginx/kube-webhook-certgen` |
| Helm chart (OCI) | `ghcr.io/kuzmenko-pavel/charts/ingress-nginx` |

## Release process

A release is triggered by pushing a Git tag matching `vX.Y.Z`:

```bash
git tag v1.15.1
git push origin v1.15.1
```

The GitHub Actions workflow `.github/workflows/release-ghcr.yml` will:

1. Determine versions from the Git tag (`controller`) and repository files (`certgen`, `chart`)
2. Build multi-arch images (`linux/amd64`, `linux/arm64`) via Docker Buildx
3. Push images to GHCR and capture digests
4. Patch `charts/ingress-nginx/values.yaml` with GHCR registry, tags, and digests (in-workflow only, not committed)
5. Run `helm lint` and `helm template` validation
6. Package and push Helm chart as OCI artifact
7. Create a GitHub Release with image digest table and chart attachment

### Manual trigger

The workflow can also be triggered manually via GitHub Actions UI (`workflow_dispatch`)
with an explicit tag input, without creating a Git tag.

## Installation via OCI Helm chart

```bash
helm install ingress-nginx \
  oci://ghcr.io/kuzmenko-pavel/charts/ingress-nginx \
  --version 4.15.1 \
  --namespace ingress-nginx \
  --create-namespace
```

## Verify image digest

```bash
docker buildx imagetools inspect \
  ghcr.io/kuzmenko-pavel/ingress-nginx/controller:v1.15.1
```

## Version mapping

| Component | Version source | Value |
|-----------|----------------|-------|
| Controller | Git tag | `v1.15.1` |
| controller-chroot | Git tag (same as controller) | `v1.15.1` |
| kube-webhook-certgen | `images/kube-webhook-certgen/TAG` | `v1.6.9` |
| Helm chart | `charts/ingress-nginx/Chart.yaml` `.version` | `4.15.1` |

## Constraints

- Runtime code (Go, Lua, nginx templates, chart templates, RBAC) is never modified
- Version strings have no custom suffixes — remain upstream-compatible
- `values.yaml` is patched at workflow runtime only; the committed file retains upstream defaults
- Source of truth for provenance: OCI labels (`org.opencontainers.image.source`, `.revision`, `.version`)

## Upstream compatibility

To sync with upstream changes:

```bash
git fetch upstream --tags
git rebase upstream/main
git push origin platform/ghcr-release-flow
```
