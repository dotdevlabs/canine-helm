# Canine Helm Chart

This Helm chart deploys Canine - a Rails-based Kubernetes deployment platform.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.2.0+
- PV provisioner support in the underlying infrastructure (if using internal PostgreSQL)
- External PostgreSQL database (if using external PostgreSQL provider)

## Installation

### Add Bitnami repository (for PostgreSQL dependency)

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

### Install the chart

```bash
# From the helm/canine directory
helm dependency update
helm install canine . --namespace canine --create-namespace
```

### Install with custom values

```bash
helm install canine . --namespace canine --create-namespace -f custom-values.yaml
```

## Configuration

The following table lists the configurable parameters and their default values:

### Image Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `image.repository` | Canine image repository | `chriszhu12/canine` |
| `image.tag` | Canine image tag | `latest` |
| `image.pullPolicy` | Image pull policy | `Always` |
| `imagePullSecrets` | Image pull secrets | `[]` |

### Canine Application

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of replicas (ignored if autoscaling enabled) | `1` |
| `canine.port` | Application port | `3000` |
| `canine.localMode` | Enable local mode | `true` |
| `canine.secretKeyBase` | Rails secret key base | `a38fcb39d60f9d146d2a0053a25024b9` |
| `canine.auth.username` | Admin username | `admin` |
| `canine.auth.password` | Admin password | `changeme` |
| `canine.allowedHostname` | Allowed hostnames for Rails host authorization (use "*" for all) | `"*"` |

### Worker Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `worker.enabled` | Enable background worker deployment | `true` |
| `worker.replicaCount` | Number of worker replicas | `1` |
| `worker.maxThreads` | Maximum threads for GoodJob | `5` |
| `worker.queues` | Comma-separated list of queues to process (use "*" for all) | `"*"` |
| `worker.resources` | Worker resource limits and requests | `{}` |

### Service Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `service.type` | Kubernetes service type | `ClusterIP` |
| `service.port` | Service port | `3000` |

### Ingress Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `ingress.enabled` | Enable ingress | `false` |
| `ingress.className` | Ingress class name | `""` |
| `ingress.annotations` | Ingress annotations | `{}` |
| `ingress.hosts[].host` | Hostname | `canine.local` |
| `ingress.hosts[].paths[].path` | Path | `/` |
| `ingress.hosts[].paths[].pathType` | Path type | `ImplementationSpecific` |
| `ingress.tls` | TLS configuration | `[]` |

### PostgreSQL Configuration

The chart supports two options for PostgreSQL:

#### Option 1: Internal PostgreSQL (Bitnami)

| Parameter | Description | Default |
|-----------|-------------|---------|
| `postgresql.enabled` | Enable internal PostgreSQL | `true` |
| `postgresql.auth.username` | PostgreSQL username | `postgres` |
| `postgresql.auth.password` | PostgreSQL password | `password` |
| `postgresql.auth.database` | PostgreSQL database | `canine_production` |
| `postgresql.primary.persistence.size` | PVC size | `8Gi` |

#### Option 2: External PostgreSQL Provider

| Parameter | Description | Default |
|-----------|-------------|---------|
| `databaseUrl` | PostgreSQL connection URL from external provider | `""` |

**Usage:**
- To use internal PostgreSQL: Set `postgresql.enabled: true` (default) and leave `databaseUrl` empty
- To use external PostgreSQL: Set `postgresql.enabled: false` and provide `databaseUrl`

Example for external PostgreSQL:
```yaml
postgresql:
  enabled: false
databaseUrl: "postgres://username:password@host:5432/database"
```

### Autoscaling Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `autoscaling.enabled` | Enable Horizontal Pod Autoscaler | `false` |
| `autoscaling.minReplicas` | Minimum number of replicas | `1` |
| `autoscaling.maxReplicas` | Maximum number of replicas | `10` |
| `autoscaling.targetCPUUtilizationPercentage` | Target CPU utilization percentage | `80` |
| `autoscaling.targetMemoryUtilizationPercentage` | Target memory utilization percentage | `nil` |

**Note:** When autoscaling is enabled, `replicaCount` is ignored and the HPA manages replica count.

### Resource Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `resources` | Resource limits and requests for main deployment | `{}` |

Example:
```yaml
resources:
  limits:
    cpu: 1000m
    memory: 1024Mi
  requests:
    cpu: 500m
    memory: 512Mi
```

### Pod Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `podAnnotations` | Additional pod annotations | `{}` |
| `podLabels` | Additional pod labels | `{}` |
| `securityContext` | Security context for containers | `{}` |
| `nodeSelector` | Node selector for pod scheduling | `{}` |
| `tolerations` | Tolerations for pod scheduling | `[]` |
| `affinity` | Affinity rules for pod scheduling | `{}` |

### Chart Name Overrides

| Parameter | Description | Default |
|-----------|-------------|---------|
| `nameOverride` | Override the chart name | `""` |
| `fullnameOverride` | Override the full name | `""` |

## Uninstalling the Chart

```bash
helm uninstall canine --namespace canine
```

## Upgrading the Chart

```bash
helm upgrade canine . --namespace canine
```

## Development

To use a local Docker image:

1. Build your Docker image locally
2. Update the values:
```yaml
image:
  repository: canine
  tag: dev
  pullPolicy: Never
```

## Notes

- The chart includes Bitnami's PostgreSQL as an optional dependency (enabled by default)
- You can use an external PostgreSQL provider by setting `postgresql.enabled: false` and providing `databaseUrl`
- Docker socket mounting is enabled by default for local mode operations
- Persistence is enabled for internal PostgreSQL by default
- The SECRET_KEY_BASE should be changed in production deployments