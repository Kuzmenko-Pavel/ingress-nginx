# Security Policy

This is a self-maintained distribution of ingress-nginx (see [`README.md`](./README.md)).
It is not the upstream `kubernetes/ingress-nginx` project — do not use the Kubernetes
security channels to report issues about this distribution.

## Reporting a Vulnerability

Please report security vulnerabilities **privately** via GitHub Security Advisories:

➡️ <https://github.com/Kuzmenko-Pavel/ingress-nginx/security/advisories/new>

Do not open a public issue for an undisclosed vulnerability. You will receive a response as
soon as possible; please allow time for triage and a fix before any public disclosure.

When reporting, please include where practical:

- affected version(s) / image tag(s) and Helm chart version,
- a description of the issue and its impact,
- steps to reproduce or a proof of concept,
- any relevant scanner output (e.g. Trivy) and CVE identifiers.

## Supported Versions

Only the latest released `vX.Y.Z` tag of this distribution receives fixes. Older releases are
provided as-is. Published images are scanned with Trivy via the repository's
`vulnerability-scans` workflow.

## Upstream provenance

This distribution is derived from `kubernetes/ingress-nginx`. Upstream is
[being retired](https://www.kubernetes.io/blog/2025/11/11/ingress-nginx-retirement/) and no
longer ships security fixes after its retirement window; security maintenance here is
independent and best-effort by this repository's maintainer.
