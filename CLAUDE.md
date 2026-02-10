# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains a **generic Helm chart** (`generic/`) for deploying containerized applications to Kubernetes. It uses Helm v2 API (Helm 3 compatible) with Go templating. The chart is published as an OCI artifact to GitHub Container Registry (ghcr.io).

## Common Commands

```bash
# Lint the chart
helm lint generic/

# Render templates locally (dry run)
helm template my-release generic/

# Render with custom values
helm template my-release generic/ -f custom-values.yaml

# Install to a cluster
helm install my-release generic/

# Upgrade an existing release
helm upgrade my-release generic/

# Package the chart
helm package generic/

# Pull from GHCR (OCI)
helm pull oci://ghcr.io/<owner>/charts/generic --version <version>
```

## CI/CD

- **`.github/workflows/push-chart.yaml`** — Pushes the chart as an OCI artifact to ghcr.io when the `version` field in `generic/Chart.yaml` changes on `main`. Uses the built-in `GITHUB_TOKEN` for authentication.

## Architecture

The chart lives entirely under `generic/` and produces a standard Kubernetes deployment stack:

- **`Chart.yaml`** — Chart metadata (name: `generic`, version: `0.1.5`, appVersion: `1.28.2`)
- **`values.yaml`** — All configurable values with defaults (nginx image, service/probes disabled by default, etc.)
- **`templates/_helpers.tpl`** — Shared template helpers prefixed with `generic.` (name, fullname, labels, selectorLabels, serviceAccountName, dockerconfigjson)
- **`templates/deployment.yaml`** — Core Deployment resource with probes, volumes, security contexts
- **`templates/service.yaml`** — Service resource (ClusterIP/NodePort/LoadBalancer)
- **`templates/ingress.yaml`** — Ingress with multi-version K8s API compatibility (1.14+)
- **`templates/httproute.yaml`** — Gateway API HTTPRoute for Envoy Gateway / Gateway API routing
- **`templates/hpa.yaml`** — Conditional HPA for autoscaling
- **`templates/serviceaccount.yaml`** — Conditional ServiceAccount
- **`templates/imagepullsecret.yaml`** — Conditional Docker registry credentials secret
- **`templates/pdb.yaml`** — Conditional PodDisruptionBudget

Key design patterns:
- All optional resources (service, ingress, HTTPRoute, HPA, service account, image pull secret, PDB) are conditionally created via `enabled` or `create` flags in values
- `generic.fullname` defaults to the release name (no chart name suffix), overridable via `fullnameOverride`
- Container name defaults to the release name, overridable via `containerName`
- Helper templates in `_helpers.tpl` handle name truncation to 63 chars (DNS spec)
- The ingress template uses `semverCompare` against `.Capabilities.KubeVersion` to select the correct API version
- The HTTPRoute template auto-wires `backendRefs` to the chart's service when not explicitly specified
- Probes can be disabled by setting `livenessProbe`/`readinessProbe` to `null`
