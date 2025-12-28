# Charts and Templates

## Chart Creation

### Using `helm create`

The easiest way to start a new chart:

```bash
helm create myapp
cd myapp
```

This generates a complete chart structure with best practices.

### Manual Chart Creation

You can also create charts manually:

```bash
mkdir mychart
cd mychart
mkdir templates
touch Chart.yaml
touch values.yaml
```

## Chart.yaml

The Chart.yaml file contains metadata about your chart.

### Required Fields

```yaml
apiVersion: v2          # Chart API version (v2 for Helm 3)
name: myapp            # Chart name (lowercase, no spaces)
description: My application # Chart description
version: 0.1.0         # Chart version (SemVer)
type: application      # Chart type (application or library)
```

### Optional Fields

```yaml
appVersion: "1.0"      # Version of the application (not the chart)
home: https://myapp.com # Application homepage
sources:               # Source code URLs
  - https://github.com/user/myapp
keywords:              # Keywords for discovery
  - web
  - api
maintainers:           # Maintainer information
  - name: John Doe
    email: john@example.com
icon: https://myapp.com/icon.png
```

### Dependencies

```yaml
dependencies:
  - name: postgresql
    version: "12.0.0"
    repository: "https://charts.bitnami.com/bitnami"
    condition: postgresql.enabled
  - name: redis
    version: "17.0.0"
    repository: "https://charts.bitnami.com/bitnami"
```

## Template Syntax

Helm uses Go templates with Sprig functions.

### Basic Syntax

```yaml
# Variable substitution
name: {{ .Values.name }}

# Default values
port: {{ .Values.port | default 8080 }}

# Conditionals
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
{{- end }}

# Loops
{{- range .Values.env }}
- name: {{ .name }}
  value: {{ .value }}
{{- end }}
```

### Common Template Functions

```yaml
# String functions
name: {{ .Values.name | lower }}
namespace: {{ .Values.namespace | upper }}
host: {{ .Values.host | trimSuffix "/" }}

# Default values
replicas: {{ .Values.replicas | default 3 }}
image: {{ .Values.image | default "nginx" }}

# Required values (fail if not provided)
apiKey: {{ required "apiKey is required" .Values.apiKey }}

# Indentation
{{- indent 8 .Values.config }}
```

### Template Helpers

Create reusable template snippets in `templates/_helpers.tpl`:

```yaml
{{- define "mychart.labels" }}
labels:
  app: {{ .Chart.Name }}
  version: {{ .Chart.AppVersion }}
  release: {{ .Release.Name }}
{{- end }}

{{- define "mychart.selector" }}
selector:
  matchLabels:
    app: {{ .Chart.Name }}
{{- end }}
```

Use helpers in templates:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
  {{- include "mychart.labels" . | nindent 4 }}
spec:
  {{- include "mychart.selector" . | nindent 2 }}
  template:
    metadata:
      {{- include "mychart.labels" . | nindent 6 }}
```

## Template Best Practices

### 1. Use Whitespace Control

```yaml
# Bad - extra blank lines
{{- if .Values.enabled }}
spec:
  replicas: 3
{{- end }}

# Good - clean output
{{- if .Values.enabled }}
spec:
  replicas: 3
{{- end }}
```

### 2. Validate Required Values

```yaml
# Fail early with clear error messages
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ required "name is required" .Values.name }}
spec:
  replicas: {{ required "replicas is required" .Values.replicas }}
```

### 3. Use Helper Templates

Avoid repeating code:

```yaml
# _helpers.tpl
{{- define "mychart.fullname" -}}
{{- printf "%s-%s" .Release.Name .Chart.Name | trunc 63 | trimSuffix "-" -}}
{{- end -}}

# deployment.yaml
metadata:
  name: {{ include "mychart.fullname" . }}
```

### 4. Handle Optional Resources

```yaml
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ .Release.Name }}
spec:
  # ...
{{- end }}
```

## Complete Template Example

Here's a complete Deployment template:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "mychart.fullname" . }}
  labels:
    {{- include "mychart.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  selector:
    matchLabels:
      {{- include "mychart.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "mychart.selectorLabels" . | nindent 8 }}
    spec:
      {{- with .Values.imagePullSecrets }}
      imagePullSecrets:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        {{- if .Values.command }}
        command:
          {{- toYaml .Values.command | nindent 10 }}
        {{- end }}
        ports:
        - name: http
          containerPort: {{ .Values.service.port }}
          protocol: TCP
        {{- if .Values.env }}
        env:
          {{- toYaml .Values.env | nindent 10 }}
        {{- end }}
        {{- if .Values.volumeMounts }}
        volumeMounts:
          {{- toYaml .Values.volumeMounts | nindent 10 }}
        {{- end }}
        {{- if .Values.resources }}
        resources:
          {{- toYaml .Values.resources | nindent 10 }}
        {{- end }}
      {{- if .Values.volumes }}
      volumes:
        {{- toYaml .Values.volumes | nindent 8 }}
      {{- end }}
```

## Testing Templates

### Dry Run

Test templates without installing:

```bash
# Render templates with values
helm template myapp ./mychart

# Render with custom values
helm template myapp ./mychart -f values-prod.yaml

# Render with --set
helm template myapp ./mychart --set replicaCount=5

# Validate syntax only (no rendering)
helm lint ./mychart
```

### Debugging

```bash
# Show computed values
helm get values myapp

# Show all rendered templates
helm get manifest myapp

# Show values that would be used
helm template myapp ./mychart --debug
```

## Packaging Charts

```bash
# Package a chart
helm package ./mychart

# This creates: mychart-0.1.0.tgz

# Install from package
helm install myapp mychart-0.1.0.tgz

# Package with custom destination
helm package ./mychart --destination ./packages
```

## Next Steps

- [Values and Configuration](./03-values-and-configuration.md) - Managing configuration
- [Advanced Helm](./04-advanced-helm.md) - Advanced topics
- [Lab 02: Nginx Deployment](../labs/02-nginx-deployment/) - Hands-on practice

