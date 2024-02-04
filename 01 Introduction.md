# Introduction
- [Introduction](#introduction)
  - [What is Kubernetes?](#what-is-kubernetes)
  - [Kubernetes Background](#kubernetes-background)
  - [Capabilities of Kubernetes](#capabilities-of-kubernetes)
  - [Kubernetes Design](#kubernetes-design)
    - [Declarative Configuration](#declarative-configuration)
    - [Reconciliation or Controllers](#reconciliation-or-controllers)
    - [Labels: Implicit or Dynamic Grouping](#labels-implicit-or-dynamic-grouping)
  - [Kubernetes Architecture](#kubernetes-architecture)
    - [Components of the Kubernetes Master](#components-of-the-kubernetes-master)
    - [Components of Kubernetes Workers](#components-of-kubernetes-workers)


## What is Kubernetes?

Kubernetes is a comprehensive platform that encompasses a vast array of services and capabilities. One of its key advantages lies in its consistent treatment of one or a hundred computers alike, enabling users to efficiently leverage resources for their software consistently, regardless of their geographical location. While its core functionality involves scheduling workloads in containers across diverse infrastructures, Kubernetes goes beyond this fundamental capability.

For detailed documentation, refer to the official Kubernetes documentation: [Kubernetes Documentation](https://kubernetes.io/docs/home/)


## Kubernetes Background
Kubernetes, an open-source system, facilitates the automation of deployment, scaling, and management for containerized applications. Some key aspects include:

- **Container Orchestration:** Kubernetes schedules and executes application containers across a cluster of machines, streamlining the deployment process.

- **Release Information:** Kubernetes v1.0 was released on July 21, 2015, by the collaborative efforts of Joe Beda, Brendan Burns, and Craig McLuckie.

- **Inspirations from Google:** Drawing inspiration from Google's internal systems like Borg and Omega, Kubernetes has been developed to encapsulate the experiences gained from these systems. Relevant research papers for Borg [here](https://research.google.com/pubs/pub43438.html) and Omega [here](https://research.google.com/pubs/pub41684.html) provide deeper insights.

  - **Lessons Learned:** A comprehensive perspective on the evolution of container management systems is covered in the research paper titled "Borg, Omega, and Kubernetes. Lessons learned from three container-management systems over a decade," available [here](http://queue.acm.org/detail.cfm?id=2898444).

- **Container Technology Support:** Kubernetes is evolving by deprecating Docker containers on K8S while extending support for alternative container technologies like containerd, rkt, and CRI-O. This adaptability ensures compatibility with various container runtimes.

![The Kubernetes Ecosystem](./images/01_01%20The%20Kubernetes%20Ecosystem.jpg)


## Capabilities of Kubernetes

Kubernetes offers a rich set of capabilities, including but not limited to:

- Mounting storage systems
- Distributing secrets
- Monitoring application health
- Replicating application instances
- Implementing horizontal pod autoscaling
- Facilitating naming and discovery
- Balancing loads
- Conducting rolling updates
- Monitoring resources
- Accessing and ingesting logs
- Debugging applications
- Providing authentication and authorization



## Kubernetes Design

### Declarative Configuration

Kubernetes employs a declarative configuration approach, differing from imperative configurations where users take direct actions (e.g., creating individual replicas). The declarative approach involves specifying the desired state, granting the system a comprehensive understanding of the user's intent. This allows Kubernetes to autonomously take actions, facilitating automatic self-correction and self-healing behaviors.

### Reconciliation or Controllers

In a departure from a monolithic controller design, Kubernetes adopts a decentralized approach. Instead of a single, centralized controller, Kubernetes is composed of numerous controllers, each executing its independent reconciliation loop. Each controller is responsible for a specific aspect of the system, operating autonomously and unaware of the broader system, promoting a modular and scalable design.

### Labels: Implicit or Dynamic Grouping

Implicit or dynamic grouping refers to defining a group without explicitly listing its members. For example, stating, "The members of my team are the people wearing orange," implies the group without explicitly listing its members. 

In Kubernetes, this concept is achieved through labels, label queries, or label selectors. Each API object in Kubernetes can be associated with an arbitrary number of key/value pairs called "labels," facilitating implicit grouping by evaluating the group definition against a set of objects.

## Kubernetes Architecture

![Kubernetes Architecture](./images/01_02%20Kubernetes%20Architecture.png)


1. **User Interaction:** Kubernetes users interact with the API server to apply the desired state.
   
2. **Master Node:** Master nodes actively **enforce the desired state** on worker nodes.
   
3. **Worker Nodes:** 
     - facilitate communication** between containers.
     - support communication from the Internet.

### Components of the Kubernetes Master

The Kubernetes master consists of several components:

- **Kube API Server:** Exposes the Kubernetes REST API, scalable horizontally, and the embodiment of the Kubernetes control plane.
  
- **Etcd:** A reliable, distributed data store storing the entire cluster state. Typically configured as a three-node or five-node cluster for redundancy and high availability.

- **Kube Controller Manager:** A collection of various managers, including the replication controller, pod controller, services controller, and endpoints controller. They watch over the cluster's state through the API, ensuring it aligns with the desired state.

- **Cloud Controller Manager:** Allows cloud providers to integrate their platforms for managing nodes, routes, services, and volumes when running in the cloud.

- **DNS Service:** Scheduled as a regular pod, provides DNS names for services and pods, facilitating automatic discovery.

- **Kube-Scheduler:** Responsible for scheduling pods into nodes, considering multiple interacting factors.

![Components of the Kubernetes Master](./images/01_03%20Components%20of%20the%20Kubernetes%20Master.png)

**Kube API Server**

Operating the API server involves three core functions:

- **API Management:** Exposes and manages APIs.
  
- **Request Processing:** Processes individual API requests from clients.
  
- **Internal Control Loops:** Manages background operations necessary for the API server's successful operation.

**Scheduler**

The scheduler's primary responsibility is to schedule containers to worker nodes in the cluster. It determines which node to schedule a pod onto, and various factors, including labels, affinity, taints, and tolerations, control the scheduling process.

### Components of Kubernetes Workers

1. **Kube Proxy:**
   - Performs low-level network housekeeping on each node.
   - Reflects Kubernetes services locally and supports TCP and UDP forwarding.
   - Discovers cluster IPs through environment variables or DNS.

2. **Container Runtime (Docker/Containerd, Rkt):**
   - Manages the execution of containers on the node.
   - Supports various container runtimes, including Docker, Containerd, and Rkt.

3. **Kubelet:**
   - Acts as the Kubernetes representative on the node.
   - Communicates with master components.
   - Manages running pods, performing actions such as:
     - Downloading pod secrets from the API server.
     - Mounting volumes.
     - Executing the pod's container (through the Container Runtime Interface (CRI) or Rkt).
     - Reporting the status of the node and each pod.
     - Running container liveness probes.

