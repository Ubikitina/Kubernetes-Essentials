
# Namespaces
Table of contents:
- [Namespaces](#namespaces)
  - [Introduction to Namespaces](#introduction-to-namespaces)
  - [Default namespaces](#default-namespaces)
  - [Namespace definition](#namespace-definition)
  - [Namespace Quotas](#namespace-quotas)
  - [Exercise 5: Namespaces](#exercise-5-namespaces)
  - [Cross Namespace communication](#cross-namespace-communication)


## Introduction to Namespaces
- Kubernetes Namespaces serve as logical isolation boundaries within a cluster.
- They facilitate safe tenant isolation by providing features such as resource quotas, network policies, and authentication mechanisms.

**Features:**
1. **Resource Quota for Scheduling:** Kubernetes enables setting resource quotas to manage resource allocation effectively.

2. **Network Isolation with Network Policies:** Network policies ensure that only allowed traffic is permitted between Pods within the namespace, enhancing network isolation.

3. **Authentication and Authorization:** Roles and Pod Security Policies are utilized for authentication and authorization purposes, ensuring secure access control.

Note: Container-level isolation still needs to be implemented to achieve stringent isolation requirements.

**Benefits of Namespaces:**
- A single Kubernetes cluster can satisfy the requirements of multiple users or groups.
- Namespaces enable different projects, teams, or customers to share resources within a Kubernetes cluster.
- Namespaces offer a unique scope for:
  - Named resources to prevent naming collisions.
  - Delegate management authority to trusted users.
  - Limit community resource consumption.

**More information about namespaces:** https://luispreciado.blog/posts/kubernetes/core-concepts/namespaces

## Default namespaces

![Default namespaces](./images/03_08%20Default%20namespaces.png)

In Kubernetes, `kube-system`, `default`, and `kube-public` are default namespaces provided by the system, each serving a specific purpose:

1. **kube-system Namespace:**
   - The `kube-system` namespace is used for Kubernetes system components and infrastructure-related resources. It contains essential system services and controllers that manage the Kubernetes cluster itself, such as the API server, scheduler, controller manager, DNS service, and networking components like the kube-proxy. 

2. **Default Namespace:**
   - The `default` namespace is the default namespace where objects are created if no namespace is specified. It's a general-purpose namespace where applications and services without specific namespace requirements are typically deployed. If a user does not explicitly specify a namespace when creating resources, Kubernetes assigns them to the `default` namespace.

3. **kube-public Namespace:**
   - The `kube-public` namespace contains public resources that are accessible to all users (including unauthenticated users). It's typically used for resources that need to be globally visible across the entire cluster, such as ConfigMaps containing cluster-wide configuration data, or to host public resources like the Kubernetes Cluster Information API, which provides metadata about the cluster.

These namespaces help organize and isolate resources within a Kubernetes cluster, providing a logical separation of concerns and allowing different teams or applications to coexist within the same cluster without interfering with each other. They also enable administrators to apply different access controls, resource quotas, and network policies to different namespaces based on the specific requirements of the applications or users running within them.

![Isolation](./images/03_09%20Isolation.png)

## Namespace definition

**Commands:**

To create a pod in a determined namespace, we can use the --namespace option.
```bash
kubectl create -f pod-definition.yaml --namespace=dev
```

**Namespace in Pod definition:**

To create the Pod in the dev environment without adding the --namespace option, we can add the `namespace` option under metadata in the `pod-definition.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  namespace: dev
  labels:
    app: myapp
    type: front-end
spec:
  containers:
  - name: nginx-container
    image: nginx
```

**Creating Namespaces: by using namespace definition file**

A way of creating namespaces is by using a namespace definition file `namespace-dev.yaml`:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: test
  labels:
    name: dev
```

Then apply the command:
```bash
kubectl create -f namespace-dev.yaml
```

**Creating Namespaces: without a YAML definition file**
```bash
kubectl create namespace dev
```

**Other interesting commands:**

This command retrieves information about pods across all namespaces in the Kubernetes cluster. Useful for getting a global view of all pods and their statuses in the cluster.

```bash
kubectl get pods --all-namespaces
```

Not to be confused with: `kubectl get pods`, this command retrieves information about pods within the namespace specified in the current context or, if no namespace is specified, within the default namespace.

The next command retrieves information about pods specifically in the "dev" namespace. It lists only the pods that belong to the "dev" namespace.

```bash
kubectl get pods --namespace=dev
```

There is a really good tool called kubens (created by Ahmet Alp Balkan) that makes it a breeze! When you run the `$ kubens` command, you should see all the
namespaces, with the active namespace highlighted.

## Namespace Quotas
To limit resources in a namespace, create a resource quota definition file and specity the namespace for which you want to create the quota. Example `compute-quota.yaml`:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev

spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```

Command to create the quota object:
```bash
kubectl create -f compute-quota.yaml
```

**Define the quotas in the Namespace vs. Pod:**

You can as well specify the minimum amount of resources required by pods in the `pod-definition.yaml`.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
    - containerPort: 8080
    resources:
      requests:
        memory: "1Gi"
        cpu: 1
      limits:
        memory: "2Gi"
        cpu: 2
```

The difference between defining quotas in the namespace versus defining them in the Pod definition is as follows:

- **Defining Quotas in Namespace:**
  - Quotas defined at the namespace level apply to all resources within that namespace, including pods, deployments, services, etc.
  - These quotas are enforced by the Kubernetes cluster, limiting the total amount of resources that can be consumed by all resources within the namespace.
  - They provide a way to manage resource allocation and prevent individual pods or deployments from consuming excessive resources, ensuring fair resource distribution within the namespace.

- **Defining Quotas in Pod Definition:**
  - Quotas defined in the pod definition apply specifically to that pod.
  - These quotas are set at the individual pod level and are used to specify the minimum and maximum amount of resources that the pod can consume.
  - They provide fine-grained control over resource allocation for each pod, allowing you to tailor resource limits based on the specific requirements of the application running in that pod.

## Exercise 5: Namespaces
In this example, we create the "test" namespace with quotas for both objects and resources. We then list pods in the system namespace and all namespaces, configure the default namespace to "test", and finally create an NGINX deployment with 3 replicas in the "test" namespace.

**1) Creating the "test" Namespace with Object and Resource Quotas**

To create the "test" namespace with quotas for objects and resources:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: test
  labels:
    name: test
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: test-quota
  namespace: test
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
    persistentvolumeclaims: "2"
    replicationcontrollers: "10"
    secrets: "5"
    services: "5"
    services.loadbalancers: "1"
```

**2) Listing Pods in System Namespace and All Namespaces**

```bash
# List pods in the system namespace
kubectl get pods --namespace=kube-system

# List pods in all namespaces
kubectl get pods --all-namespaces
```

**3) Configuring Default Namespace to "test"**

```bash
kubectl config set-context --current --namespace=test
```

**4) Creating NGINX Deployment in the "test" Namespace**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: test
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx-container
        image: nginx:1.14.2
        resources:
          requests:
            cpu: 200m
            memory: 0.5Gi
          limits:
            cpu: 200m
            memory: 0.5Gi
```

## Cross Namespace communication

In Kubernetes, namespaces serve as isolated environments for organizing and managing resources. While namespaces are isolated from each other by default, Kubernetes allows services in one namespace to communicate with services in another namespace.

When your application needs to access a Kubernetes service, you can leverage the built-in DNS service discovery mechanism. By simply pointing your application to the service's name, Kubernetes automatically resolves the DNS to the appropriate service endpoint.

However, it's essential to note that you can create services with the same name in multiple namespaces. To handle this scenario, you can use the expanded form of the DNS address. Kubernetes services expose their endpoints using a common DNS pattern:

```
<Service Name>.<Namespace Name>.svc.cluster.local
```

By incorporating the namespace name into the DNS address, Kubernetes ensures unique resolution of services across namespaces, preventing conflicts and enabling seamless communication between services regardless of the namespace they belong to.

**More information:** https://luispreciado.blog/posts/kubernetes/core-concepts/namespaces

**Service Discovery: Simplifying Communication Between Services**

Overview: In Kubernetes, service discovery refers to the process by which services within a cluster can locate and communicate with each other seamlessly. This solves the problem of how one service can find and interact with another service running within the same cluster.

**Components of Service Discovery:**
1. **Service Registry/Naming:** Services are registered with a naming system that allows them to be easily identified within the cluster.
   
2. **Service Announcement:** Services announce their availability to the cluster, making their endpoints known to other services.

3. **Lookup/Discovery:** Services can perform lookups to discover the endpoints of other services they need to communicate with.

4. **Load Balancing:** Load balancing mechanisms ensure that requests are evenly distributed among instances of a service for optimal performance.

![Service Discovery](./images/03_10%20Service%20Discovery.png)

**Service Discovery - DNS:**
- In Kubernetes, all services have a DNS name.
- The DNS naming convention is structured as follows:
  `<my-Service-name>.<my-namespace>.svc.cluster.local`
- This naming convention simplifies the identification of services required by your application.

![Service Discovery](./images/03_11%20DNS.png)
  
By leveraging DNS-based service discovery, Kubernetes provides a straightforward and reliable mechanism for services to locate and communicate with each other within the cluster. This ensures efficient communication and seamless interaction between microservices, promoting the scalability and reliability of distributed applications.

![Core DNS](./images/03_12%20Core%20DNS.png)
