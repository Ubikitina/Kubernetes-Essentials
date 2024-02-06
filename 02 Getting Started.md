# Creating a Kubernetes Cluster
Table of contents
- [Creating a Kubernetes Cluster](#creating-a-kubernetes-cluster)
  - [Prerequisites](#prerequisites)
  - [Setting up Minikube](#setting-up-minikube)
    - [Minikube](#minikube)
    - [Minikube Setup](#minikube-setup)
  - [Commands](#commands)
    - [Starting Minikube](#starting-minikube)
    - [kubectl commands](#kubectl-commands)
    - [Exercise: Interacting with K8S using Kubectl](#exercise-interacting-with-k8s-using-kubectl)
    - [Exercise: My first Pod](#exercise-my-first-pod)


## Prerequisites
How do we create a Kubernetes cluster?

To get started with K8S and develop locally, you have a few options:

1. **Minikube or Kind**: These are lightweight solutions for setting up a single-node Kubernetes cluster on your local machine.

2. **Create a multi-node cluster from scratch using the Hard Way**: This involves manually setting up a Kubernetes cluster with multiple nodes. 

3. **Using Kubeadm**: Kubeadm is a tool for easily creating Kubernetes clusters. It's suitable for setting up clusters on-premises or in the cloud.

4. **Managed Kubernetes clusters in the cloud**:
   - For Google Cloud Platform (GCP), you can use:
     ```
     gcloud container clusters create kuar-cluster
     ```
   - For Microsoft Azure, you can use:
     ```
     az aks create --resource-group=kuar --name=kuar-cluster
     ```
   - For Amazon Web Services (AWS), you can use:
     ```
     eksctl create cluster --name kuar-cluster
     ```

These options provide different levels of flexibility and management, allowing you to choose the approach that best fits your needs and resources.

## Setting up Minikube

### Minikube

Minikube offers the quickest way to get started with Kubernetes (K8S). It comes integrated with VSCode and Docker Desktop.

- It sets up a single-node Kubernetes cluster where you can run workloads and scale them across multiple instances.
- Compatible with Linux, macOS, and Windows, supporting various hypervisors:
  - KVM
  - VMware Fusion
  - xhyve
  - Hyper-V
  - Virtualbox
- Minikube automatically configures kubectl, enabling interaction with K8S via the console as well.
- [Minikube GitHub Repository](https://github.com/kubernetes/minikube)

### Minikube Setup

**Prerequisites**:
- Minikube and kubectl will be used as the tools for interaction and control of the K8S API.
- Computer administration rights to enable a hypervisor: VirtualBox/KVM on Linux or VirtualBox/Hyper-V.
- Download Minikube. On Linux, execute:
  ```bash
  curl -Lo minikube https://storage.googleapis.com/minikube/releases/v0.28.2/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
  ```
  On Windows, using Chocolatey:
  ```bash
  choco install minikube
  ```
- Finally, download and install kubectl using Linux or with Windows Subsystem for Linux (WSL):
  ```bash
  sudo apt-get update && sudo apt-get install -y apt-transport-https
  curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
  echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
  sudo apt-get update
  sudo apt-get install -y kubectl
  ```

**Minikube Configuration**:
```bash
$ minikube config -h
* vm-driver
* v
* cpus
* disk-size
* host-only-cidr
* memory
* log_dir
* kubernetes-version
* iso-url
* WantUpdateNotification
* ReminderWaitPeriodInHours
* WantReportError
* WantReportErrorPrompt
* WantKubectlDownloadMsg
* WantNoneDriverWarning
* profile
* bootstrapper
* ShowBootstrapperDeprecationNotification
```

The `minikube config -h` command displays the available configuration options for Minikube. Here's a brief explanation of each configuration option:

- **vm-driver**: Specifies the virtual machine driver to use (e.g., VirtualBox, KVM, Hyper-V).
- **v**: Verbosity level for the Minikube command output.
- **cpus**: Number of CPU cores to allocate for the Minikube virtual machine.
- **disk-size**: Size of the disk allocated for the Minikube virtual machine in MB.
- **host-only-cidr**: The CIDR block used for the host-only network.
- **memory**: Amount of memory to allocate for the Minikube virtual machine in MB.
- **log_dir**: Directory to store Minikube logs.
- **kubernetes-version**: Specifies the version of Kubernetes to install.
- **iso-url**: URL to the ISO image used for the Minikube virtual machine.
- **WantUpdateNotification**: Whether to receive update notifications.
- **ReminderWaitPeriodInHours**: Wait period for update reminders in hours.
- **WantReportError**: Whether to report errors automatically.
- **WantReportErrorPrompt**: Whether to prompt the user to report errors.
- **WantKubectlDownloadMsg**: Whether to display a message about downloading kubectl.
- **WantNoneDriverWarning**: Whether to display a warning when using the none driver.
- **profile**: The name of the Minikube profile.
- **bootstrapper**: Specifies the bootstrapper tool to use (e.g., kubeadm, kops).
- **ShowBootstrapperDeprecationNotification**: Whether to display a notification about bootstrapper deprecation.

These options allow you to customize various aspects of the Minikube setup to suit your requirements and preferences. You can use these options when running the `minikube config` command to configure Minikube according to your needs.

This should help you set up Minikube for your Kubernetes development environment.

## Commands

### Starting Minikube

Once Minikube and Kubectl are installed, you can start the cluster and verify its status:

| Command            | Description                                                                                                                                                                                                                         |
|--------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `minikube version` | Displays the version of Minikube currently installed on the system, providing information about the installed version for compatibility and ensuring the correct usage.                                                            |
| `minikube start`   | Initiates the Minikube virtual machine, configuring and launching the Kubernetes cluster environment locally on the machine, enabling the user to start utilizing Kubernetes for development and testing purposes.                  |
| `minikube status`  | Checks the status of the Minikube virtual machine and associated Kubernetes cluster, providing information about whether the virtual machine and Kubernetes components are running as expected.                                     |
| `minikube dashboard` | Opens the Kubernetes dashboard in the default web browser, providing a graphical user interface for managing and monitoring the Kubernetes cluster, allowing users to visualize resource usage and perform various management tasks. |
| `minikube delete`  | Removes the Minikube virtual machine and all associated resources, effectively shutting down and deleting the Kubernetes cluster, freeing up system resources and disk space, and cleaning up after Minikube usage.                 |

### kubectl commands
The kubectl status commands are as follows:
Here's a table summarizing the descriptions of each `kubectl` command:

| Command                               | Description                                                                                                                                                                                          |
|---------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `kubectl cluster-info`                | Retrieves information about the Kubernetes cluster, including the Kubernetes master's API server address and the cluster services' endpoints, giving an overview of the cluster's health and status. |
| `kubectl get componentstatuses`      | Displays the status of the various components running on the Kubernetes cluster, such as the etcd, scheduler, controller manager, and API server, providing insight into the health of the cluster's core components. |
| `kubectl get events`                 | Retrieves a list of events that have occurred within the Kubernetes cluster, showing information about various cluster activities, including pod scheduling, scaling events, and other resource lifecycle events. |
| `kubectl get pods --all-namespaces`  | Lists all pods deployed across all namespaces within the Kubernetes cluster, giving a comprehensive view of the running pods and their status, useful for monitoring and troubleshooting purposes.          |
| `kubectl get services --all-namespaces` | Lists all services deployed across all namespaces within the Kubernetes cluster, displaying information about the services' endpoints, ports, and associated labels, providing an overview of the cluster's network services. |

Using Kubectl to manage/interact with K8S:

| Command                    | Description                                                                                                                      |
|----------------------------|----------------------------------------------------------------------------------------------------------------------------------|
| `kubectl run hello-minikube` | Creates and deploys a new Kubernetes deployment named "hello-minikube", typically used to deploy applications or services.        |
| `kubectl cluster-info`       | Retrieves information about the Kubernetes cluster, including the address of the Kubernetes master's API server and its status.   |
| `kubectl get nodes`          | Retrieves a list of all nodes in the Kubernetes cluster, displaying information such as their names, statuses, and roles.        |

For an overview of kubectl, visit: [Kubectl Documentation](https://kubernetes.io/docs/reference/kubectl/)

### Exercise: Interacting with K8S using Kubectl

- **Start Minikube (specify the hypervisor)**

```bash
minikube start --vm-driver=<hypervisor> --extra-config=apiserver.runtime-config=apps/v1beta1=true,extensions/v1beta1/deployments=true
```
Starts Minikube with the specified hypervisor and additional configuration to enable the use of the `apps/v1` API.

- **Configure the context**

```bash
kubectl config use-context minikube
```
Sets the current context to Minikube, allowing subsequent `kubectl` commands to operate within the Minikube cluster.

- **Or select the context inline**

```bash
kubectl get pods --context=minikube
```
Executes the `kubectl get pods` command in the context of Minikube directly without changing the global context.

- **View the Kubernetes Dashboard**

```bash
minikube dashboard
```
Opens the Kubernetes Dashboard in the default web browser for visualization and management of the Minikube cluster.

- **View available addons**

```bash
minikube addons list
```
Lists the available addons that can be enabled or disabled in the Minikube cluster.

- **Kubectl commands: Context and Configuration**

These commands are used to manage the context and configuration of `kubectl` for interacting with Kubernetes. They include setting contexts, switching contexts, and configuring `kubectl` settings.

### Exercise: My first Pod
Deploying an Application:
```bash
kubectl run nginx --image=nginx
```
Wait until the POD is in the "Ready" state:
```bash
kubectl get pod --watch
```
To Delete a Pod:
```bash
kubectl delete pod nginx
```