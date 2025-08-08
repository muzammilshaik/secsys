---
author: muju
title: "Kubernetes Helm Charts"
description: "Master Kubernetes package management with Helm Charts - learn how to create, deploy, and manage applications using Helm."
categories: ["Devops"]
tags: ["Automation", "Kubernetes", "DevOps", "Helm", "Package Management"]
image:
  path: /assets/dev/kube/kubernetes.webp
  lqip: data:image/webp;base64,UklGRtQBAABXRUJQVlA4WAoAAAAQAAAAEwAAEwAAQUxQSIsAAAABgFNt2/LmycSJBGYwwCACNSQT6uBkZsaik54K6OnI48/4/hIiYgKgGZzOQiBG5owgMMuonsSGFVTZbVItfuAEndwpAcD8JRC/LUBKIGeAFq0DHGln4IP2CbdgoLdqRL1vxPDBiKciQ2NK6NB6gOlIOZsAON/0vbugjP3p+U9Avcxr8VVoN141m1ACAFZQOCAiAQAA8AYAnQEqFAAUAD6RQJpKJaOiIagIALASCWwAnTKEdXem8IZsH33hQHPt6YBvDZKh/GuLZkgF/OkWZ+bNTygMAAD6MfA3/NCO7/ftr7xz71nWzdAYviZtYg+gfdrtg/Tp4hzzy1+pO2kCX7u4nkR6Fk3ZkqICWtz+MTCn9H1ZhMdsmn1btqfkXfNRiyYkHFGKBFBmKYzn8f/9iLH91lHvYiPWd+mLRGD1UhH/jRNx/9jB9qalU9hqU8cN4tj/hMl0TGLIrjwy3tJfq1IYi7OpURv7z9/Qi5lO+41B2kD5EDUE/+zEwCoUer+P3T1kjyZ0JR/1JGnx/zHb+qJb/2Lx9B8P+9UlMJF1/nyvJDtSIRA+78jkMzmAjIZWiCxI2KAAAAA=j
published: true
hidden: true
toc: true
---

## ⎈ Kubernetes Helm Charts

As your Kubernetes applications grow in complexity, managing numerous YAML files becomes challenging. Helm, often described as the "package manager for Kubernetes," solves this problem by providing a powerful way to define, install, and upgrade even the most complex Kubernetes applications.

### 📦 What is Helm?

Helm is a package manager for Kubernetes that helps you manage Kubernetes applications. Helm Charts are packages of pre-configured Kubernetes resources that can be easily shared and deployed.

Key Helm concepts include:

- **Chart**: A Helm package containing all resource definitions necessary to run an application in Kubernetes
- **Release**: An instance of a chart running in a Kubernetes cluster
- **Repository**: A place where charts can be stored and shared
- **Values**: Configuration that can be supplied to customize a chart during installation

### 🏗️ Helm Chart Structure

A typical Helm chart has the following structure:

```
mychart/
  ├── Chart.yaml           # Metadata about the chart
  ├── values.yaml          # Default configuration values
  ├── charts/              # Directory containing dependencies
  ├── templates/           # Directory containing templates
  │   ├── deployment.yaml  # Kubernetes resource templates
  │   ├── service.yaml
  │   ├── ingress.yaml
  │   └── _helpers.tpl     # Template helpers
  └── .helmignore          # Patterns to ignore when packaging
```

#### Chart.yaml

This file contains metadata about the chart:

```yaml
apiVersion: v2
name: mychart
version: 0.1.0
description: A Helm chart for my application
type: application
appVersion: "1.16.0"
maintainers:
  - name: Your Name
    email: your.email@example.com
```

#### values.yaml

This file contains default configuration values that can be overridden during installation:

```yaml
replicaCount: 1

image:
  repository: nginx
  tag: "1.21.6"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: false
  hosts:
    - host: chart-example.local
      paths: []

resources:
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 100m
    memory: 128Mi
```

#### Templates

Templates are Kubernetes manifest files with Go templating syntax that allow for dynamic generation of resources based on values:

```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "mychart.fullname" . }}
  labels:
    {{- include "mychart.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "mychart.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "mychart.selectorLabels" . | nindent 8 }}
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
            {{- toYaml .Values.resources | nindent 12 }}
```

### 🚀 Using Helm

#### Installing Helm

```bash
# Using Homebrew (macOS)
brew install helm

# Using Chocolatey (Windows)
choco install kubernetes-helm

# Using Snap (Linux)
sudo snap install helm --classic

# Using script
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

#### Adding Repositories

```bash
# Add the official Helm stable repository
helm repo add stable https://charts.helm.sh/stable

# Add Bitnami repository (popular for many applications)
helm repo add bitnami https://charts.bitnami.com/bitnami

# Update repositories
helm repo update
```

#### Searching for Charts

```bash
# Search for available charts
helm search repo nginx

# Get information about a specific chart
helm show chart bitnami/nginx
```

#### Installing Charts

```bash
# Install a chart with default values
helm install my-release bitnami/nginx

