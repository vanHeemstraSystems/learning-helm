# Helm Fundamentals

## What is Helm?

Helm is the **package manager for Kubernetes**. Think of it like:
- **apt/yum** for Linux packages
- **npm** for Node.js packages
- **pip** for Python packages
- **Homebrew** for macOS

But instead of managing software packages, Helm manages Kubernetes applications.

## Why Use Helm?

### Without Helm
```yaml
# Deploying an application requires multiple files:
- deployment.yaml
- service.yaml
- configmap.yaml
- secret.yaml
- ingress.yaml
- persistentvolumeclaim.yaml
# ... and more
```

### With Helm
```bash
# Single command to deploy everything:
helm install myapp ./myapp-chart

# Easy upgrades:
helm upgrade myapp ./myapp-chart

# Simple rollbacks:
helm rollback myapp 1
```

## Key Concepts

### Chart

A **Chart** is a Helm package. It contains:
- All the Kubernetes resource definitions needed to run an application
- Template files for customization
- Metadata about the chart
- Default configuration values

Think of it as a "blueprint" for your Kubernetes application.

```
mychart/
├── Chart.yaml          # Chart metadata
├── values.yaml         # Default configuration
├── templates/          # Kubernetes manifests (templated)
│   ├── deployment.yaml
│   ├── service.yaml
│   └── configmap.yaml
└── charts/             # Chart dependencies (optional)
```

### Release

A **Release** is an instance of a chart deployed to a Kubernetes cluster.

- One chart can have multiple releases
- Each release has a unique name
- Releases track version history for rollbacks

```bash
# Install creates a release
helm install myapp ./mychart

# This creates a release named "myapp" from the "mychart" chart
```

### Repository

A **Repository** is a place where charts can be stored and shared.

- Public repositories (e.g., Artifact Hub)
- Private repositories (company internal)
- Local repositories (file system)

```bash
# Add a repository
helm repo add bitnami https://charts.bitnami.com/bitnami

# Search for charts
helm search repo nginx

# Install from repository
helm install nginx bitnami/nginx
```

## Helm Architecture

```
┌─────────────────┐
│   Helm Client   │  (helm CLI on your machine)
│                 │
│  - helm install │
│  - helm upgrade │
│  - helm list    │
└────────┬────────┘
         │
         │ HTTP/gRPC
         │
         ▼
┌─────────────────┐
│ Helm Library    │  (Go library - not installed separately in Helm 3)
└────────┬────────┘
         │
         │ Renders templates
         │ Applies to cluster
         │
         ▼
┌─────────────────┐
│ Kubernetes API  │
│                 │
│  - Creates      │
│    Resources    │
│  - Stores       │
│    Release Info │
│    (as Secrets) │
└─────────────────┘
```

## Helm 3 vs Helm 2

**Important:** Helm 2 is deprecated. This guide focuses on Helm 3.

### Key Differences:

| Feature | Helm 2 | Helm 3 |
|---------|--------|--------|
| **Tiller** | Required (server-side component) | Removed (client-only) |
| **Release Storage** | ConfigMaps | Secrets |
| **Chart API Version** | v1 | v2 |
| **Security** | Requires cluster-wide RBAC | Uses kubectl credentials |

Helm 3 is simpler and more secure!

## Basic Helm Commands

### Installing a Chart
```bash
# From a local chart directory
helm install myapp ./mychart

# From a repository
helm install nginx bitnami/nginx

# With custom values
helm install myapp ./mychart -f values-prod.yaml

# With inline values
helm install myapp ./mychart --set replicaCount=5
```

### Listing Releases
```bash
# List all releases
helm list

# List releases in a namespace
helm list -n production

# List all releases (including deleted)
helm list --all
```

### Upgrading a Release
```bash
# Upgrade with new chart version
helm upgrade myapp ./mychart

# Upgrade with new values
helm upgrade myapp ./mychart --set replicaCount=10

# Upgrade with values file
helm upgrade myapp ./mychart -f values-prod.yaml
```

### Rolling Back
```bash
# See release history
helm history myapp

# Rollback to previous version
helm rollback myapp

# Rollback to specific revision
helm rollback myapp 3
```

### Uninstalling
```bash
# Delete a release
helm uninstall myapp
```

## Chart Structure

When you run `helm create mychart`, you get this structure:

```
mychart/
├── Chart.yaml              # Chart metadata
├── values.yaml             # Default configuration values
├── templates/              # Template files
│   ├── deployment.yaml     # Deployment template
│   ├── service.yaml        # Service template
│   ├── ingress.yaml        # Ingress template (optional)
│   ├── serviceaccount.yaml # ServiceAccount template
│   ├── _helpers.tpl        # Helper templates
│   └── tests/              # Test files
│       └── test-connection.yaml
├── charts/                 # Chart dependencies (sub-charts)
└── .helmignore            # Files to ignore when packaging
```

### Chart.yaml Example
```yaml
apiVersion: v2
name: myapp
description: A Helm chart for My Application
type: application
version: 0.1.0
appVersion: "1.0"
dependencies: []
maintainers:
  - name: Your Name
    email: your.email@example.com
```

### values.yaml Example
```yaml
replicaCount: 3

image:
  repository: nginx
  pullPolicy: IfNotPresent
  tag: "1.21"

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: false
  className: "nginx"
  annotations: {}
  hosts:
    - host: myapp.local
      paths:
        - path: /
          pathType: Prefix
```

## Templates

Templates use Go's templating language with Sprig functions for Kubernetes manifests.

### Simple Template Example
```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        ports:
        - containerPort: {{ .Values.service.port }}
```

### Template Variables

- `.Values` - Values from values.yaml or --set
- `.Release` - Release metadata (name, namespace, revision)
- `.Chart` - Chart metadata from Chart.yaml
- `.Template` - Current template information

## Next Steps

- [Charts and Templates](./02-charts-and-templates.md) - Deep dive into chart creation
- [Values and Configuration](./03-values-and-configuration.md) - Managing configuration
- [Lab 01: Create Your First Chart](../labs/01-first-chart/) - Hands-on practice

