---
author: muju
title: "Kubernetes Operators and Custom Resources"
description: "Learn about Kubernetes Operators, Custom Resources, and how they extend Kubernetes capabilities for complex applications."
categories: ["Devops"]
tags: ["Automation", "Kubernetes", "DevOps", "Operators", "CRDs"]
image:
  path: /assets/dev/kube/kubernetes.webp
  lqip: data:image/webp;base64,UklGRtQBAABXRUJQVlA4WAoAAAAQAAAAEwAAEwAAQUxQSIsAAAABgFNt2/LmycSJBGYwwCACNSQT6uBkZsaik54K6OnI48/4/hIiYgKgGZzOQiBG5owgMMuonsSGFVTZbVItfuAEndwpAcD8JRC/LUBKIGeAFq0DHGln4IP2CbdgoLdqRL1vxPDBiKciQ2NK6NB6gOlIOZsAON/0vbugjP3p+U9Avcxr8VVoN141m1ACAFZQOCAiAQAA8AYAnQEqFAAUAD6RQJpKJaOiIagIALASCWwAnTKEdXem8IZsH33hQHPt6YBvDZKh/GuLZkgF/OkWZ+bNTygMAAD6MfA3/NCO7/ftr7xz71nWzdAYviZtYg+gfdrtg/Tp4hzzy1+pO2kCX7u4nkR6Fk3ZkqICWtz+MTCn9H1ZhMdsmn1btqfkXfNRiyYkHFGKBFBmKYzn8f/9iLH91lHvYiPWd+mLRGD1UhH/jRNx/9jB9qalU9hqU8cN4tj/hMl0TGLIrjwy3tJfq1IYi7OpURv7z9/Qi5lO+41B2kD5EDUE/+zEwCoUer+P3T1kjyZ0JR/1JGnx/zHb+qJb/2Lx9B8P+9UlMJF1/nyvJDtSIRA+78jkMzmAjIZWiCxI2KAAAAA=j
published: true
hidden: true
toc: true
---

## 🤖 Kubernetes Operators and Custom Resources

As your Kubernetes journey advances, you'll encounter scenarios where standard resources like Deployments and Services aren't enough for complex applications. This is where Kubernetes Operators and Custom Resources come into play, extending Kubernetes capabilities to manage specialized workloads.

### 🧩 What are Custom Resources?

Custom Resources (CRs) extend the Kubernetes API, allowing you to define your own resource types beyond the built-in ones. They're like creating your own specialized Kubernetes objects.

#### Custom Resource Definitions (CRDs)

A Custom Resource Definition (CRD) defines a new resource type in Kubernetes. It specifies the name, schema, and validation rules for your custom resource.

Here's an example of a CRD for a simple database:

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databases.example.com
spec:
  group: example.com
  names:
    kind: Database
    plural: databases
    singular: database
    shortNames:
    - db
  scope: Namespaced
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              engine:
                type: string
                enum: ["mysql", "postgresql", "mongodb"]
              version:
                type: string
              storageSize:
                type: string
                pattern: '^\\d+Gi$'
              replicas:
                type: integer
                minimum: 1
            required: ["engine", "version", "storageSize"]
```

Once the CRD is created, you can create instances of your custom resource:

```yaml
apiVersion: example.com/v1
kind: Database
metadata:
  name: my-production-db
spec:
  engine: postgresql
  version: "13.4"
  storageSize: 10Gi
  replicas: 3
```

### 🔄 What are Kubernetes Operators?

Operators combine custom resources with custom controllers to automate complex application management tasks. They encapsulate operational knowledge and best practices for specific applications.

An Operator typically consists of:

1. **Custom Resources**: Define the desired state of your application
2. **Custom Controllers**: Watch for changes to your custom resources and take actions to reconcile the actual state with the desired state

#### How Operators Work

Operators follow the Kubernetes control loop pattern:

1. **Observe**: Monitor the state of custom resources
2. **Analyze**: Compare the observed state with the desired state
3. **Act**: Take actions to reconcile any differences
4. **Repeat**: Continue monitoring for changes

#### Benefits of Using Operators

- **Automation**: Automate complex operational tasks like backups, scaling, and upgrades
- **Encapsulation**: Package application-specific knowledge into a reusable component
- **Standardization**: Manage applications consistently using Kubernetes-native patterns
- **Reduced Operational Burden**: Minimize manual intervention for routine tasks

### 🛠️ Creating a Simple Operator

Let's walk through creating a simple operator using the Operator SDK, a framework for building Kubernetes operators.

#### 1. Install the Operator SDK

```bash
# Install the Operator SDK CLI
export ARCH=$(case $(uname -m) in x86_64) echo -n amd64 ;; aarch64) echo -n arm64 ;; *) echo -n $(uname -m) ;; esac)
export OS=$(uname | awk '{print tolower($0)}')
export OPERATOR_SDK_DL_URL=https://github.com/operator-framework/operator-sdk/releases/download/v1.28.0
curl -LO ${OPERATOR_SDK_DL_URL}/operator-sdk_${OS}_${ARCH}
chmod +x operator-sdk_${OS}_${ARCH}
sudo mv operator-sdk_${OS}_${ARCH} /usr/local/bin/operator-sdk
```

#### 2. Create a New Operator Project

```bash
# Create a new directory for your operator
mkdir myapp-operator
cd myapp-operator