# Install with custom values
helm install my-release bitnami/nginx --set replicaCount=3

# Install with values from a file
helm install my-release bitnami/nginx -f custom-values.yaml

# Install in a specific namespace
helm install my-release bitnami/nginx --namespace my-namespace
```

#### Managing Releases

```bash
# List all releases
helm list

# Upgrade a release
helm upgrade my-release bitnami/nginx --set replicaCount=5

# Rollback to a previous revision
helm rollback my-release 1

# Uninstall a release
helm uninstall my-release
```

### 🛠️ Creating Your Own Helm Chart

#### Creating a Chart

```bash
# Create a new chart
helm create mychart

# Or with more modern syntax
helm create chart mychart
```

#### Customizing the Chart

Edit the generated files to customize your chart:

1. Update `Chart.yaml` with your chart's metadata
2. Modify `values.yaml` with your default configuration
3. Create or edit templates in the `templates/` directory

#### Testing Your Chart

```bash
# Validate your chart
helm lint mychart

# Render templates locally without installing
helm template mychart

# Test installation with dry-run
helm install --dry-run --debug my-release ./mychart
```

#### Packaging and Sharing

```bash
# Package your chart
helm package mychart

# Create a local chart repository
helm repo index --url https://example.com/charts .
```

### 🧩 Helm Chart Dependencies

You can define dependencies in your `Chart.yaml`:

```yaml
dependencies:
  - name: mysql
    version: 8.8.26
    repository: https://charts.bitnami.com/bitnami
    condition: mysql.enabled
  - name: redis
    version: 16.13.1
    repository: https://charts.bitnami.com/bitnami
    condition: redis.enabled
```

Manage dependencies with:

```bash
# Update dependencies
helm dependency update mychart

# Build dependencies
helm dependency build mychart
```

### 🔄 Helm Chart Hooks

Helm provides hooks to intervene at certain points in a release's lifecycle:

```yaml
# templates/pre-install-job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ include "mychart.fullname" . }}-pre-install
  annotations:
    "helm.sh/hook": pre-install
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  template:
    spec:
      containers:
        - name: pre-install-job
          image: "busybox"
          command: ["/bin/sh", "-c", "echo Pre-install job running"]
      restartPolicy: Never
```

Common hook types include:
- `pre-install`: Executes before any resources are installed
- `post-install`: Executes after all resources are installed
- `pre-delete`: Executes before any resources are deleted
- `post-delete`: Executes after all resources are deleted
- `pre-upgrade`: Executes before any resources are upgraded
- `post-upgrade`: Executes after all resources are upgraded
- `pre-rollback`: Executes before any resources are rolled back
- `post-rollback`: Executes after all resources are rolled back

### 🔍 Helm Chart Testing

Helm provides a plugin for testing charts:

```bash
# Install the helm-test plugin
helm plugin install https://github.com/helm/chart-testing

# Run tests
ct lint --all
ct install --all
```

### 🌟 Helm Best Practices

1. **Use Semantic Versioning**: Follow semantic versioning for your charts
2. **Document Values**: Add comments to your `values.yaml` file
3. **Use Helper Templates**: Create reusable templates in `_helpers.tpl`
4. **Validate Resources**: Use `helm lint` and `helm template` to validate
5. **Set Resource Limits**: Always define resource requests and limits
6. **Use Conditions**: Make components optional with conditions
7. **Test Thoroughly**: Test your charts in different environments
8. **Keep Charts Simple**: One chart should do one thing well
9. **Use Labels Consistently**: Follow Kubernetes labeling best practices
10. **Secure Your Values**: Don't commit sensitive data to your chart

### 🏆 Popular Helm Charts

Some popular Helm charts include:

- **Prometheus**: Monitoring and alerting
- **Grafana**: Visualization and dashboards
- **Elasticsearch**: Search and analytics
- **Jenkins**: Continuous integration and delivery
- **MySQL/PostgreSQL**: Databases
- **RabbitMQ/Kafka**: Message brokers
- **Nginx/Traefik**: Ingress controllers

### 🔄 Helm vs. Other Tools

| Tool | Purpose | Comparison with Helm |
|------|---------|----------------------|
| **Kustomize** | Customization of Kubernetes manifests | No templating, overlay-based approach |
| **Operators** | Application lifecycle management | More powerful for day-2 operations, but more complex |
| **Plain YAML** | Direct Kubernetes configuration | Simple but lacks templating and versioning |
| **Jsonnet** | Data templating language | More powerful but steeper learning curve |

### 🏁 Conclusion

Helm Charts provide a powerful way to package, distribute, and manage Kubernetes applications. By using Helm, you can:

- Simplify complex deployments
- Manage application configurations
- Version and share your applications
- Rollback to previous versions easily
- Manage dependencies between applications

As you continue your Kubernetes journey, mastering Helm will significantly enhance your ability to deploy and manage applications at scale.