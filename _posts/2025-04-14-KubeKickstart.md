---
author: muju
title: "☸️ kubernetes kickstart"
description: "Get hands-on with Kubernetes ConfigMaps, Secrets, and environment variables using practical YAML examples."
categories: ["Linux"]
tags: ["Automation", "Linux", "DevOps"]
image:
  path: /assets/dev/kube/kubernetes.webp
  lqip: data:image/webp;base64,UklGRtQBAABXRUJQVlA4WAoAAAAQAAAAEwAAEwAAQUxQSIsAAAABgFNt2/LmycSJBGYwwCACNSQT6uBkZsaik54K6OnI48/4/hIiYgKgGZzOQiBG5owgMMuonsSGFVTZbVItfuAEndwpAcD8JRC/LUBKIGeAFq0DHGln4IP2CbdgoLdqRL1vxPDBiKciQ2NK6NB6gOlIOZsAON/0vbugjP3p+U9Avcxr8VVoN141m1ACAFZQOCAiAQAA8AYAnQEqFAAUAD6RQJpKJaOiIagIALASCWwAnTKEdXem8IZsH33hQHPt6YBvDZKh/GuLZkgF/OkWZ+bNTygMAAD6MfA3/NCO7/ftr7xz71nWzdAYviZtYg+gfdrtg/Tp4hzzy1+pO2kCX7u4nkR6Fk3ZkqICWtz+MTCn9H1ZhMdsmn1btqfkXfNRiyYkHFGKBFBmKYzn8f/9iLH91lHvYiPWd+mLRGD1UhH/jRNx/9jB9qalU9hqU8cN4tj/hMl0TGLIrjwy3tJfq1IYi7OpURv7z9/Qi5lO+41B2kD5EDUE/+zEwCoUer+P3T1kjyZ0JR/1JGnx/zHb+qJb/2Lx9B8P+9UlMJF1/nyvJDtSIRA+78jkMzmAjIZWiCxI2KAAAAA=j
published: false
hidden: false
toc: true
# https://medium.com/@venkataramarao.n/kubernetes-setup-using-ansible-script-8dd6607745f6
# https://medium.com/@sirtcp/deploying-a-three-node-kubernetes-cluster-on-debian-12-with-kubeadm-and-rook-ceph-for-persistent-eb080f31d3fc
# https://medium.com/@subhampradhan966/how-to-install-kubernetes-cluster-kubeadm-setup-on-ubuntu-22-04-step-by-step-guide-dfcf33253f5f ==> kube on ubuntu 20.04 vps
# https://github.com/CloudHub-Social/CloudHub-Cluster?tab=readme-ov-file ==> good modules are present used as source
# https://github.com/manupanand-freelance-developer/kubernetes-cluster-infra-aws/blob/main/k8s-infra-selfmanaged/ansible/roles/common/tasks/common.yml ==> ansible code exist for refference
# https://github.com/HouseOfLogicGH/KubernetesOnProxmoxWithTerraformAndAnsible
# lxc  is not recomended as it cause issues https://forum.proxmox.com/threads/help-me-install-k8s-into-lxc-container.149140/ so need wto switch the setup to vm

# https://www.learnlinux.tv/how-to-build-an-awesome-kubernetes-cluster-using-proxmox-virtual-environment/
# br_netfilter and overlay

# https://manjit28.medium.com/automating-vm-creation-in-proxmox-a-complete-guide-18652f346db5 ==> auto vm creation with bsh scripts
# https://pswalia2u.medium.com/deploying-kubernetes-cluster-2ef2fbdd233a
---

Setting up a Kubernetes cluster is just the beginning securing it with fine grained access control is what takes your setup from basic to production ready. In this guide, we’ll explore Kubernetes RBAC (Role Based Access Control) from the ground up, including the creation of users, service accounts, roles, and role bindings.

> 🛠️ New to Kubernetes?  If you haven’t set up your `Kubernetes cluster` yet or are unsure about your current setup, I highly recommend checking out my previous [blog post]({% post_url 2025-04-04-kubernetes %}){:target="_blank"} where I walk you through the complete cluster configuration from installing essential tools to running your first kubectl command. This blog builds directly on that foundation.
{: .prompt-info }

## 🛡️ Role Base Access RBAC-K8S