# Initialize a new operator project
operator-sdk init --domain example.com --repo github.com/example/myapp-operator
```

#### 3. Create an API (Custom Resource and Controller)

```bash
# Create a new API
operator-sdk create api --group apps --version v1alpha1 --kind MyApp --resource --controller
```

#### 4. Define Your Custom Resource

Edit the generated CRD in `api/v1alpha1/myapp_types.go`:

```go
type MyAppSpec struct {
	// Size is the number of replicas
	Size int32 `json:"size"`
	// Image is the container image to use
	Image string `json:"image"`
	// ConfigMapName is the name of the ConfigMap to use
	ConfigMapName string `json:"configMapName,omitempty"`
}

type MyAppStatus struct {
	// Nodes are the names of the MyApp pods
	Nodes []string `json:"nodes"`
}
```

#### 5. Implement the Controller Logic

Edit the controller in `controllers/myapp_controller.go` to implement your reconciliation logic.

#### 6. Build and Deploy Your Operator

```bash
# Build the operator image
make docker-build docker-push IMG=<your-registry>/myapp-operator:v0.1.0

# Deploy the CRDs
make install

# Deploy the operator
make deploy IMG=<your-registry>/myapp-operator:v0.1.0
```

### 📊 Popular Kubernetes Operators

Many popular applications have operators available:

1. **Prometheus Operator**: Manages Prometheus monitoring instances
2. **Elasticsearch Operator**: Manages Elasticsearch clusters
3. **PostgreSQL Operator**: Manages PostgreSQL databases
4. **Redis Operator**: Manages Redis instances and clusters
5. **Kafka Operator**: Manages Kafka clusters

### 🔄 Operators vs. Helm Charts

Both Operators and Helm Charts help manage applications in Kubernetes, but they serve different purposes:

| Feature | Operators | Helm Charts |
|---------|-----------|-------------|
| **Purpose** | Application lifecycle management | Package management and deployment |
| **Complexity** | Higher | Lower |
| **Day 2 Operations** | Strong (backups, scaling, upgrades) | Limited |
| **Custom Logic** | Yes (custom controllers) | Limited (hooks) |
| **API Extension** | Yes (CRDs) | No |
| **Learning Curve** | Steeper | Gentler |
| **Development Effort** | Higher | Lower |
| **Best For** | Complex, stateful applications | Simpler, mostly stateless applications |

In many cases, Helm Charts and Operators can be complementary. You might use a Helm Chart to deploy an Operator, which then manages the application lifecycle.

### 🌟 Best Practices for Using Operators

1. **Use Existing Operators When Possible**: Check the [OperatorHub](https://operatorhub.io/) before building your own
2. **Follow the Operator Capability Levels**:
   - Level 1: Basic installation
   - Level 2: Seamless upgrades
   - Level 3: Full lifecycle
   - Level 4: Deep insights
   - Level 5: Auto-pilot
3. **Design for Idempotency**: Controllers should be able to run multiple times without causing issues
4. **Implement Proper Error Handling**: Handle failures gracefully and provide meaningful status information
5. **Use Status Subresources**: Update the status subresource to provide information about the application state
6. **Implement Finalizers**: Ensure proper cleanup when resources are deleted
7. **Version Your CRDs**: Plan for future changes by implementing proper versioning
8. **Document Your Custom Resources**: Provide clear documentation for users of your operator

### 🚀 Getting Started with Operators

To start using operators in your Kubernetes environment:

1. **Explore OperatorHub**: Browse available operators at [OperatorHub.io](https://operatorhub.io/)
2. **Install OLM**: The [Operator Lifecycle Manager](https://olm.operatorframework.io/) helps manage operators in your cluster
3. **Try Simple Operators First**: Start with well-established operators like Prometheus Operator
4. **Learn the Operator SDK**: If you need to build custom operators, the [Operator SDK](https://sdk.operatorframework.io/) is a great starting point

### 🏁 Conclusion

Kubernetes Operators and Custom Resources represent a powerful extension mechanism that allows Kubernetes to manage complex, stateful applications with the same declarative approach used for simpler workloads. By encapsulating operational knowledge into code, operators enable true "Day 2" operations automation and bring the benefits of Kubernetes to a wider range of applications.

Whether you're using existing operators from the community or building your own, understanding this pattern will help you manage complex applications more effectively in your Kubernetes environment.