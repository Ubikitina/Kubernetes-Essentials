# Services and Networking
Table of contents:
- [Services and Networking](#services-and-networking)
  - [Services](#services)
    - [What are Services?](#what-are-services)
    - [iptables kube-proxy](#iptables-kube-proxy)
    - [Service Types](#service-types)
    - [Kubernetes manifest: Service YAML](#kubernetes-manifest-service-yaml)
      - [ClusterIP](#clusterip)
      - [NodePort](#nodeport)
    - [Update by using a Service](#update-by-using-a-service)
    - [Exercise 10: Services in K8S](#exercise-10-services-in-k8s)
  - [Networking](#networking)
    - [Networking in Kubernetes](#networking-in-kubernetes)
    - [Network Policies](#network-policies)
      - [Solutions Supporting Network Policies:](#solutions-supporting-network-policies)
    - [Kubernetes Manifest: Network Policies YAML](#kubernetes-manifest-network-policies-yaml)
  - [Ingress Controller](#ingress-controller)
    - [Ingresses](#ingresses)
    - [Ingress Controller](#ingress-controller-1)
    - [Exercise 11: Ingress Controller](#exercise-11-ingress-controller)


## Services
### What are Services?
A service defines a logical set of pods (your microservice). They are essentially a virtual **load balancer** in front of pods.

**Load balancing:** is the process of distributing workloads across multiple servers, collectively known as a server cluster. The main purpose of load balancing is to prevent any single server from getting overloaded and possibly breaking down. In other words, load balancing improves service availability and helps prevent downtimes.

**Services features:**
- A group of pods that work together, grouped by a selector.
- Defines access policy: "Load balanced" or "headless".
- Can have a stable virtual IP and Port. Also a DNS name.
- VIP (virtual IP) is managed by kube-proxy.
  - Watches all services.
  - Updates iptables when backends change.
  - Default implementation - can be replaced.
- Hides complexity.

![Services](./images/09_01%20Services.png)

### iptables kube-proxy
`kube-proxy` plays a crucial role in facilitating network communication within the Kubernetes cluster by managing `iptables` rules and ensuring proper routing of traffic to the appropriate pods.

1. **API Server Interaction**: When the Kubernetes API server needs to communicate with a specific node (let's call it Node X), it does so through the kube-proxy service running on that node. It accomplishes this by utilizing `watch` services and endpoints. These watches monitor changes to services and endpoints in the cluster.

2. **User Action - `kubectl run`**: A user initiates an action by executing a command like `kubectl run`. This command might create a new deployment or a pod.

3. **Scheduling by API Server**: The API server is responsible for scheduling the newly created resources, such as pods, to appropriate nodes within the cluster.

4. **User Action - `kubectl expose`**: Subsequently, the user might decide to expose the newly created deployment or pods to the outside world using `kubectl expose`. This command creates a service, exposing the deployment or pods.

5. **Service Detection by kube-proxy**: The kube-proxy service constantly watches for new services being created within the cluster.

6. **Configuration of `iptables`**: Upon detecting a new service, kube-proxy configures `iptables` rules on the node. These rules typically involve setting up port forwarding or load balancing to redirect incoming traffic to the appropriate pods.

7. **Endpoint Detection by kube-proxy**: Similarly, kube-proxy also monitors for changes in endpoints associated with services.

8. **Additional `iptables` Configuration**: Upon detecting new endpoints, kube-proxy updates the `iptables` configuration accordingly. This ensures that traffic is correctly routed to the pods backing the service.

9. **Assignment of Virtual IPs**: As part of this process, kube-proxy assigns virtual IP addresses to the pods. These virtual IPs are used for routing traffic within the cluster.

10. **Client Interaction**: With the setup complete, external clients or services can now contact the virtual IPs assigned to the pods. Traffic directed to these virtual IPs will be appropriately forwarded to the corresponding pods by the `iptables` rules configured by kube-proxy.


![iptables kube-proxy](./images/09_02%20iptables%20kube-proxy.png)

### Service Types
- **ClusterIP:** Exposes the service on a cluster-internal IP (only visible for Kubernetes). Choosing this value makes the service only reachable from within the cluster.
- **NodePort:** Exposes the service on each Node’s IP at a static port (the NodePort). Connect from outside the cluster by requesting `<NodeIP>:<NodePort>`.
  - In addition to the internal IP (red dot), the machines are exposed through a port.
  - Each port corresponds to a specific service (to a Node).
  - Disadvantage: clients need to know which one to use, and there might be overload if many people try to contact the same port.
- **LoadBalancer:** Exposes the service externally using a cloud provider’s load balancer.

![Service Types](./images/09_03%20Service%20Types.png)

![Service with](./images/09_04%20Service%20with%20NodePort.png)

### Kubernetes manifest: Service YAML

#### ClusterIP
This YAML manifest defines a Kubernetes Service named "my-first-service" that exposes port 80 internally and forwards traffic to port 9376 on pods labeled with the "app: web" label. The service type is ClusterIP, which means it's only accessible within the Kubernetes cluster.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-first-service  # Name of the service
spec:
  selector:
    app: web  # Selector to match pods labeled with 'app: web'
  type: ClusterIP  # Type of service (ClusterIP, NodePort, LoadBalancer, or ExternalName)
  ports:
    - protocol: TCP  # Protocol used (TCP or UDP)
      port: 80  # Port exposed on the service
      targetPort: 9376  # Port on the pods that the service forwards traffic to
```

#### NodePort
This YAML manifest defines a Kubernetes Service named "my-first-service" that exposes port 80 externally on all cluster nodes, forwarding traffic to port 9376 on pods labeled with the "app: web" label. The service type is NodePort, which means it's accessible externally on each node's IP address at the specified nodePort (in this case, 30008).

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-first-service  # Name of the service
spec:
  selector:
    app: web  # Selector to match pods labeled with 'app: web'
  type: NodePort  # Type of service (ClusterIP, NodePort, LoadBalancer, or ExternalName)
  ports:
    - protocol: TCP  # Protocol used (TCP or UDP)
      port: 80  # Port exposed on the service
      targetPort: 9376  # Port on the pods that the service forwards traffic to
      nodePort: 30008  # Port accessible on all cluster nodes
```

### Update by using a Service
Starting point:
![Starting point](./images/09_05%20Starting%20point.png)

Canary:
![Canary](./images/09_06%20Canrary.png)

Replicaset:
![Replicaset Step 1](./images/09_07%20ReplicaSet%20Step1.png) 
![Replicaset Step 2](./images/09_08%20ReplicaSet%20Step2.png)
![Replicaset Step 3](./images/09_09%20ReplicaSet%20Step3.png)

### Exercise 10: Services in K8S
- Deploy an application using a deployment, for example, we use the image xstabel/dotnet-core-publish. Wait for the POD to be in the "Ready" state.
- Expose the Service
- Access the service... we perform a Curl to the endpoint.

Commands executed:

1. `kubectl expose deployment front3 --type=NodePort --port=8080`:
   - This command exposes the deployment named "front3" as a service.
   - `--type=NodePort` specifies the type of the service, which in this case is NodePort. It exposes the service on a port on each node in the cluster.
   - `--port=8080` specifies the port number on which the service will listen.

2. `kubectl get all`:
   - This command retrieves information about all resources in the current Kubernetes namespace.
   - It includes information about pods, services, deployments, and any other resources present.

3. `minikube service front3`:
   - This command opens the exposed service named "front3" in the default browser.
   - When executed, it launches a web browser and directs it to the URL of the service.

4. `minikube service front3 --url`:
   - This command prints the URL of the exposed service named "front3".
   - Instead of opening the service in a browser, it only displays the URL, allowing you to use it programmatically or in other scripts.

5. `kubectl port-forward deployment/my-deployment 8080:8080`:
   - This command sets up a proxy that forwards traffic from a local port (8080) to a port on a specific pod (also 8080) belonging to the deployment named "my-deployment".
   - It allows you to access a specific pod's service locally on your machine without exposing it to the broader network.

![Exercise 10](./images/09_10%20Exercise%2010.png)


## Networking
### Networking in Kubernetes
Kubernetes Networking model introduces 3 methods of communications:
- **Pod-to-Pod** communication directly by IP address. Kubernetes has a Pod-IP wide metric simplifying communication.
- **Pod-to-Service** Communication – Client traffic is directed to service virtual IP by iptables rules that are modified by the kub-proxy process (running on all
hosts) and directed to the correct Pod.
- **External-to-Internal** Communication – external access is captured by an external load balancer which targets nodes in a cluster. The Pod-to-Service flow stays the same.

![Networking](./images/09_11%20Networking.png)

![Networking](./images/09_12%20Networking.png)

### Network Policies

Kubernetes Network Policies are a native Kubernetes feature that allows you to control the traffic flow between pods within your cluster. With Network Policies, you can define rules that specify how pods are allowed to communicate with each other and with external services. These rules can permit or deny traffic based on various criteria such as source IP address, destination IP address, and port numbers.

**Key points**:
- **Ingress rules**: Define how incoming traffic is handled.
- **Egress rules**: Define how outgoing traffic is handled.
- **Allow rules**: Permit specified traffic to pass through.
- **Deny rules**: Block specified traffic from passing through.

#### Solutions Supporting Network Policies:
Several networking solutions integrate with Kubernetes to enforce Network Policies. Some popular ones include:

1. **Kube-router**:
   - Kube-router is a highly integrated solution that offers network routing, firewalling, and network policy enforcement for Kubernetes clusters.
   - It provides support for Kubernetes Network Policies, allowing you to define and enforce fine-grained network access controls.

2. **Calico**:
   - Calico is a popular open-source networking and network security solution for containers and virtual machines.
   - It supports Kubernetes Network Policies and provides advanced networking features such as IP address management, network policy enforcement, and network isolation.

3. **Romana**:
   - Romana is a container networking solution designed specifically for cloud-native applications and Kubernetes environments.
   - It offers built-in support for Kubernetes Network Policies, enabling you to define and enforce network access controls based on application requirements.

4. **Weave-net**:
   - Weave Net is a simple and lightweight container networking solution for Kubernetes and Docker.
   - It supports Kubernetes Network Policies and provides features like encrypted communication, automatic IP address management, and network segmentation.

### Kubernetes Manifest: Network Policies YAML
**Example 1:** This YAML defines a Kubernetes Network Policy named "db-policy" that controls network traffic to pods labeled with `role: db`. It permits Ingress traffic to pods labeled with `role: db` from pods labeled with `name: api-pod` in namespaces labeled with `name: prod` on TCP port 3306.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy  # Name of the Network Policy
spec:
  podSelector:
    matchLabels:
      role: db  # Select pods labeled with 'role: db'
  policyTypes:
  - Ingress  # Allow Ingress traffic

  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: api-pod  # Allow traffic from pods labeled with 'name: api-pod'
      namespaceSelector:
        matchLabels:
          name: prod  # Allow traffic from pods in the namespace labeled with 'name: prod'

    ports:
    - protocol: TCP  # Specify TCP protocol
      port: 3306  # Allow traffic on port 3306
```

**Example 2:** This YAML defines a Kubernetes Network Policy allowing both Ingress and Egress traffic. It permits Ingress traffic from pods labeled with `name: api-pod` on TCP port 3305, and it allows Egress traffic to the IP address 192.168.5.10 on TCP port 80.

```yaml
policyTypes:
  - Ingress  # Allow Ingress traffic
  - Egresss  # Allow Egress traffic

ingress:
  - from:
      - podSelector:
          matchLabels:
            name: api-pod  # Allow traffic from pods labeled with 'name: api-pod'

    ports:
      - protocol: TCP  # Specify TCP protocol
        port: 3305  # Allow traffic on port 3305

egress:
  - to:
      - ipBlock:
          cidr: 192.168.5.10/32  # Allow traffic to IP address 192.168.5.10

    ports:
      - protocol: TCP  # Specify TCP protocol
        port: 80  # Allow traffic on port 80

```


## Ingress Controller
### Ingresses
Typically, services and pods have IPs only routable by the cluster network. All traffic that ends up at an edge router is either dropped or forwarded elsewhere.

- **Ingress** is a collection of rules that allow inbound connections to reach the cluster services. 
- An **Ingress controller** is responsible for fulfilling the Ingress, usually with a loadbalancer, though it may also configure your edge router or additional frontends to help handle the traffic in an HA manner.

![Ingress](./images/09_13%20Ingress.png)

### Ingress Controller

An Ingress Controller is a Kubernetes resource responsible for managing ingress network traffic, typically acting as a reverse proxy to route incoming requests to services within the cluster.

**Instructions for deploying an NGINX Ingress Controller using Helm:**

Helm is a package manager for Kubernetes that simplifies the deployment and management of applications. It uses charts, which are pre-configured packages that contain all the Kubernetes resources necessary to run an application.

1. Create a new Kubernetes namespace named "ingress-basic"
    ```bash
    kubectl create namespace ingress-basic
    ```
    Namespaces provide a way to organize and isolate resources within a Kubernetes cluster.

2. Use Helm to install the NGINX Ingress Controller
    ```bash
    helm install nginx-ingress stable/nginx-ingress \
      --namespace ingress-basic \
      --set controller.replicaCount=2 \
      --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux \
      --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux \
      --set controller.service.loadBalancerIP="STATIC_IP"
    ``` 
   - `nginx-ingress` is the name assigned to the release.
   - `stable/nginx-ingress` is the Helm chart repository and chart name that specifies the NGINX Ingress Controller.
   - `--namespace ingress-basic` specifies the namespace in which to install the NGINX Ingress Controller. In this case, it's "ingress-basic".
   - `--set controller.replicaCount=2` sets the replica count for the NGINX Ingress Controller to 2. This means that two replicas of the NGINX Ingress Controller will be deployed to ensure high availability and redundancy.
   - `--set controller.nodeSelector."beta\.kubernetes\.io/os"=linux` sets a node selector for the NGINX Ingress Controller pods. This selector ensures that the pods are scheduled only on nodes with the label `beta.kubernetes.io/os=linux`.
   - `--set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux` sets a node selector for the default backend pod of the NGINX Ingress Controller. Similar to the previous option, this ensures that the default backend pod is scheduled only on nodes with the label `beta.kubernetes.io/os=linux`.
   - `--set controller.service.loadBalancerIP="STATIC_IP"` sets a static IP address for the NGINX Ingress Controller's LoadBalancer service. This is useful when you want to assign a specific IP address to the Ingress Controller's external load balancer.

These commands, when executed together, deploy an NGINX Ingress Controller to the Kubernetes cluster in the specified namespace, with specific configurations such as replica count, node selectors, and load balancer IP address.

**YAML definition of an Ingress resource**
This YAML defines an Ingress resource named "ingress-tripviewer" in the namespace "api-dev".


```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-tripviewer  # Name of the Ingress resource
  namespace: api-dev  # Namespace in which the Ingress resides
  annotations:
    kubernetes.io/ingress.class: nginx  # Specifies the Ingress controller to use (nginx)
    nginx.ingress.kubernetes.io/ssl-redirect: "false"  # Disables SSL redirect
    # nginx.ingress.kubernetes.io/rewrite-target: /$1

spec:
  rules:
  - http:  # HTTP rules for routing traffic
      paths:  # Paths for routing traffic to different services
      - backend:  # Backend service configuration
          serviceName: poisvc  # Service to route traffic to
          servicePort: 80  # Port of the service
        path: /api/poi(.*)  # Path pattern to match and route traffic to the specified service
      - backend:
          serviceName: tripssvc
          servicePort: 80
        path: /api/trips(.*)
      - backend:
          serviceName: userprofilesvc
          servicePort: 80
        path: /api/user(.*)
      - backend:
          serviceName: userjavasvc
          servicePort: 80
        path: /api/user-java(.*)
```

This Ingress configuration routes incoming HTTP traffic to different backend services based on the requested URL path patterns. When a request matches one of these paths, it's routed to the corresponding backend service specified by `serviceName` and `servicePort`.

![Ingress Controller](./images/09_14%20Ingress%20Controller.png)

### Exercise 11: Ingress Controller
**1) Enable Ingress Controller on Minikube:**
```bash
$ minikube addons enable ingress
```

**2) Verify that the Ingress controller is running:**
Option 1:
```bash
$ kubectl get pods -n kube-system
```

This command lists all the pods in the `kube-system` namespace, where Kubernetes system components, including the Ingress controller, are typically deployed. Look for pods with names related to the Ingress controller, such as `nginx-ingress-controller`, `traefik`, or any other Ingress controller you have installed. If the Ingress controller pod is running, it means that the Ingress controller is successfully deployed and running in your Minikube cluster.

Option 2:
```bash
minikube addons list
```

This command will display a list of available addons along with their statuses (enabled or disabled).

**3) Deploy an application and expose the service.**

Done in previous exercises.

**4) Create an Ingress to route traffic to the application.**

The ingress is created in `./exercise_11/exercise_11.yaml`.

Apply ingress:
```bash
kubectl apply -f exercise_11.yaml
```
Here we can see the IP address and port of our ingress.

**5) Verify the Ingress.**
Verify that the ingress is applied:
```bash
kubectl get ingress
```

Now we will do a curl (in Linux, it does not work in Windows):
```bash
curl --resolve "hello-world.info:80:$( minikube ip )" -i http://hello-world.info
```
This command sends an HTTP request to http://hello-world.info, but it directs the request to the IP address of the Minikube cluster. It's commonly used to test connectivity or to access services running in a Minikube cluster that are not exposed to the host machine directly.
- `--resolve "hello-world.info:80:$( minikube ip )"`: This option allows you to manually specify the IP address and port number to be used when making the request. In this case, it resolves the hostname `hello-world.info` to the IP address of the Minikube cluster. The `$( minikube ip )` command is a shell command substitution that retrieves the IP address of the Minikube cluster.
- `-i`: This option tells `curl` to include the HTTP response headers in the output.
- `http://hello-world.info`: The URL to which the HTTP request is made.





**6) Deploy another application using the image `gcr.io/google-samples/hello-app:2.0`, expose the service, and add it to the Ingress.**
