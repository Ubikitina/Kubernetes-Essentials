# POD

Table of contents:
- [POD](#pod)
  - [What is a POD?](#what-is-a-pod)
  - [Example 1: Nginx Pod](#example-1-nginx-pod)
  - [Example 2: Stress test Pod](#example-2-stress-test-pod)
  - [Lifecycle of Pod Creation](#lifecycle-of-pod-creation)
  - [Exercise 1: kubectl dry-run feature](#exercise-1-kubectl-dry-run-feature)
  - [Exercise 2: xtabel/dotnet-core-publish](#exercise-2-xtabeldotnet-core-publish)


## What is a POD?

A Pod is the fundamental component in Kubernetes and serves as the "building block" of the system. It is used to deploy one or more containers and has several key characteristics:

- A Pod can contain multiple containers, for example, sidecars (tightly coupled).
- It encapsulates containers, storage, network IP, and execution options.
- It utilizes _Deployments_ to specify the types of resources used, such as:
  - ReplicaSet
  - StatefulSet
  - DaemonSet
  - Job
  - InitContainer

## Example 1: Nginx Pod
This is the file pod-definition.yaml:

```yaml
apiVersion: v1  # Specifies the API version being used
kind: Pod  # Defines the Kubernetes resource type as a Pod
metadata:  # Metadata section for the Pod
    name: myapp-pod  # Specifies the name of the Pod
    labels:  # Labels assigned to the Pod
        app: myapp  # Label "app" with value "myapp"
        type: front-end  # Label "type" with value "front-end"
spec:  # Specification section for the Pod
    containers:  # List of containers within the Pod
        - name: nginx-container  # Name of the container within the Pod
          image: nginx  # Docker image to be used for this container (NGINX in this case)
```

## Example 2: Stress test Pod
Another example of a pod-definition.yaml:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-demo
  namespace: mem-example  # Specifies the namespace in which the Pod resides as "mem-example"
spec:
  containers:
    - name: memory-demo-ctr
      image: polinux/stress
      resources:  # Section for specifying resource constraints for the container
        limits:  # Resource limits for the container
          memory: "200Mi"  # Sets the memory limit to 200MiB for the container
        requests:  # Resource requests for the container
          memory: "100Mi"  # Sets the memory request to 100MiB for the container
      command: ["stress"]  # Specifies the command to be executed within the container
      args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"]  # Specifies the arguments for the command executed within the container
```
This pod definition creates a Kubernetes pod named "pod-demo" in the "mem-example" namespace. Here's what this pod does:

- **Container**: The pod contains a single container named "memory-demo-ctr" running the Docker image "polinux/stress". This image is a stress-testing tool used to consume system resources.
  
- **Resources**: The container has resource constraints defined. It is limited to using a maximum of 200MiB of memory (`limits.memory: "200Mi"`) and is requested to use at least 100MiB of memory (`requests.memory: "100Mi"`).

- **Command and Arguments**: The container runs the command "stress" with specific arguments:
  - `--vm 1`: Specifies the number of virtual machines to start (1 in this case).
  - `--vm-bytes 150M`: Specifies the amount of memory to allocate for each virtual machine (150MB in this case).
  - `--vm-hang 1`: Specifies to hang (suspend) the virtual machines after starting (1 in this case).

In summary, this pod definition creates a pod running a stress-testing tool with resource constraints on memory. The tool starts one virtual machine with 150MB of memory and then hangs (suspends) it after starting. This can be used for testing how the system behaves under high memory usage scenarios.

## Lifecycle of Pod Creation

1. Execute `kubectl apply -f pod.yaml` to request the kube-apiserver to create a Pod resource.

2. The **kube-scheduler** continuously monitors Pods that are not yet assigned to any nodes. 
   - When a Pod is detected, the **kube-scheduler** determines which **node** should run the containers within the Pod. 
   - It then instructs the **kube-apiserver** to annotate the Pod with the node's name.

3. The **kubelet** monitors Pods assigned to the node where it is running. Upon discovering a Pod, the kubelet interacts with the **Container Runtime Interface (CRI)**, such as containerd in Google Kubernetes Engine (GKE), to instantiate containers.

4. The CRI pulls the necessary container image from the specified repository.

5. Finally, the CRI creates the container within the node, completing the Pod creation process.

![Lifecycle of Pod Creation](./images/03_03%20Lifeccycle%20of%20a%20Pod%20Creation.png)

## Exercise 1: kubectl dry-run feature
In the context of Kubernetes, a "dry-run" is a feature provided by the `kubectl` command-line tool that allows users to preview the changes that would be applied to a Kubernetes resource without actually executing those changes. When performing a dry-run, `kubectl` simulates the API request for creating, updating, or deleting a resource and returns the resulting YAML representation of the resource after applying the changes, without actually modifying the cluster state.


To perform a dry-run with `kubectl`, we use the `--dry-run=client` flag along with the appropriate command, such as `apply`, `create`, `replace`, or `delete`. For example:
```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
```

This command creates a new pod named "nginx" using the NGINX image, but instead of actually creating the pod in the cluster, it performs a dry run and outputs the resulting YAML representation of the pod to the `pod.yaml` file. So it represents the configuration of the pod that would be created if we were to apply it to the Kubernetes cluster.

See result in: `./exercise_01/pod.yaml`


Dry-run is particularly useful for:

1. **Testing Changes**: Before making actual changes to Kubernetes resources, users can use dry-run to test their configurations and ensure that they are correct.

2. **Auditing Changes**: Users can review the proposed changes to Kubernetes resources before applying them to the cluster, allowing for verification and auditing of the changes.

3. **Preventing Accidental Modifications**: Dry-run helps prevent accidental modifications to the cluster by allowing users to see the impact of their actions before committing to them.

## Exercise 2: xtabel/dotnet-core-publish
Practice 2

1. Generate a POD Manifest YAML (-o yaml) without creating it (--dry-run) for the DockerHub image: xstabel/dotnet-core-publish.

   ```bash
   kubectl run pod-demo --image=xstabel/dotnet-core-publish --dry-run=client -o yaml > pod.yaml
   ```

2. Use the created YAML to deploy the application.

   ```bash
   kubectl apply -f pod.yaml
   ```

3. List all pods from all namespaces.

   ```bash
   kubectl get pods --all-namespaces
   ```

4. List only the pods from the default namespace.

   ```bash
   kubectl get pods -n default
   ```

5. Inspect and describe our POD.

   ```bash
   kubectl describe pod pod-demo
   ```

6. Execute the console in a pod and run bash commands, for example, what do we get when we curl www.google.com.

   ```bash
   kubectl exec -it pod-demo -- /bin/bash
   ```

   Inside the pod, run:
   ```bash
   curl www.google.com
   ```
   Exit the console by typing `exit`.

7. Delete everything created.

   ```bash
   kubectl delete pod pod-demo
   ```