RBAC stands for Role Based Access Control. It lets you define who (user or service account) can do what (verbs like get, create, delete) on which resources (like pods, deployments, configmaps) and where (namespace or cluster wide).

There are four core RBAC objects:
- **Role:** Defines permissions within a namespace.
- **ClusterRole:** Like a Role, but cluster wide.
- **RoleBinding:** Grants Role permissions to a user or service account in a namespace.
- **ClusterRoleBinding:** Grants ClusterRole permissions across namespaces.

Let’s break it down with a real world example.

- 🧱 1. Create a Namespace
```bash
kubectl create namespace development
```

- 🔐 2. Private Key and CSR for User
```bash
cd ~/.kube
openssl genrsa -out DevUser.key 2048
openssl req -new -key DevUser.key -out DevUser.csr -subj "/CN=DevUser/O=development"
```
The `CN` (Common Name) will be the Kubernetes username. The `O` (Organization) maps to group membership.

- 📜 3. Generate a Signed Certificate
Use the Kubernetes cluster’s CA to sign the request:
```bash
# minikube
openssl x509 -req -in DevUser.csr -CA ${HOME}/.minikube/ca.crt -CAkey ${HOME}/.minikube/ca.key -CAcreateserial -out DevUser.crt -days 45
# HA_k8S_Cluster
sudo openssl x509 -req -in DevUser.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out DevUser.crt -days 45
```

- ⚙️ 4. Add the User to Kubeconfig
```bash
kubectl config set-credentials DevUser --client-certificate=${HOME}/.kube/DevUser.crt   --client-key=${HOME}/.kube/DevUser.key
```

- 🌐 5. Set the Context for the User
```bash
kubectl config set-context DevUser-context   --cluster=\$CLUSTER   --namespace=development   --user=DevUser
```

### 🔑 Creating Roles and RoleBindings

Create the `pod-reader-role.yaml` file and apply the configuration

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: development
rules:
  - apiGroups: [""]
    resources: ["pods", "pods/log"]
    verbs: ["get", "watch", "list", "update"]
```

```bash
kubectl apply -f pod-reader-role.yaml
```

### 🔗 RoleBinding YAML 

Create the `pod-reader-rolebinding.yaml` file and apply the configuration

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader
  namespace: development
subjects:
- kind: User
  name: DevUser
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

```bash
kubectl apply -f pod-reader-rolebinding.yaml
```

### ✅ Test Permissions

Switch to the context and try listing pods:

```bash
kubectl get pods --context=DevUser-context
```

Create a pod to test it further:

```bash
kubectl run nginx --image=nginx --context=DevUser-context
```
![kubernetes](/assets/dev/kube/kickstart/ks2.png){: width="800" height="500" }
![kubernetes](/assets/dev/kube/kickstart/ks1.png){: width="800" height="500" }

## 🤖 RBAC-K8S with ServiceAccounts

You can also bind roles to ServiceAccounts, which are typically used by applications running inside the cluster.

- 🔧 Create ServiceAccount
    ```yaml
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: my-serviceaccount
      namespace: development
    automountServiceAccountToken: false
    ```
    ```bash
    kubectl apply -f serviceaccount.yaml
    ```

- 🔗 RoleBinding for ServiceAccount

    ```yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
      name: sa-pod-reader
      namespace: development
    subjects:
    - kind: ServiceAccount
      name: my-serviceaccount
      namespace: development
    roleRef:
      kind: Role
      name: pod-reader
      apiGroup: rbac.authorization.k8s.io
    ```
    ```bash
    kubectl apply -f serviceaccount-rolebinding.yaml
    ```
![kubernetes](/assets/dev/kube/kickstart/ks3.png){: width="800" height="500" }

## 🧼 Clean Up (Optional)
```bash
kubectl delete ns development
rm ${HOME}/.kube/DevUser.*
```

## 🔧 K8s: Vars, ConfigMaps & Secrets

When working with real world applications in Kubernetes, environment variables, configuration files, and secrets are essential. They help you manage dynamic configuration, separate sensitive data, and keep your pods flexible.

we’ll explore:

- ✅ Setting environment variables from ConfigMaps and Secrets
- 📦 Using `envFrom` to bulk import environment variables
- 📁 Mounting ConfigMaps and Secrets as volumes

Let’s kickstart with practical YAML examples 🚀

- 🔧 What Are ConfigMaps & Secrets?

    - **ConfigMap**: Stores configuration data as key-value pairs. Ideal for non-sensitive information like file names, port numbers, or settings.
    - **Secret**: Used to store sensitive data like credentials and API tokens. Kubernetes encodes this data in base64.

- 🧪 Creating a ConfigMap

Here’s a sample ConfigMap with both simple key-values and file-style data:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-config
data:
  player_lives: "3"
  properties_file_name: "ui-settings.properties"
  base.properties: |
    enemy.types=ghosts,zombies
    player.maximum-lives=7
  ui-settings.properties: |
    theme=dark
    sounds.enabled=true
```
> Tip: You can apply this ConfigMap using kubectl apply -f $FILE
{: .prompt-info }

