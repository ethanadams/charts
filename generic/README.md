# Generic Helm Chart

A general-purpose Helm chart for deploying containerized applications to Kubernetes.

## Installation

### From OCI registry

```bash
helm install my-app oci://ghcr.io/ethanadams/charts/generic --version 0.1.5
```

### From source

```bash
helm install my-app ./generic
```

## Usage

Resource names default to the Helm release name. For example:

```bash
helm install my-app oci://ghcr.io/ethanadams/charts/generic
```

Creates a Deployment and container named `my-app`.

## Configuration

| Parameter | Description | Default |
|---|---|---|
| `replicaCount` | Number of pod replicas | `1` |
| `image.repository` | Container image repository | `nginx` |
| `image.tag` | Container image tag | Chart `appVersion` |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `containerName` | Override the container name | Release name |
| `fullnameOverride` | Override all resource names | Release name |
| `nameOverride` | Override the chart name in labels | `generic` |
| **Service** | | |
| `service.enabled` | Create a Service | `false` |
| `service.type` | Service type | `ClusterIP` |
| `service.port` | Service port | `80` |
| `service.containerPort` | Container port | `80` |
| **Ingress** | | |
| `ingress.enabled` | Enable Ingress resource | `false` |
| `ingress.className` | Ingress class name | `""` |
| `ingress.annotations` | Ingress annotations | `{}` |
| `ingress.hosts` | Ingress host rules | See `values.yaml` |
| `ingress.tls` | Ingress TLS configuration | `[]` |
| **HTTPRoute (Gateway API)** | | |
| `httpRoute.enabled` | Enable HTTPRoute resource | `false` |
| `httpRoute.annotations` | HTTPRoute annotations | `{}` |
| `httpRoute.parentRefs` | Gateway references | `[]` |
| `httpRoute.hostnames` | Hostnames to match | `[]` |
| `httpRoute.rules` | Routing rules | PathPrefix `/` |
| **Probes** | | |
| `livenessProbe` | Liveness probe config | `null` |
| `readinessProbe` | Readiness probe config | `null` |
| `startupProbe` | Startup probe config | `null` |
| **Pod** | | |
| `env` | Environment variables | `[]` |
| `envFrom` | Environment variable sources | `[]` |
| `command` | Container command override | `[]` |
| `args` | Container args override | `[]` |
| `volumes` | Additional volumes | `[]` |
| `volumeMounts` | Additional volume mounts | `[]` |
| `initContainers` | Init containers | `[]` |
| `podAnnotations` | Pod annotations | `{}` |
| `podLabels` | Additional pod labels | `{}` |
| `podSecurityContext` | Pod security context | `{}` |
| `securityContext` | Container security context | `{}` |
| `resources` | CPU/memory resources | `{}` |
| `nodeSelector` | Node selector | `{}` |
| `tolerations` | Tolerations | `[]` |
| `affinity` | Affinity rules | `{}` |
| **Optional resources** | | |
| `autoscaling.enabled` | Enable HPA | `false` |
| `serviceAccount.create` | Create a ServiceAccount | `false` |
| `imageCredentials.create` | Create image pull secret | `false` |
| `podDisruptionBudget.enabled` | Create PDB | `false` |

## HTTPRoute (Gateway API)

For clusters using [Gateway API](https://gateway-api.sigs.k8s.io/) (e.g., Envoy Gateway), enable the HTTPRoute instead of (or alongside) Ingress:

```yaml
httpRoute:
  enabled: true
  parentRefs:
    - name: my-gateway
  hostnames:
    - "app.example.com"
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /
```

When `backendRefs` is omitted from a rule, it automatically routes to this chart's Service.

## Examples

### Basic deployment

```bash
helm install my-app ./generic \
  --set image.repository=my-registry/my-app \
  --set image.tag=v1.0.0
```

### With Ingress

```bash
helm install my-app ./generic \
  --set image.repository=my-registry/my-app \
  --set image.tag=v1.0.0 \
  --set ingress.enabled=true \
  --set ingress.hosts[0].host=app.example.com \
  --set ingress.hosts[0].paths[0].path=/ \
  --set ingress.hosts[0].paths[0].pathType=Prefix
```

### With Gateway API

```bash
helm install my-app ./generic \
  --set image.repository=my-registry/my-app \
  --set image.tag=v1.0.0 \
  --set httpRoute.enabled=true \
  --set httpRoute.parentRefs[0].name=my-gateway \
  --set httpRoute.hostnames[0]=app.example.com
```
