# ingress-nginx (maintained distribution)

A **self-maintained, standalone distribution** of the ingress-nginx Ingress controller for
Kubernetes, based on the historical [`kubernetes/ingress-nginx`](https://github.com/kubernetes/ingress-nginx)
codebase. It is developed and released independently under the `Kuzmenko-Pavel` namespace,
with its own container images and Helm chart published to GitHub Container Registry (GHCR).

> **Relationship to upstream.** The upstream `kubernetes/ingress-nginx` project is
> [being retired](https://www.kubernetes.io/blog/2025/11/11/ingress-nginx-retirement/)
> (best-effort maintenance until ~March 2026, then no further releases or security fixes).
> This repository is **not** the upstream project and is **not** affiliated with or endorsed
> by the Kubernetes project, the CNCF, or F5/NGINX. It continues the codebase as an
> independent distribution. Pull requests are not sent upstream; upstream is kept only as a
> read-only historical reference.

This is a derivative work distributed under the Apache License 2.0 (see [`LICENSE`](./LICENSE)).
"NGINX" is a trademark of F5, Inc.; "Kubernetes" is a trademark of the Linux Foundation —
used here descriptively only.

## Overview

ingress-nginx is an Ingress controller for Kubernetes using [NGINX](https://www.nginx.org/)
as a reverse proxy and load balancer. See [`docs/how-it-works.md`](docs/how-it-works.md) for
the architecture and [`docs/developer-guide/code-overview.md`](docs/developer-guide/code-overview.md)
for a code map. For agent/automation guidance, see [`AGENTS.md`](./AGENTS.md).

## Usage warning

Do not use in multi-tenant Kubernetes production installations. This project assumes that
users who can create Ingress objects are administrators of the cluster. See the
[FAQ](docs/faq.md) for more.

## Install (Helm, OCI)

```bash
helm install ingress-nginx \
  oci://ghcr.io/kuzmenko-pavel/charts/ingress-nginx \
  --version 4.15.1 \
  --namespace ingress-nginx --create-namespace
```

## Published artifacts

| Artifact | Location |
|----------|----------|
| Controller image | `ghcr.io/kuzmenko-pavel/ingress-nginx/controller` |
| Controller-chroot image | `ghcr.io/kuzmenko-pavel/ingress-nginx/controller-chroot` |
| kube-webhook-certgen | `ghcr.io/kuzmenko-pavel/ingress-nginx/kube-webhook-certgen` |
| Helm chart (OCI) | `oci://ghcr.io/kuzmenko-pavel/charts/ingress-nginx` |

Release and versioning details: [`docs/maintained-distribution-release.md`](docs/maintained-distribution-release.md).
Releases use plain SemVer tags (`vX.Y.Z`); image tags match the release tag.

## Build & test

This repository uses the project's native `Makefile`. Common targets:

```bash
make build         # build controller binaries
make test          # Go unit tests
make lua-test      # Lua unit tests
make helm-test     # helm-unittest for the chart
make kind-e2e-test # e2e on a local kind cluster
make help          # list all targets
```

## Troubleshooting & support

Review the [troubleshooting docs](docs/troubleshooting.md) and the [FAQ](docs/faq.md). For
problems with this distribution, open an issue in **this** repository:
<https://github.com/Kuzmenko-Pavel/ingress-nginx/issues>.

## Contributing & security

- Contributing: see [`CONTRIBUTING.md`](./CONTRIBUTING.md).
- Security: report vulnerabilities privately via GitHub Security Advisories — see
  [`SECURITY.md`](./SECURITY.md).

## License

[Apache License 2.0](./LICENSE). Modifications from the upstream codebase are tracked in this
repository's git history.