- 🔐  Create a Secret

Now let’s securely store sensitive data like username and password:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: game-secret
type: Opaque
stringData:
  username: adminYWRtaW4=
  password: c3VwZXJzZWNyZXQxMjM=
```

>  Why use Secrets? Unlike ConfigMaps, Secrets are base64-encoded and can be restricted via RBAC
{: .prompt-info }

#### ⚙️ Method 1: Inject ConfigMap & Secret as Environment Variables

Here's how you can load individual keys from a ConfigMap and Secret into environment variables in your Pod `env` allows you to set environment variables for a container, specifying a value directly for each variable that you name.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-env-demo
spec:
  containers:
    - name: app-container
      image: alpine
      command: ["sleep", "3600"]
      env:
        - name: PLAYER_LIVES
          valueFrom:
            configMapKeyRef:
              name: game-config
              key: player_lives
        - name: CONFIG_FILE
          valueFrom:
            configMapKeyRef:
              name: game-config
              key: properties_file_name
        - name: USERNAME
          valueFrom:
            secretKeyRef:
              name: game-secret
              key: username
        - name: PASSWORD
          valueFrom:
            secretKeyRef:
              name: game-secret
              key: password
```

![kubernetes](/assets/dev/kube/kickstart/ks4.png){: width="800" height="500" }

> 🛠️ Use case: When you only need specific variables from a ConfigMap/Secret.
{: .prompt-info }

#### 🌱 Method 2: Load ConfigMap via envFrom

To inject all values from a ConfigMap as environment variables, use `envFrom:` allows you to set environment variables for a container by referencing either a ConfigMap or a Secret. When you use `envFrom`, all the key-value pairs in the referenced ConfigMap or Secret are set as environment variables for the container. You can also specify a common prefix string.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-envfrom-demo
spec:
  containers:
    - name: webapp
      image: nginx
      envFrom:
        - configMapRef:
            name: game-config-lite
```

Here’s the `game-config-lite` ConfigMap:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-config-lite
data:
  PLAYER_LIVES: "3"
  CONFIG_MODE: "light"
```

![kubernetes](/assets/dev/kube/kickstart/ks5.png){: width="800" height="500" }

> All keys in the ConfigMap become environment variables inside the container.
{: .prompt-info }

#### 📁 Method 3: Mount ConfigMap & Secret as Volumes

For scenarios where your app expects config files, mount them as volumes:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-volume-demo
spec:
  containers:
    - name: alpine-container
      image: alpine
      command: ["sleep", "3600"]
      volumeMounts:
        - name: config-vol
          mountPath: /app/config
        - name: secret-vol
          mountPath: /app/secret
  volumes:
    - name: config-vol
      configMap:
        name: game-config
    - name: secret-vol
      secret:
        secretName: game-secret
