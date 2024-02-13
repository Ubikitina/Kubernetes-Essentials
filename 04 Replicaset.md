# Replicaset
Table of contents:
- [Replicaset](#replicaset)
  - [What is a Replicaset?](#what-is-a-replicaset)
  - [replicaset-definition.yaml](#replicaset-definitionyaml)
  - [ReplicaSet commands](#replicaset-commands)
  - [Exercise 3: Pods removal and ReplicaSet removal](#exercise-3-pods-removal-and-replicaset-removal)

## What is a Replicaset?
A ReplicaSet ensures that the specified number of PODs are running. It is used to maintain a stable set of replicated pods running within a cluster at any given time.

![ReplicaSet](./images/03_04%20ReplicaSet.png)

## replicaset-definition.yaml

Here's the `replicaset-definition.yaml` file:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset  # Specifies the name of the ReplicaSet.
  labels:
    app: myapp  # Assigns the label "app: myapp" to the ReplicaSet.
    type: front-end  # Assigns the label "type: front-end" to the ReplicaSet.
spec:
  template:  # Defines the template for creating Pods controlled by this ReplicaSet.
    metadata:
      name: myapp-pod  # Specifies the name of the Pods created from this template.
      labels:
        app: myapp  # Assigns the label "app: myapp" to the Pods created from this template.
        type: front-end  # Assigns the label "type: front-end" to the Pods created from this template.
    spec:
      containers:
      - name: nginx-container  # Specifies the name of the container within the Pod.
        image: nginx  # Specifies the Docker image to use for the container.
  replicas: 3  # Specifies that there should be 3 replicas (PODs) of the specified template.
  selector:
    matchLabels:
      type: front-end  # Specifies the labels used to select the Pods controlled by this ReplicaSet.
```


The provided YAML code defines a ReplicaSet in Kubernetes, which is responsible for ensuring a specified number of identical PODs (replicas) are running at all times. Here's a brief explanation of what the code does:

1. `apiVersion: apps/v1`: Specifies the Kubernetes API version being used for the ReplicaSet resource.
2. `kind: ReplicaSet`: Defines the type of Kubernetes resource being created, in this case, a ReplicaSet.
3. `metadata`: Contains metadata about the ReplicaSet, including its name and labels.
4. `spec`: Describes the desired state of the ReplicaSet, including the number of replicas to maintain and the template for creating Pods.
   - `replicas: 3`: Specifies that the ReplicaSet should ensure there are three replicas (PODs) running at all times.
   - `selector`: Defines the criteria for selecting which Pods are controlled by the ReplicaSet.
     - `matchLabels`: Specifies the labels used to select the Pods. In this case, it selects Pods with the label `type: front-end`.
   - `template`: Defines the template for creating Pods controlled by the ReplicaSet.
     - `metadata`: Contains metadata for the Pods created from this template, including their name and labels.
     - `spec`: Specifies the specification for the Pods, including the containers to run.
       - `containers`: Defines the containers to run in the Pods.
         - `name: nginx-container`: Specifies the name of the container.
         - `image: nginx`: Specifies the Docker image to use for the container (in this case, `nginx`).

This image shows how `matchLabels` works: it is only able to control Pods that have `tier:front-end`.
![matchLabels](./images/03_05%20matchLabels.png)

## ReplicaSet commands
Replica Set is used for scaling instances (PODs) in a Kubernetes cluster. Let's see some of the commands we can use:


1. Create a ReplicaSet using the configuration defined in the `replicaset-definition.yaml` file. It instructs Kubernetes to create the specified ReplicaSet based on the provided YAML configuration.
    ```bash
    kubectl create -f replicaset-definition.yaml
    ```

2. Retrieve information about ReplicaSets in the Kubernetes cluster. It lists all ReplicaSets along with their current status, including the number of replicas they manage.
    ```bash
    kubectl get replicaset
    ```

3. Delete the ReplicaSet named `myapp-replicaset` from the Kubernetes cluster. Deleting a ReplicaSet also deletes all underlying PODs managed by that ReplicaSet.
    ```bash
    kubectl delete replicaset myapp-replicaset
    ```

4. Replace the existing ReplicaSet with the configuration defined in the `replicaset-definition.yaml` file. It updates the ReplicaSet's configuration without changing its name or other metadata.
    ```bash
    kubectl replace -f replicaset-definition.yaml
    ```

5. Scale the number of replicas (PODs) managed by the ReplicaSet defined in the `replicaset-definition.yaml` file to 6. It adjusts the number of replicas to match the desired state specified in the YAML configuration file.
    ```bash
    kubectl scale --replicas=6 -f replicaset-definition.yaml
    ```
6. Scale the number of replicas (PODs) managed by the ReplicaSet named `myapp-replicaset` to 6 (in this case, without using the .yaml). It increases or decreases the number of replicas to match the desired state defined in the ReplicaSet.
    ```bash
    kubectl scale --replicas=6 replicaset myapp-replicaset
    ```

## Exercise 3: Pods removal and ReplicaSet removal

Create ReplicaSet YAML for the application from Exercise 2 with 3 replicas and apply the YAML. We name our ReplicaSet "front" (using the image `xstabel/dotnet-core-publish`).

See result in: `./exercise_03/replicaset.yaml`

```bash
kubectl apply -f replicaset.yaml
```

Remove the pods, what happens?

```bash
kubectl delete pods --all
```
Even though my Pods have been deleted, the ReplicaSet generates three other Pods.

Scale down the same application, now reducing to 1 replica using the command line or YAML.

```bash
kubectl scale --replicas=1 replicaset front
```

View all existing ReplicaSet objects.

```bash
kubectl get replicaset
```

Delete the ReplicaSet, what happens?

```bash
kubectl delete replicaset front
```
Now when we do `kubectl get replicaset` and `kubectl get pods` we get nothing.

