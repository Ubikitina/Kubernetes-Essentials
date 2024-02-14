# Other Concepts
Table of contents:
- [Other Concepts](#other-concepts)
  - [Field Selector Commands](#field-selector-commands)
  - [Special Pods](#special-pods)
    - [Daemonsets](#daemonsets)
    - [StatefulSets](#statefulsets)
    - [Jobs](#jobs)
  - [Scaling](#scaling)
    - [Scaling Applications in Kubernetes](#scaling-applications-in-kubernetes)
    - [Exercise 12: Autoscaling](#exercise-12-autoscaling)
  - [Role Based Access Control (RBAC)](#role-based-access-control-rbac)
  - [StatefulSet](#statefulset)
  - [Operators](#operators)
  - [Storage](#storage)
    - [Volumes in Kubernetes](#volumes-in-kubernetes)
    - [How to Setup Volumes in Kubernetes](#how-to-setup-volumes-in-kubernetes)
    - [Persistent Volumes](#persistent-volumes)
    - [Exercise 13: Storage](#exercise-13-storage)
  - [Monitoring](#monitoring)
    - [What indicators matter in Kubernetes](#what-indicators-matter-in-kubernetes)
    - [Logging](#logging)
    - [Health Checks: livenessProbe and readinessProbe](#health-checks-livenessprobe-and-readinessprobe)
    - [Exercise 14: livenessProbes and readinessProbes](#exercise-14-livenessprobes-and-readinessprobes)
    - [Monitoring, Logging and Troubleshooting your cluster](#monitoring-logging-and-troubleshooting-your-cluster)
    - [Monitoring on Minikube](#monitoring-on-minikube)
    - [Monitoring by using Prometheus](#monitoring-by-using-prometheus)
    - [Monitoring using Grafana](#monitoring-using-grafana)
    - [Monitoring using Prometheus and Grafana (both)](#monitoring-using-prometheus-and-grafana-both)
    - [Exercise 15: Enable monitoring](#exercise-15-enable-monitoring)
  - [Tolerances and Affinities in Kubernetes](#tolerances-and-affinities-in-kubernetes)
    - [Scheduling with Node Selector](#scheduling-with-node-selector)
    - [Scheduling with Node Affinity](#scheduling-with-node-affinity)
    - [Taints and Tolerations](#taints-and-tolerations)


## Field Selector Commands

Field selectors, along with selectors in general, enable you to choose Kubernetes resources based on the value of one or more resource fields.

Here are some examples of field selector queries:

```bash
kubectl get pods --field-selector metadata.namespace!=default
kubectl get pods --field-selector metadata.name=my-service
kubectl get pods --field-selector status.phase=Running
```

Supported field selectors vary by Kubernetes resource type. All resource types support the `metadata.name` and `metadata.namespace` fields. Attempting to use unsupported field selectors will result in an error.

The supported operators are `=`, `==`, and `!=` with field selectors (where `=` and `==` have the same meaning). For instance, this `kubectl` command selects all Kubernetes Services that are not in the default namespace:

```bash
kubectl get services --all-namespaces --field-selector metadata.namespace!=default
```

Chained Selectors

```bash
kubectl get pods --field-selector=status.phase!=Running,spec.restartPolicy=Always
```

Multiple Resource Types

```bash
kubectl get statefulsets,services --all-namespaces --field-selector metadata.namespace!=default
```

These commands demonstrate how to effectively use field selectors to filter Kubernetes resources based on specific criteria.

## Special Pods
These special pods cater to specific use cases such as managing background tasks, ensuring consistent pod distribution, and maintaining persistent state across pod instances.

### Daemonsets
- Ensures that all (or some) nodes run a copy of a pod.
- It disregards whether a node is marked as unschedulable, meaning it can still schedule pods on those nodes.
- Can make pods (operate) even if the scheduler is not running.

### StatefulSets
- Used for workloads that require consistent, persistent hostnames such as databases like etcd (etcd-01, etcd-02) or CockroachDB.
- Each pod in a StatefulSet gets its own persistent volume storage.
- Example use cases include databases like:
  - Zookeeper, where each instance needs to maintain its state.
  - cockroachdb

### Jobs
- Creates one or more pods and ensures that a specified number of them  successfully terminate.
- There are three types of jobs:
  - **Non-parallel Jobs:** Only one pod is created, and it terminates when its task is complete and successful.
  - **Parallel Jobs with a fixed completion count:** Multiple pods are created, and the job completes when a certain number of them successfully terminate. E.g. one successful pod for each value in the range 1 to .spec.completions.
  - **Parallel Jobs with a work queue:** Pods coordinate with each other through a work queue, and the job terminates when one pod successfully completes its task.

## Scaling
### Scaling Applications in Kubernetes

- **ReplicaSets in K8S**:
  - ReplicaSets in Kubernetes are used to specify the number of instances of a POD in the platform.
  - ReplicaSets can be directly specified within deployment YAML files.
  
- **Autoscaling**:
  - Autoscaling can also be configured depending on the available resources.
  - The Horizontal Pod Autoscaler (HPA) is a feature that allows adjusting the replicas of a POD to meet the specified utilization.
  - There are several configuration options available to maintain resources in an acceptable state.
  - Example command for autoscaling:
    ```bash
    kubectl autoscale <Deployment-name> --min=10 --max=15 --cpu-percent=80
    ```

These adjustments help ensure that the application can handle varying loads efficiently and effectively utilize available resources in the Kubernetes cluster.

### Exercise 12: Autoscaling
You can scale any of our applications using HPA. This exercise demonstrates how HPA dynamically adjusts the number of replicas based on the application's resource utilization. 

In this exercise, we'll use the following deployment: [https://k8s.io/examples/application/php-apache.yaml](https://k8s.io/examples/application/php-apache.yaml).

**1) Apply HPA to our deployment, with a minimum of 1 replica and a maximum of 10 replicas, triggered when the CPU usage reaches 40%.**
```bash
kubectl apply -f https://k8s.io/examples/application/php-apache.yaml
kubectl autoscale deployment php-apache --cpu-percent=40 --min=1 --max=10
kubectl get hpa
```


**2) Test HPA with a heavy load by sending many requests to our Apache web application**

```bash
kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c 'while sleep 0.01; do wget -q -O- http://php-apache; done'
```
The latter will print us infinite "OK"-s. It is creating load. With the generated load, we expect it to scale at some point.

**3) Observe how HPA behaves and changes over time:**

We can review how it has escalated by making:
```bash
kubectl get po
kubectl get hpa
```

To output the information of a determined HPA (XXXX) in YAML format:
```bash
kubectl get hpa XXXX -o yaml
```

## Role Based Access Control (RBAC)
- RBAC in Kubernetes allows for the definition of dynamic policies to control access to the Kubernetes API based on roles and permissions.
- RBAC allows the creation of both:
  - "Roles", which are scoped to a specific namespace, and 
  - "ClusterRoles", which are applied cluster-wide.
- To grant permissions: Access can be granted to specific resources or API verbs, allowing fine-grained control over permissions. It contain users, groups, service accounts.
  - Use "RoleBindings" for within namespaces.
  - Use "ClusterRoleBindings" for cluster-wide access

**Example rules:**
```
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["batch", "extensions"]
  resources: ["jods"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```

- **rules**: This section defines the rules that specify what actions are permitted for a given role.
- **apiGroups**: Specifies the API groups for which the rules apply. An empty string ("") represents the core API group.
- **resources**: Defines the Kubernetes resources to which the rules apply. For example, "pods" and "jobs".
- **verbs**: Specifies the actions that are permitted for the specified resources. Common verbs include "get", "list", "watch", "create", "update", "patch", and "delete".


These rules demonstrate how RBAC can be used to control access to specific Kubernetes resources and actions, providing a foundation for secure and granular access control within Kubernetes clusters.


## StatefulSet
StatefulSet is a Kubernetes workload API object designed for managing stateful applications. It offers precise control over the deployment and scaling of a set of Pods, ensuring strict guarantees regarding the ordering and uniqueness of these Pods.

**Headless Services:**
In scenarios where load balancing and a single Service IP aren't necessary, you have the option to create "headless" Services. This is achieved by explicitly setting the cluster IP to "None" in the `.spec.clusterIP` field. Headless Services provide a way to interface with other service discovery mechanisms, allowing flexibility without being confined to Kubernetes' native implementation.

![StatefulSet](./images/10_01%20StatefulSet.png)

## Operators

Operators are a powerful way to automate the management of complex applications on Kubernetes. They allow developers to codify best practices and operational knowledge into software that can autonomously manage and operate applications in a Kubernetes environment. They are an extension of Kubernetes' declarative API.

[For further exploration, visit OperatorHub.io](https://operatorhub.io/)

**Operators Extend Kubernetes Functionality:**

Operators significantly expand the capabilities of Kubernetes, making it more manageable and accessible than ever before. In contrast to the past, operators offer a streamlined approach to extending Kubernetes functionality. As exemplified by the introduction of the original etcd Operator and Prometheus Operator by coreOS, operators address the challenge of managing stateful applications, a task traditionally considered difficult. A prime illustration of this is database operators, which empower Kubernetes users to securely deploy and oversee various databases without the need to develop custom workarounds.

**Operators Systematize Human Knowledge as Code:**

Kubernetes facilitates the automation of infrastructure, alleviating the operational burden associated with managing containerized applications. Operators play a pivotal role in this automation process by translating human expertise into executable code. Much like reusable templates, operators standardize application management, eliminating the need to start from scratch for each deployment.

**Operators Enable Standardization:**

Operators function akin to reusable templates, streamlining and standardizing application management processes. With operators, organizations can automate various aspects of application lifecycle management, fostering consistency and efficiency across deployments. This eliminates the need to reinvent the wheel for each new application or service.

**Operators Improve Resiliency:**

By defining the installation, scaling, updates, and management lifecycle of complex distributed applications, operators enhance resilience in Kubernetes environments. Particularly in the realm of highly intricate distributed database management, operators simplify the process, ensuring robust and reliable operation throughout the application lifecycle.

## Storage
### Volumes in Kubernetes

Volumes are associated with Pods to preserve data, and only the Pod can access them. Kubernetes supports various types of volumes:

- Different storage types from Cloud providers.
- SAN-type, file systems, etc.
- Local support only for testing purposes, such as in Minikube.
- Cloud Provider: Azure Files and Azure Disks, AWS EBS, Google Compute Engine Persistent Disk.

Some types of volumes may provide the ability to share files between Pods.

### How to Setup Volumes in Kubernetes

- Define the volume.
- Request a volume: Claims Volume.
- The platform chooses from the ones defined.
- Attach the volume to the Pod.

![How claims and volumes interact](./images/10_02%20How%20claims%20and%20volumes%20interact.png)

In the Pod definition, specify where and what to mount:
- The `spec.volumes` field indicates the required volume.
- In the `spec.containers.volumeMounts` field, specify where to mount it.

### Persistent Volumes

Using Persistent Volumes:

- A Persistent Volume is designed to provide data persistence functionality.
- The workflow for using volumes:
  - Provision a Persistent Volume: Virtual disk or specific hardware.
  - PODs establish a claim: `PersistentVolumeClaim`. This is used to request storage by a user/Pod. It specifies the name, size, and type of storage.
  - Kubernetes uses the claims to search for the defined volume that satisfies the requirements.
- By provisioning the volume and requesting volume claims and executing the Pod, Kubernetes makes persistent storage available.

Example of a `PersistentVolume` YAML:

```yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

Example of a `PersistentVolumeClaim` YAML:

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: task-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```

### Exercise 13: Storage
**1) Let's create a 1Gi PersistentVoluemClaim (PVC) and see what happens behind the scenes, and then clean up.**

File created: `./exercise_13/volume-claim.yaml`

Apply:
```bash
kubectl apply -f volume-claim.yaml
```

See that it is created:
```bash
kubectl get pvc
```

See the details of the pvc:
```bash
kubectl describe pvc my-pv-claim
```

Remove:
```bash
kubectl delete pvc my-pv-claim
```

**2) Create our stateful app to use persistent volumes.**
- We start with an NGINX image `k8s.gcr.io/nginx-slim:0.8`.
- We create a volume claim with "ReadWriteOnce" access.
- Mount the volume to the app's path: `/usr/share/nginx/html`.

File created:`./exercise_13/stateful-set.yaml`

Apply:
```bash
kubectl apply -f stateful-set.yaml
```

See that the StatefulSet is created:
```bash
kubectl get stateFulSet
```

This will run two PODs, we can see more details about them by doing:
```bash
kubectl get pods
```

**3) Validate the creation of PVs and PVCs.**

See that the PVC-s are created:
```bash
kubectl get pvc
```

Delete all:
```bash
kubectl delete all --all
```

Or delete only the pvc-s:
```bash
kubectl delete pvc www-web-0 www-web-1
```

We can check that the PVCs have been deleted by doing `kubectl get pvc`.

## Monitoring
### What indicators matter in Kubernetes
- Cluster health: total number of nodes, pods, etc
- Node health: available compute resources, health
- Application health

Tooling Examples:
- Datadog
- Prometheus
- Cloud-specific: Azure Monitoring, CloudWatch (AWS)

### Logging
- Pod logs: applications must log to standard out!
- Cluster-wide event logs
- Logging forwarder solutions
  - fluentd
  - logstash
- Log indexers
  - Splunk
  - ELK stack
  - Cloud providers solutions

### Health Checks: livenessProbe and readinessProbe
There are two types of Health Checks:
- **Readiness Probes**: These checks ensure that the Pod has loaded what it needs internally from the image and is ready to receive requests from external services.
- **Liveness Probes**: These checks determine the health of the Pod, ensuring that it can continue to receive requests.

Health Checks in a Deployment YAML:
```yaml
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: tomcat-deployment
spec:
  selector:
    matchLabels:
      app: tomcat
    replicas: 4
    template:
      metadata:
        labels:
          app: tomcat
      spec:
        containers:
        - name: tomcat
          image: tomcat:9.0
          ports:
          - containerPort: 8080
           # Liveness Probe checks if the container is alive by sending an HTTP GET request to the specified path and port
          livenessProbe:
            httpGet:
              path: /
              port: 8080
            # Initial delay in seconds before the first probe is executed
            initialDelaySeconds: 30
            # Period in seconds between each probe
            periodSeconds: 30
          # Readiness Probe checks if the container is ready to receive traffic by sending an HTTP GET request to the specified path and port
          readinessProbe:
            httpGet:
              path: /
              port: 8080
            initialDelaySeconds: 15
            periodSeconds: 3
```

### Exercise 14: livenessProbes and readinessProbes
Add livenessProbes and readinessProbes to a YAML.
![Exercise 14](./images/10_03%20Exercise%2014.png)

### Monitoring, Logging and Troubleshooting your cluster
- Log Everything to stdout / stderr
- Key Metrics:
  - Node metrics (CPU Usage, Memory Usage, Disk Usage, Network Usage)
  - Kube_node_status_condition
  - Pod memory usage / limit; memory_failures_total
    - container_memory_working_set_bytes
  - Pod CPU usage average / limit
  - Filesystem Usage / limit
  - Network receive / transmit errors
- There are many free and paid packages and services for K8S and beyond:
  - Performance analytics with Kubernetes dashboard
  - Central logging
  - Detecting problems at the node level
  - Troubleshooting scenarios
  - Using Prometheus

### Monitoring on Minikube
- In most of the Cloud-based platforms it comes enabled.
- On Minikube, youʼll have to enable the add-on
  - Run the following command:
    ```bash
    $ minikube addons enable metrics
    ```
  - Wait some minutes

### Monitoring by using Prometheus

Prometheus is a pull-based system. It sends an HTTP request, a so-called scrape. The response to this scrape request is stored and parsed in storage along with the metrics for the scrape itself.

The storage is a custom database on the Prometheus server and can handle a massive influx of data. Itʼs possible to monitor thousands of machines simultaneously with a single server.

![Prometheus](./images/10_04.%20Prometheuspng.png)

Install Prometheus using Helm:
- Install HELM: https://helm.sh/docs/intro/install/
    ```bash
    $ helm version
    $ helm repo add stable https://kubernetes-charts.storage.googleapis.com/
    ```
- Install prometheus
    ```bash
    $helm install prometheus stable/prometheus --namespace monitoring --dry-run
    ```

Monitoring with Prometheus:

```bash
# Export the name of the Prometheus pod in the 'monitoring' namespace
$ export POD_NAME=$(kubectl get pods --namespace monitoring -l "app=prometheus,component=server" -o jsonpath="{.items[0].metadata.name}")

# Port forward to access Prometheus on port 9090
$ kubectl --namespace monitoring port-forward $POD_NAME 9090
```

For queries in Prometheus, refer to the following documentation: [Prometheus Querying Basics](https://prometheus.io/docs/prometheus/latest/querying/basics/)


### Monitoring using Grafana
Installation steps:

```bash
# Install Grafana using Helm
$ helm install stable/grafana

# Step 1: Retrieve the 'admin' user password
$ kubectl get secret --namespace default mygrafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

# Step 2: Access Grafana server via port 80
$ mygrafana.default.svc.cluster.local

# Step 3: Get Grafana URL for port forwarding
$ export POD_NAME=$(kubectl get pods --namespace default -l "app=grafana,release=mygrafana" -o jsonpath="{.items[0].metadata.name}")
$ kubectl --namespace default port-forward $POD_NAME 3000

# Step 4: Login with the provided password and username: admin
```

Get Grafana URL:

```bash
# List minikube services to retrieve Grafana URL
$ minikube service list
```

For Grafana Dashboards, you can download JSON from [here](https://grafana.com/grafana/dashboards/10856), then open `http://grafana_url/dashboard/import` to upload the JSON and configure Prometheus as the Data source.

### Monitoring using Prometheus and Grafana (both)
Once the Prometheus Operator is up and running along with Grafana and the Alertmanager, you can access their UIs and interact with the different components:
- Prometheus UI on node port 30900
- Alertmanager UI on node port 30903
- Grafana on node port 30902
- Prometheus supports a dizzying array of metrics to choose from.

![Prometheus Dashboard](./images/10_05%20Prometheus%20Dashboard.png)


### Exercise 15: Enable monitoring

Enable monitoring in our environment.

```bash
minikube addons list
minikube addons enable metrics
```

Send load to our app, let's see the queries in Prometheus and Grafana.


## Tolerances and Affinities in Kubernetes

### Scheduling with Node Selector

- Label selectors are utilized to identify a subset of nodes in a Kubernetes cluster designated for scheduling a specific Pod.
  - Example: This command applies a label "hardware:highmem" to the node named "aks-nodepool1" in the Kubernetes cluster:
    ```bash
    kubectl label node aks-nodepool1 hardware:highmem
    ```
- By default, all nodes in the cluster are considered potential candidates for scheduling. However, by populating the spec.nodeSelector field in a Pod or PodTemplate, the initial set of nodes can be narrowed down to a subset.

YAML example:
```yaml
kind: Pod
apiVersion: v1
metadata:
  name: tf-mnist
spec:
  containers:
  - name: tf-mnist
    image: microsoft/samples-tf-mnist-demo:gpu
    resources:
      requests:
        cpu: 0.5
        memory: 2Gi
      limits:
        cpu: 4.0
        memory: 16Gi
    nodeSelector: # Node selector for scheduling the Pod
      hardware: highmem # Selects nodes with the label "hardware: highmem"
```

![Node Selector](./images/10_06%20Node%20Selector.png)


### Scheduling with Node Affinity
The notion of affinity was added to node selection via the affinity structure in the Pod spec. Affinity is a more complicated structure to understand, but it is significantly **more flexible** if you want to express more complicated scheduling policies.

Types:
- requiredDuringSchedulingIgnoredDuringExecution
- preferredDuringSchedulingIgnoredDuringExecution
- requiredDuringSchedulingRequiredDuringExecution(planned)

Command:
```bash
kubectl label node aks-nodepool1 hardware:highmem
```

Adds a label `hardware: highmem` to the Kubernetes node named `aks-nodepool1`. This label can be used for node selection in various Kubernetes resources such as pods, deployments, and services.

YAML applying affinity:
```yaml
kind: Pod
apiVersion: v1
metadata:
  name: tf-mnist
spec:
  containers:
  - name: tf-mnist
    image: microsoft/samples-tf-mnist-demo:gpu
    resources:
      requests:
        cpu: 0.5
        memory: 2Gi
      limits:
        cpu: 4.0
        memory: 16Gi
  affinity: # Affinity settings for pod scheduling
    nodeAffinity: # Node affinity rules
      requiredDuringSchedulingIgnoredDuringExecution: # The node must satisfy this condition for scheduling and is not enforced during pod execution
        nodeSelectorTerms: # List of node selector terms
        - matchExpressions: # Specifies the match expressions for node selection
          - key: hardware # Select nodes based on the 'hardware' label
            operator: In # Define the operator to match the label values
            values: highmem # Match nodes with the label value 'highmem'
```

### Taints and Tolerations
Scheduler can use taints and tolerations to restrict what workloads can run on nodes.

- A **taint** is applied to a node that indicates only specific pods can be scheduled on them.
- A **toleration** is then applied to a pod that allows them to tolerate a node's taint.

Command:
```bash
kubectl describe node node01 | grep -i taint
```
This command is used to describe the taints applied to a Kubernetes node named "node01".

- `kubectl describe node node01`: describe the details of a Kubernetes node named "node01". This includes various information about the node such as its capacity, allocated resources, labels, etc.

- `|`: This is a pipe operator in Linux, which is used to redirect the output of one command as input to another command.

- `grep -i taint`: This part of the command uses 
  - `grep` which is a command-line utility for searching plain-text data sets for lines that match a regular expression.
  - `taint` It's searching for lines that contain the word "taint", ignoring case (-i flag). 

This is useful for filtering the output of the `kubectl describe` command to only show information related to taints applied to the node.


Command:
```bash
kubectl taint node aks-nodepool1 sku=gpu:NoSchedule
```
The `kubectl taint node` command is used to **apply a taint** to a specific **Kubernetes node**:

- `aks-nodepool1`: Refers to the name of the node pool (a group of nodes) in your **Azure Kubernetes Service (AKS)** cluster.
- `sku=gpu:NoSchedule`: This is the actual taint being applied. It consists of two parts:
    - `sku=gpu`: The key-value pair where `sku` is the key, and `gpu` is the value. This represents the taint.
    - `NoSchedule`: The effect of the taint. In this case, it prevents new pods from being scheduled on the tainted node.


YAML:
```yaml
kind: Pod
apiVersion: v1
metadata:
  name: tf-mnist
spec:
  containers:
  - name: tf-mnist
    image: microsoft/samples-tf-mnist-demo:gpu
    resources:
      requests:
        cpu: 0.5
        memory: 2Gi
      limits:
        cpu: 4.0
        memory: 16Gi
  tolerations:
  - key: "sku"
    operator: "Equal"
    value: "gpu"
    effect: "NoSchedule"
```

![Taints and Tolerations](./images/10_07%20Taints%20and%20Tolerations.png)