```

![kubernetes](/assets/dev/kube/kickstart/ks6.png){: width="800" height="500" }

📂 This creates files in the container at /app/config and /app/secret.

#### 🔍 Verification

After applying the YAMLs:

- Use `kubectl exec -it $pod-name -- /bin/sh` to enter the container
- Run `env` or `printenv` to see loaded environment variables
- Use `cat /app/config/*` to read mounted files

#### 🔐 NGINX Auth with ConfigMap & Secret
To create the above setup in Kubernetes, we need to define:
- A **ConfigMap** to hold the `nginx.conf` file.
- A **Secret** to hold the `.htpasswd` credentials.
- A **Pod** definition to mount both the `ConfigMap` and `Secret` and run the NGINX container.


1. ✅ Step: nginx.conf file (save as nginx default.conf)

    ```bash
    server {
        listen       80;
        listen  [::]:80;
        server_name  localhost;

        #access_log  /var/log/nginx/host.access.log  main;

        location / {
        
            auth_basic "Restricted";
            auth_basic_user_file /etc/nginx/config/basicauth;

            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/html;
        }
    }
    ```

2. ✅ Step: Create ConfigMap from `nginx.conf`
    ```bash
    kubectl create configmap nginx-config-file --from-file default.conf
    kubectl get configmap && kubectl describe configmap nginx-config-file
    ```

3. ✅ Step 3: Create Secret for `.htpasswd`

    Generate `.htpasswd` using Apache utils:
    
    ```bash
    htpasswd -bc basicauth admin MyStrongPassword
    ```
    
    Then create the secret:
    
    ```bash
    kubectl create secret generic nginx-htpasswd --from-file basicauth
    kubectl get secret && kubectl describe secret nginx-htpasswd
    ```

4. ✅ Step: Pod Manifest (nginx-pod.yaml)

    ```bash
    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx-pod
    spec:
      containers:
        - name: nginx-container
          image: nginx:1.19.1
          ports:
            - containerPort: 80
          volumeMounts:
            - name: nginx-config-volume
              mountPath: /etc/nginx/conf.d/default.conf
              subPath: default.conf
            - name: htpasswd-volume
              mountPath: /etc/nginx/config

      volumes:
        - name: nginx-config-volume
          configMap:
            name: nginx-config-file
        - name: htpasswd-volume
          secret:
            secretName: nginx-htpasswd
    ```

5. ✅ Step: Deploy the Pod

    ```bash
    kubectl apply -f nginx-pod.yaml
    ```

    Once it's up, you can port-forward to access it:

    ```bash
    kubectl port-forward pod/nginx-pod 8080:80
    ```
    Then navigate to http://localhost:8080 and you'll be prompted for authentication. Use the username and password you added to .htpasswd.

    ![kubernetes](/assets/dev/kube/kickstart/ks7.png){: width="800" height="500" }

## 🐳 K8S Container Resources
In Kubernetes, Container Resources play a vital role in efficiently managing workloads across nodes in a cluster. By defining how much CPU and memory a container needs, you can ensure optimal resource utilization, avoid overloading nodes, and maintain application stability.


### 🎯 Resource Request

A Resource Request is the minimum amount of CPU or memory that a container expects to use. It doesn’t restrict the container’s usage but acts as a scheduling guide for the Kubernetes scheduler.

- Helps the Kube Scheduler decide where to run a pod.
- Prevents pods from being scheduled on nodes that don’t have sufficient resources.
- This doesn’t cap resource usage. A container can consume more than requested if available.
  - 🧠 Memory is measured in bytes (e.g., 64Mi for 64 Megabytes).
  - 🧮 CPU is measured in millicores (250m = 0.25 vCPU).

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"

```

### 🚦 Resource Limit

A Resource Limit defines the maximum amount of CPU or memory a container is allowed to consume. These limits are enforced at runtime to prevent any single container from hogging system resources.

- 🔐 Purpose: Protects the node from resource exhaustion due to misbehaving containers.
- 📏 Imposes a hard cap on how much CPU/memory the container can use.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: app
    image: nginx
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
```

## 🩺 Monitoring Containers in K8S

Kubernetes is a feature-rich platform that goes beyond deployment it also offers robust container monitoring tools to ensure high availability, automatic recovery, and smooth operations.

### ❤️ Container Health

Kubernetes actively monitors containers to determine their state and perform automatic actions, such as restarting a failed container. This keeps your app highly available and fault-tolerant.

- 📡 Monitoring allows Kubernetes to detect crashes, hangs, or readiness delays.
- 🔄 Kubernetes can automatically restart unhealthy containers.

### 🧪 Liveness Probe
A Liveness Probe checks if your container is alive and functioning properly. Without it, Kubernetes assumes a container is fine as long as the process is running even if it’s stuck.

Liveness probes can be configured in two ways:

- ✅ Exec: Run a command inside the container.
- 🌐 HTTP: Perform periodic health checks via HTTP.

Example: Exec Probe
```yaml
livenessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 5
```

Example: HTTP Probe
```yaml
livenessProbe:
  httpGet:
    path: /health.html
    port: 8080
    httpHeaders:
    - name: Custom-Header
      value: Awesome
  initialDelaySeconds: 3
  periodSeconds: 3
  timeoutSeconds: 1
```

### 🚀 Startup Probe

Some applications take longer to start. That’s where Startup Probes come in. They delay the execution of liveness probes until the application is fully ready to be monitored.

- 🕒 Ensures long-startup apps don’t get killed prematurely.
- ✅ Runs only once at startup; once it succeeds, liveness kicks in.

📄 Example: Startup Probe
```yaml
startupProbe:
  httpGet:
    path: /health.html
    port: 8080
  failureThreshold: 30
  periodSeconds: 10
```

> ⏱️ This config allows up to 5 minutes (30 * 10s) for startup.
{: .prompt-info }

### ⚙️ Readiness Probe

A Readiness Probe checks if the container is ready to handle requests. Until this probe passes, no traffic is routed to the pod.

- Useful when the app depends on external services or needs time to load configs/data.
- Can run in parallel with Liveness Probe.

📄 Example: Readiness Probe
```yaml
readinessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 5
```


### 🧪 Liveness Probe using Exec

This pod creates a file `/tmp/healthcheck`, deletes it after 60 seconds, and the probe checks for its existence.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-probe-exec-demo
spec:
  containers:
    - name: busybox-checker
      image: k8s.gcr.io/busybox
      args:
        - /bin/sh
        - -c
        - touch /tmp/healthcheck; sleep 60; rm -rf /tmp/healthcheck; sleep 600
      livenessProbe:
        exec:
          command:
            - stat
            - /tmp/healthcheck
        initialDelaySeconds: 5
        periodSeconds: 5
```

📝 Explanation:

- touch creates the file.
- After 60 seconds, the file is removed — causing the probe to fail.
- Kubernetes will restart the container after the probe fails.

### 🌐 Liveness Probe using HTTP

This example sends HTTP GET requests to the root path of an NGINX container.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-probe-http-demo
spec:
  containers:
    - name: nginx-liveness
      image: k8s.gcr.io/nginx
      livenessProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 3
        periodSeconds: 3
```

📝 Explanation:

- It checks if the web server responds on /.
- If NGINX fails to respond, the container will be restarted.

### 🚀 Startup Probe using HTTP

Use this probe when your container takes a long time to start.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: startup-probe-http-demo
spec:
  containers:
    - name: nginx-startup
      image: k8s.gcr.io/nginx
      startupProbe:
        httpGet:
          path: /
          port: 80
        failureThreshold: 30
        periodSeconds: 10
```
📝 Explanation:

- Gives up to 300 seconds (30 * 10) for the container to become healthy.
- If it doesn’t respond within this time, Kubernetes marks it as failed.

### ⚙️ Readiness Probe using exec

This pod simulates an app that becomes ready after 10 seconds by creating a file `/tmp/ready`. The readiness probe checks for that file every 5 seconds.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: readiness-probe
spec:
  containers:
    - name: readiness-container
      image: busybox
      args:
        - /bin/sh
        - -c
        - "sleep 10; touch /tmp/ready; sleep 300"
      readinessProbe:
        exec:
          command:
            - cat
            - /tmp/ready
        initialDelaySeconds: 5
        periodSeconds: 5
```
📝 Explanation:

- The container sleeps 10 seconds and then creates the `/tmp/ready` file.
- The `readinessProbe` keeps checking every 5 seconds if the file exists using `cat /tmp/ready`.
- Only after the file is created, the pod becomes Ready, and starts receiving traffic.

## 🚑 Restart Policies in K8S

Kubernetes is designed to provide high availability and resilience. One of its powerful features is self-healing ability to automatically restart containers when they fail. This behavior is controlled through container restart policies.

Let’s dive into the different Restart Policies available in Kubernetes and how they contribute to creating self-healing workloads.

🔁 Container Restart Policies

Kubernetes provides built-in restart policies that define how the kubelet should handle container restarts within a Pod. These policies allow you to define when and how a container should be restarted in case of failure or success.

Kubernetes supports three restart policies:
- Always
- OnFailure
- Never

> 🔹 Note: Restart policies apply only to Pods created directly (not managed by controllers like Deployments). Deployments always use restartPolicy: `Always.`
{: .prompt-info }

### ✅ Always Restart Policy

- This is the default restart policy in Kubernetes.
- Containers are restarted regardless of the exit status—even if the container exits successfully.
- Ideal for long-running services that should always remain running (e.g., web servers, APIs).

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-restart-always
spec:
  restartPolicy: Always
  containers:
    - name: always-container
      image: alpine
      command: ["sleep", "10"]
```

### ⚠️ OnFailure Restart Policy

- The container is restarted only if it exits with a non-zero status code (i.e., on failure).
- It also triggers restart if the liveness probe fails.
- Best for batch jobs or scripts that should run again only on failure.

```ymal
apiVersion: v1
kind: Pod
metadata:
  name: pod-restart-onfailure
spec:
  restartPolicy: OnFailure
  containers:
    - name: onfailure-container
      image: alpine
      command: ["sh", "-c", "exit 1"]
```

### ⛔ Never Restart Policy

- The container will never be restarted, regardless of success or failure.
- Suitable for one-time jobs or scripts that should not be retried automatically.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-restart-never
spec:
  restartPolicy: Never
  containers:
    - name: never-container
      image: alpine
      command: ["sh", "-c", "exit 1"]
```

## 🧩 Multi-Container & Init in K8S

In Kubernetes, a Pod is the smallest deployable unit that can contain one or more containers. While most Pods typically run a single container, there are use cases where multiple containers working closely together sharing resources like storage, networking and before these containers run, you might need to set up the environment or check dependencies — that’s where Init Containers shine.

Multi-Container Pods Kubernetes allows Pods to run multiple containers that:
- Share the same network namespace (localhost)
- Can share storage volumes
- Are scheduled and managed as a single unit

Init Containers are special containers in a Pod that:
- Run before the application containers start.
- Execute only once during Pod startup.
- Are executed sequentially, one after the other.
- Must complete successfully before any app container starts.

They are ideal for tasks like waiting for a service, performing setup logic, or preparing configuration/data before launching the main container.

🔁 Init Container Flow

```css
[ Init Container 1 ] ---> [ Init Container 2 ] ---> [ App Container(s) ]
```

- All Init Containers must run to completion before the main container(s) start.
- You can define multiple Init Containers, and they will execute in order.


```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
---
apiVersion: v1
kind: Service
metadata:
  name: redis
spec:
  ports:
    - protocol: TCP
      port: 6379
      targetPort: 6379
---
apiVersion: v1
kind: Pod
metadata:
  name: webapp-pod
spec:
  volumes:
    - name: shared-logs
      emptyDir: {}

  initContainers:
    - name: init-db
      image: busybox:1.28
      command:
        - sh
        - -c
        - |
          until nslookup mysql.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local;
          do echo "⏳ Waiting for MySQL..."; sleep 5; done
    - name: init-redis
      image: busybox:1.28
      command:
        - sh
        - -c
        - |
          until nslookup redis.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local;
          do echo "⏳ Waiting for Redis..."; sleep 5; done

  containers:
    - name: web-app
      image: busybox:1.28
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/app
      command:
        - sh
        - -c
        - |
          echo "🚀 Starting Web App" > /var/log/app/app.log;
          sleep 3600

    - name: log-collector
      image: busybox:1.28
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/app
      command:
        - sh
        - -c
        - tail -f /var/log/app/app.log
```

This setup uses init containers (`init-db` and `init-redis`) to ensure that MySQL and Redis services are reachable before starting the main application. These init containers simulate readiness checks, blocking the app from starting until all dependencies are up. Once they complete, the main containers start. 

The `web-app` container writes logs to a shared path (`/var/log/app/app.log`), and the `log-collector` container continuously reads (tails) these logs using a shared volume mounted between them. 

This demonstrates how containers within the same Pod can communicate and share data. To test this setup, you can deploy fake or real MySQL and Redis services, apply the manifest using `kubectl apply -f file.yaml`, and then run `kubectl logs webapp-pod -c log-collector` to see the log output.