# Helm Common Library

A generic implementation of a Helm **Library Chart** that provides standard templates for Kubernetes resources. This library allows other charts to share common patterns for `Deployment`, `Service`, and `Ingress` resources, reducing code duplication and maintenance effort.

## Features

- **Standardized Deployment**: Provides a robust `Deployment` template with support for autoscaling, probes, security contexts, and more.
- **Service Configuration**: A reusable `Service` template.
- **Ingress Support**: A reusable `Ingress` template.
- **Helper Functions**: Common named templates for labels, selectors, and naming conventions.

## Prerequisities

- Helm v3+
- Kubernetes 1.12+

## Usage

To use this library in your own Helm chart, follow these steps:

### 1. Add Dependency to `Chart.yaml`

In your consumer chart's `Chart.yaml` (e.g., `service-a/Chart.yaml`), add `common-lib` as a dependency. 
*Note: Since this is a local library in this repository, you might reference it by file path relative to your chart, or by version if hosted in a repository.*

```yaml
dependencies:
  - name: common-lib
    version: 0.1.8
    repository: file://../common-lib
    # Or if hosted:
    # repository: http://your-chart-repo/
```

### 2. Implement Templates

In your consumer chart's templates (e.g., `service-a/templates/deployment.yaml`), simply include the library templates.

**Example `deployment.yaml`:**
```yaml
{{- include "lib.deployment" . -}}
```

**Example `service.yaml`:**
```yaml
{{- include "lib.service" . -}}
```

**Example `ingress.yaml`:**
```yaml
{{- include "lib.ingress" . -}}
```

### 3. Configure `values.yaml`

Ensure your consumer chart's `values.yaml` has the expected structure. The library templates rely on standard Helm conventions:

```yaml
replicaCount: 1

image:
  repository: nginx
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: ""

imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""

serviceAccount:
  create: true
  annotations: {}
  name: ""

podAnnotations: {}

podSecurityContext: {}
  # fsGroup: 2000

securityContext: {}
  # capabilities:
  #   drop:
  #   - ALL
  # readOnlyRootFilesystem: true
  # runAsNonRoot: true
  # runAsUser: 1000

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: false
  className: ""
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: chart-example.local
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls: []
  #  - secretName: chart-example-tls
  #    hosts:
  #      - chart-example.local

resources: {}
  # limits:
  #   cpu: 100m
  #   memory: 128Mi
  # requests:
  #   cpu: 100m
  #   memory: 128Mi

autoscaling:
  enabled: false
  minReplicas: 1
  maxReplicas: 100
  targetCPUUtilizationPercentage: 80
  # targetMemoryUtilizationPercentage: 80

nodeSelector: {}

tolerations: []

affinity: {}

livenessProbe:
  httpGet:
    path: /
readinessProbe:
  httpGet:
    path: /
```

## Available Templates

| Template Name       | Description |
| ------------------- | ----------- |
| `lib.deployment`    | Generates a full `apps/v1` Deployment resource. |
| `lib.service`       | Generates a `v1` Service resource. |
| `lib.ingress`       | Generates a `networking.k8s.io/v1` Ingress resource (check `_ingress.yaml` for implementation details). |
| `lib.fullname`      | Generates the full name of the resource. |
| `lib.name`          | Generates the name of the chart. |
| `lib.labels`        | Generates standard usage labels. |
| `lib.selectorLabels`| Generates selector labels. |

## Maintaining

To update the library:
1. Modify templates in `common-lib/templates/`.
2. Bump the version in `common-lib/Chart.yaml`.
3. Update specific service charts to depend on the new version.
