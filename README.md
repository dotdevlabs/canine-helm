# Canine Helm Chart

This Helm chart deploys Canine - a Rails-based Kubernetes deployment platform.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.8+ (required for OCI registry support)
- PV provisioner support in the underlying infrastructure (if using internal PostgreSQL)
- External PostgreSQL database (if using external PostgreSQL provider)

## Chart Repository

This chart is automatically published to GitHub Container Registry (GHCR) via GitHub Actions when changes are pushed to the `main` or `master` branch.

**Chart Location:** `oci://ghcr.io/<your-github-org>/helm-charts/canine`

**Note:** Replace `<your-github-org>` with your GitHub organization or username in all examples below.

The chart is versioned using semantic versioning as defined in `Chart.yaml`. Each push to the main branch triggers a CI/CD pipeline that:
1. Lints the chart for errors
2. Validates Kubernetes manifests
3. Packages the chart
4. Publishes it to GHCR

## Installation

### Installing from GHCR (Recommended)

The chart is available from GitHub Container Registry. To install:

```bash
# Add the GHCR Helm repository (replace <your-github-org> with your organization)
helm repo add my-helm-charts oci://ghcr.io/<your-github-org>/helm-charts
helm repo update

# Install the chart
helm install canine my-helm-charts/canine \
  --namespace canine \
  --create-namespace
```

#### Install with custom values

```bash
helm install canine my-helm-charts/canine \
  --namespace canine \
  --create-namespace \
  -f custom-values.yaml
```

#### Install a specific version

```bash
helm install canine my-helm-charts/canine \
  --version 0.1.0 \
  --namespace canine \
  --create-namespace
```

### Installing from Source (Development)

For local development or testing changes:

```bash
# Clone the repository
git clone <repository-url>
cd canine-helm

# Add Bitnami repository (for PostgreSQL dependency)
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Update chart dependencies
helm dependency update

# Install from local directory
helm install canine . --namespace canine --create-namespace
```

### Add Bitnami repository (for PostgreSQL dependency)

If using internal PostgreSQL, you'll need to add the Bitnami repository:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
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

### Upgrade from GHCR

```bash
# Update the repository
helm repo update my-helm-charts

# Upgrade to latest version
helm upgrade canine my-helm-charts/canine \
  --namespace canine

# Upgrade to specific version
helm upgrade canine my-helm-charts/canine \
  --version 0.1.0 \
  --namespace canine
```

### Upgrade from Source

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

## Publishing the Chart

The chart is automatically published to GHCR via GitHub Actions when you push changes to the `main` or `master` branch. The CI/CD pipeline:

1. **Lints** the chart using `helm lint`
2. **Validates** Kubernetes manifests using `kubectl --dry-run`
3. **Packages** the chart using `helm package`
4. **Publishes** to GHCR using Helm's native OCI support

### Manual Publishing

If you need to publish manually:

```bash
# Login to GHCR
echo $GITHUB_TOKEN | helm registry login ghcr.io -u <username> --password-stdin

# Package the chart
helm package .

# Push to GHCR
helm push canine-<version>.tgz oci://ghcr.io/<organization>/helm-charts
```

## Notes

- The chart includes Bitnami's PostgreSQL as an optional dependency (enabled by default)
- You can use an external PostgreSQL provider by setting `postgresql.enabled: false` and providing `databaseUrl`
- Docker socket mounting is enabled by default for local mode operations
- Persistence is enabled for internal PostgreSQL by default
- The SECRET_KEY_BASE should be changed in production deployments
- Charts are automatically versioned based on the `version` field in `Chart.yaml`