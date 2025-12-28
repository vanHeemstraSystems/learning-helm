# Lab 01: Create Your First Helm Chart

## Objective

Create your first Helm chart and deploy a simple application to Kubernetes.

## Prerequisites

- Kubernetes cluster (AKS, EKS, GKE, or local)
- kubectl configured
- Helm 3 installed (`helm version`)

## Lab Steps

### Step 1: Create a Chart

```bash
# Create a new chart
helm create myfirstchart

# Explore the structure
cd myfirstchart
ls -la
tree  # if available
```

### Step 2: Examine the Chart Structure

```bash
# Look at Chart.yaml
cat Chart.yaml

# Look at values.yaml
cat values.yaml

# Look at a template
cat templates/deployment.yaml
```

### Step 3: Install the Chart

```bash
# Install with default values
helm install myapp .

# Check the release
helm list

# See what was created
kubectl get all
```

### Step 4: Customize Values

Edit `values.yaml`:

```yaml
replicaCount: 5

image:
  repository: nginx
  tag: "1.22"

service:
  type: LoadBalancer
  port: 80
```

### Step 5: Upgrade the Release

```bash
# Upgrade with new values
helm upgrade myapp .

# Verify changes
kubectl get deployment myapp
kubectl get service myapp
```

### Step 6: Check Release Status

```bash
# Get release status
helm status myapp

# Get release values
helm get values myapp

# Get release manifest
helm get manifest myapp
```

### Step 7: Rollback (Optional)

```bash
# Check release history
helm history myapp

# Rollback to previous version
helm rollback myapp

# Or rollback to specific revision
helm rollback myapp 1
```

### Step 8: Uninstall

```bash
# Uninstall the release
helm uninstall myapp

# Verify removal
helm list
kubectl get all
```

## Exercises

### Exercise 1: Modify the Template

1. Edit `templates/deployment.yaml`
2. Add an environment variable: `LOG_LEVEL=info`
3. Upgrade the release
4. Verify the environment variable is set

### Exercise 2: Create a ConfigMap

1. Create `templates/configmap.yaml`:
   ```yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: {{ .Release.Name }}-config
   data:
     app.conf: |
       server_port=8080
       log_level={{ .Values.logLevel | default "info" }}
   ```

2. Add to `values.yaml`:
   ```yaml
   logLevel: debug
   ```

3. Mount the ConfigMap in the deployment

### Exercise 3: Test Template Rendering

```bash
# Dry run to see rendered output
helm template myapp .

# Test with custom values
helm template myapp . --set replicaCount=10

# Validate chart
helm lint .
```

## Expected Output

After completing this lab, you should:

- ✅ Understand Helm chart structure
- ✅ Know how to install, upgrade, and uninstall releases
- ✅ Be able to customize values
- ✅ Understand template syntax basics

## Next Steps

- [Lab 02: Nginx Deployment](../02-nginx-deployment/) - Deploy a specific application
- Review [Concepts: Charts and Templates](../../concepts/02-charts-and-templates.md)

