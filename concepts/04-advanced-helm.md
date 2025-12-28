# Advanced Helm

## Chart Dependencies

Charts can depend on other charts (sub-charts).

### Managing Dependencies

```yaml
# Chart.yaml
dependencies:
  - name: postgresql
    version: "12.0.0"
    repository: "https://charts.bitnami.com/bitnami"
    condition: postgresql.enabled
    tags:
      - database
  - name: redis
    version: "17.0.0"
    repository: "https://charts.bitnami.com/bitnami"
    condition: redis.enabled
    tags:
      - cache
```

### Dependency Commands

```bash
# Update dependencies
helm dependency update

# Build dependencies (download and package)
helm dependency build

# List dependencies
helm dependency list
```

### Using Sub-Charts

Sub-charts are installed in `charts/` directory:

```
mychart/
├── Chart.yaml
├── values.yaml
├── charts/              # Downloaded dependencies
│   ├── postgresql-12.0.0.tgz
│   └── redis-17.0.0.tgz
└── templates/
```

Configure sub-charts in values.yaml:

```yaml
# values.yaml
postgresql:
  enabled: true
  auth:
    postgresPassword: "secret"
  primary:
    persistence:
      size: 20Gi

redis:
  enabled: true
  auth:
    enabled: true
    password: "secret"
```

### Global Values

Global values are available to all sub-charts:

```yaml
# values.yaml
global:
  imageRegistry: "myregistry.com"
  storageClass: "fast-ssd"
  domain: "example.com"

postgresql:
  # Can access global.imageRegistry in templates
  image:
    registry: {{ .Values.global.imageRegistry }}
```

## Hooks

Hooks allow you to run jobs at specific points in a release lifecycle.

### Hook Types

- `pre-install` - Before templates are installed
- `post-install` - After all resources are loaded
- `pre-delete` - Before a release is deleted
- `post-delete` - After a release is deleted
- `pre-upgrade` - Before templates are upgraded
- `post-upgrade` - After an upgrade is complete
- `pre-rollback` - Before a rollback is performed
- `post-rollback` - After a rollback is complete

### Hook Example

```yaml
# templates/migrations-job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ .Release.Name }}-migrations
  annotations:
    "helm.sh/hook": pre-upgrade
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": before-hook-creation,hook-succeeded
spec:
  template:
    spec:
      containers:
      - name: migrations
        image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
        command: ["npm", "run", "migrate"]
      restartPolicy: Never
```

### Hook Weight

Hooks execute in weight order (ascending):

```yaml
annotations:
  "helm.sh/hook-weight": "-5"  # Executes first
```

### Hook Delete Policy

Control when hooks are deleted:

- `before-hook-creation` - Delete previous hook before new one
- `hook-succeeded` - Delete after successful execution
- `hook-failed` - Delete after failed execution

## Library Charts

Library charts are charts meant to be used as dependencies but don't create resources themselves.

```yaml
# Chart.yaml
apiVersion: v2
name: mylib
description: A library chart
type: library  # Not application
version: 0.1.0
```

Library charts contain only templates (no default values):

```yaml
# templates/_helpers.tpl (library chart)
{{- define "mylib.commonLabels" }}
labels:
  app: {{ .Chart.Name }}
  version: {{ .Chart.AppVersion }}
{{- end }}
```

Use in dependent charts:

```yaml
# templates/deployment.yaml (your chart)
metadata:
  {{- include "mylib.commonLabels" . | nindent 4 }}
```

## Testing

### Test Hooks

Helm test hooks run after installation:

```yaml
# templates/tests/test-connection.yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ .Release.Name }}-test-connection"
  annotations:
    "helm.sh/hook": test
spec:
  containers:
  - name: wget
    image: busybox
    command: ['wget']
    args: ['{{ .Release.Name }}:{{ .Values.service.port }}']
  restartPolicy: Never
```

Run tests:

```bash
helm test myapp
```

## Chart Repositories

### Add Repository

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

### Create Your Own Repository

1. Package charts:
   ```bash
   helm package ./mychart
   ```

2. Create index:
   ```bash
   helm repo index . --url https://myrepo.com/charts
   ```

3. Host the repository (GitHub Pages, S3, etc.)

4. Add to clients:
   ```bash
   helm repo add myrepo https://myrepo.com/charts
   ```

## Plugins

Helm plugins extend functionality:

```bash
# Install plugin
helm plugin install https://github.com/hickeyma/helm-mapkubeapis

# List plugins
helm plugin list

# Use plugin
helm mapkubeapis myapp
```

## Best Practices

### 1. Version Your Charts

Follow SemVer:
- `1.0.0` - Major version (breaking changes)
- `1.1.0` - Minor version (new features)
- `1.1.1` - Patch version (bug fixes)

### 2. Use .helmignore

Like `.gitignore`, exclude unnecessary files:

```
.helmignore
*.swp
*.bak
.env
.git/
```

### 3. Lint Your Charts

```bash
helm lint ./mychart
```

### 4. Test Before Release

```bash
# Dry run
helm install --dry-run --debug myapp ./mychart

# Template test
helm template myapp ./mychart | kubectl apply --dry-run=client -f -

# Install and test
helm install myapp ./mychart
helm test myapp
```

### 5. Document Values

Use comments in values.yaml:

```yaml
# Number of application replicas
replicaCount: 3

# Container image configuration
image:
  # Docker image repository
  repository: nginx
  # Image tag (defaults to Chart.AppVersion)
  tag: ""
```

### 6. Use Named Templates

Create reusable templates in `_helpers.tpl`:

```yaml
{{- define "mychart.fullname" -}}
{{- printf "%s-%s" .Release.Name .Chart.Name | trunc 63 | trimSuffix "-" -}}
{{- end -}}
```

### 7. Handle Resource Limits

Always set resource requests and limits:

```yaml
# values.yaml
resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 512Mi
```

## Troubleshooting

### Debug Template Rendering

```bash
# See rendered output
helm template myapp ./mychart --debug

# Show computed values
helm get values myapp

# Show all manifests
helm get manifest myapp
```

### Common Issues

1. **Template Syntax Errors**
   ```bash
   helm lint ./mychart
   ```

2. **Value Not Found**
   Use defaults: `{{ .Values.port | default 8080 }}`

3. **Dependency Issues**
   ```bash
   helm dependency update
   ```

4. **Release Conflicts**
   ```bash
   # List releases
   helm list --all
   
   # Clean up failed releases
   helm uninstall myapp
   ```

## Next Steps

- [Lab 04: Chart Dependencies](../labs/04-chart-dependencies/) - Practice with dependencies
- Review [Examples](../examples/) for reference implementations

