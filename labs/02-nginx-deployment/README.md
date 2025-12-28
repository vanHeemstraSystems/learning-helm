# Lab 02: Nginx Deployment with Helm

## Objective

Create a Helm chart to deploy nginx with customizable configuration.

## Prerequisites

- Completed Lab 01
- Kubernetes cluster running
- Helm 3 installed

## Lab Steps

### Step 1: Create Chart from Scratch

```bash
# Create chart structure manually
mkdir nginx-chart
cd nginx-chart
mkdir templates
touch Chart.yaml
touch values.yaml
```

### Step 2: Create Chart.yaml

```yaml
apiVersion: v2
name: nginx-chart
description: A Helm chart for nginx
type: application
version: 0.1.0
appVersion: "1.21"
```

### Step 3: Create values.yaml

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
    - host: nginx.local
      paths:
        - path: /
          pathType: Prefix
  tls: []

resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 200m
    memory: 256Mi
```

### Step 4: Create Deployment Template

Create `templates/deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
  labels:
    app: {{ .Chart.Name }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Chart.Name }}
  template:
    metadata:
      labels:
        app: {{ .Chart.Name }}
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - name: http
          containerPort: 80
          protocol: TCP
        resources:
          {{- toYaml .Values.resources | nindent 10 }}
```

### Step 5: Create Service Template

Create `templates/service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}
  labels:
    app: {{ .Chart.Name }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http
      protocol: TCP
      name: http
  selector:
    app: {{ .Chart.Name }}
```

### Step 6: Install the Chart

```bash
# Install
helm install my-nginx .

# Verify
helm list
kubectl get deployment
kubectl get service
kubectl get pods
```

### Step 7: Upgrade with Different Values

```bash
# Upgrade replica count
helm upgrade my-nginx . --set replicaCount=5

# Upgrade image version
helm upgrade my-nginx . --set image.tag=1.22

# Upgrade service type
helm upgrade my-nginx . --set service.type=LoadBalancer
```

### Step 8: Create Values Files for Environments

Create `values-dev.yaml`:

```yaml
replicaCount: 1
image:
  tag: "1.21"
resources:
  requests:
    cpu: 50m
    memory: 64Mi
  limits:
    cpu: 100m
    memory: 128Mi
```

Create `values-prod.yaml`:

```yaml
replicaCount: 5
image:
  tag: "1.22"
resources:
  requests:
    cpu: 200m
    memory: 256Mi
  limits:
    cpu: 500m
    memory: 512Mi
```

Install with environment-specific values:

```bash
# Development
helm install nginx-dev . -f values-dev.yaml

# Production
helm install nginx-prod . -f values-prod.yaml
```

## Exercises

### Exercise 1: Add ConfigMap for nginx.conf

1. Create a ConfigMap template for custom nginx configuration
2. Mount it in the deployment
3. Test with a custom nginx.conf

### Exercise 2: Add Ingress

1. Create an Ingress template (conditionally based on `values.ingress.enabled`)
2. Enable ingress in values
3. Install and verify ingress creation

### Exercise 3: Add Health Checks

1. Add liveness and readiness probes to the deployment
2. Test pod health

### Exercise 4: Package the Chart

```bash
# Package the chart
helm package .

# Install from package
helm install my-nginx nginx-chart-0.1.0.tgz
```

## Expected Output

After completing this lab, you should have:

- ✅ A working nginx Helm chart
- ✅ Understanding of template creation
- ✅ Experience with values files
- ✅ Knowledge of upgrading releases

## Next Steps

- [Lab 03: Multi-Tier Application](../03-multi-tier-app/) - Complex multi-component application
- Review [Concepts: Values and Configuration](../../concepts/03-values-and-configuration.md)

