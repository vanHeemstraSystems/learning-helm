# Values and Configuration

## Understanding Values

Values are the configuration data that gets injected into templates. They provide customization without modifying templates.

## values.yaml

The default configuration file in your chart:

```yaml
# values.yaml
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

## Providing Values

Multiple ways to provide values (in order of precedence, highest first):

### 1. Command Line (`--set`)

```bash
helm install myapp ./mychart --set replicaCount=5

# Multiple values
helm install myapp ./mychart \
  --set replicaCount=5 \
  --set image.tag=1.22

# Nested values
helm install myapp ./mychart \
  --set image.repository=myregistry/nginx \
  --set image.tag=1.22

# Arrays/lists
helm install myapp ./mychart \
  --set ingress.hosts[0].host=app1.local \
  --set ingress.hosts[1].host=app2.local
```

### 2. Value Files (`-f`)

```bash
# Single file
helm install myapp ./mychart -f values-prod.yaml

# Multiple files (later files override earlier)
helm install myapp ./mychart \
  -f values.yaml \
  -f values-prod.yaml \
  -f values-overrides.yaml
```

### 3. Environment-Specific Values

Common pattern:

```
mychart/
├── values.yaml          # Base/default values
├── values-dev.yaml      # Development overrides
├── values-staging.yaml  # Staging overrides
└── values-prod.yaml     # Production overrides
```

```yaml
# values-dev.yaml
replicaCount: 1
image:
  tag: "dev"
resources:
  requests:
    cpu: 100m
    memory: 128Mi
```

```yaml
# values-prod.yaml
replicaCount: 5
image:
  tag: "stable"
resources:
  requests:
    cpu: 500m
    memory: 512Mi
  limits:
    cpu: 1000m
    memory: 1Gi
```

### 4. Default values.yaml

Lowest precedence, used as base configuration.

## Value File Best Practices

### 1. Organize by Component

```yaml
# values.yaml
# Frontend configuration
frontend:
  replicaCount: 3
  image:
    repository: frontend
    tag: "1.0"
  
# Backend configuration
backend:
  replicaCount: 5
  image:
    repository: backend
    tag: "1.0"

# Database configuration
database:
  enabled: true
  type: postgresql
  storage: 20Gi
```

### 2. Use Sensible Defaults

```yaml
# Good defaults
replicaCount: 3
image:
  pullPolicy: IfNotPresent  # Safe default
resources:
  requests:
    cpu: 100m
    memory: 128Mi
```

### 3. Document with Comments

```yaml
# Number of replicas to deploy
replicaCount: 3

# Container image configuration
image:
  # Docker image repository
  repository: nginx
  # Image pull policy: Always, IfNotPresent, Never
  pullPolicy: IfNotPresent
  # Image tag (defaults to Chart.AppVersion if not set)
  tag: ""
```

### 4. Use Type Hints

```yaml
# String (explicit)
name: "myapp"
version: "1.0"

# Number (no quotes)
replicaCount: 3
port: 8080

# Boolean
enabled: true
debug: false

# Array
env:
  - name: ENV1
    value: "value1"
  - name: ENV2
    value: "value2"

# Object
resources:
  requests:
    cpu: 100m
    memory: 128Mi
```

## Advanced Value Patterns

### Conditional Values

```yaml
# values.yaml
ingress:
  enabled: true
  className: "nginx"

# Only create ingress if enabled
# templates/ingress.yaml
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
# ...
{{- end }}
```

### Value Merging

Helm merges values, not replaces:

```yaml
# values.yaml
resources:
  requests:
    cpu: 100m

# values-prod.yaml (only overrides what's specified)
resources:
  requests:
    memory: 512Mi
  limits:
    cpu: 500m

# Result: Both cpu and memory in requests, plus limits
resources:
  requests:
    cpu: 100m      # from base
    memory: 512Mi  # from prod
  limits:
    cpu: 500m      # from prod
```

### Value Validation

Use template functions to validate:

```yaml
# templates/deployment.yaml
{{- if not (hasKey .Values "replicaCount") }}
  {{- fail "replicaCount is required" }}
{{- end }}

{{- if lt .Values.replicaCount 1 }}
  {{- fail "replicaCount must be at least 1" }}
{{- end }}
```

## Using Values in Templates

### Simple Access

```yaml
name: {{ .Values.name }}
replicas: {{ .Values.replicaCount }}
```

### Nested Access

```yaml
image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

### With Defaults

```yaml
port: {{ .Values.service.port | default 8080 }}
```

### Required Values

```yaml
apiKey: {{ required "apiKey is required" .Values.apiKey }}
```

### Complex Structures

```yaml
{{- range .Values.env }}
- name: {{ .name }}
  value: {{ .value }}
{{- end }}
```

## Managing Secrets

### Never Put Secrets in values.yaml

Instead, use:

1. **Kubernetes Secrets** (recommended)
   ```bash
   kubectl create secret generic my-secret \
     --from-literal=password=secret123
   ```
   Then reference in values:
   ```yaml
   # values.yaml
   secrets:
     existingSecret: my-secret
   ```

2. **External Secret Management** (e.g., Sealed Secrets, External Secrets Operator)

3. **Value Files** (only for development, never commit)
   ```bash
   helm install myapp ./mychart -f secrets.yaml
   # secrets.yaml is in .gitignore
   ```

## Environment Variables from Values

```yaml
# values.yaml
env:
  - name: DATABASE_HOST
    value: "postgres.default.svc.cluster.local"
  - name: LOG_LEVEL
    value: "info"
  - name: API_KEY
    valueFrom:
      secretKeyRef:
        name: my-secret
        key: api-key
```

```yaml
# templates/deployment.yaml
containers:
- name: app
  env:
    {{- toYaml .Values.env | nindent 4 }}
```

## Next Steps

- [Advanced Helm](./04-advanced-helm.md) - Advanced topics
- [Lab 03: Multi-Tier Application](../labs/03-multi-tier-app/) - Complex configuration practice

