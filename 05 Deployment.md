# Deployment
Table of contents:
- [Deployment](#deployment)
  - [What is a Deployment?](#what-is-a-deployment)
  - [deployment-definition.yaml](#deployment-definitionyaml)
  - [Deployment commands](#deployment-commands)
  - [Deploying Applications in Kubernetes](#deploying-applications-in-kubernetes)
  - [Deployment Strategy](#deployment-strategy)
  - [Exercise 4:  Deployments and Pods](#exercise-4--deployments-and-pods)



## What is a Deployment?
A Deployment is a higher-level abstraction that orchestrates ReplicaSets and offers declarative updates to Pods, along with numerous other valuable functionalities.

It's advisable to utilize Deployments instead of directly interacting with ReplicaSets. This recommendation stands unless you specifically need custom update orchestration or do not require updates at all.

![Deployment](./images/03_06%20Deployments.png)

## deployment-definition.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata: # Metadata for the Deployment
  name: myapp-deployment # Name of the Deployment
  labels: # Labels associated with the Deployment
    app: myapp
    type: front-end
spec:  # Specification for the Deployment
  template: # Template for creating Pods managed by the Deployment
    metadata:
      name: myapp-pod
      labels: # Labels associated with the Pods created from this template
        app: myapp
        type: front-end
    spec: # Specification for the Pods created from this template
      containers: # Containers specification within the Pod
      - name: nginx-container
        image: nginx
  replicas: 3 # Number of replicas (Pods) to maintain
  selector: # Selector to determine which Pods are managed by the Deployment
    matchLabels:
      type: front-end
```

## Deployment commands
Create Deployment

```bash
kubectl create -f deployment-definition.yaml
```

List Deployments:

```bash
kubectl get deployments
```

View ReplicaSets:

```bash
kubectl get replicaset
```
We see that when we create a Deployment, it also creates its underlying ReplicaSet.

View Replica Pods:

```bash
kubectl get pods
```

Delete Deployment:

```bash
kubectl delete deployment myapp-deployment
```
This command will delete the specified Deployment along with all the Pods it manages.


## Deploying Applications in Kubernetes

**Deployment Objects:** In Kubernetes, a "Deployment" is an object used to manage the deployment of your application. It provides declarative updates to applications, ensuring that the desired state of the deployed application matches the actual state.

Here's a breakdown of what "Deployment Objects" entail:

1. **Creating new deployments:** When you create a new deployment object in Kubernetes, you define the desired state of your application, including things like the number of replicas (instances) of your application to run, the container image to use, and any other configurations.

2. **Updating deployments:** Deployments allow you to easily update your application by changing the deployment's configuration, such as updating the container image version or modifying other settings. Kubernetes automatically handles the rolling updates, ensuring that your application remains available and healthy during the update process.

3. **Applying rollback to previous versions:** If an update to your application causes issues or you need to revert to a previous version for any reason, Deployments support rollback functionality. You can rollback to a previous known good state by specifying the revision you want to rollback to, and Kubernetes will handle the process of reverting your application to that state.

In summary, "Deployment Objects" in Kubernetes provide a powerful and flexible way to manage the lifecycle of your applications, from initial deployment to updates and rollbacks, ensuring that your applications run smoothly and reliably.


Here are some useful commands for working with deployments:

- List Deployments:
    ```bash
    kubectl get deployments
    ```
    or
    ```bash
    kubectl get deploy
    ```

- Configure the image of a deployment:
    ```bash
    kubectl set image <deployment> <image>
    ```

- View the rollout status of deployments:
    ```bash
    kubectl rollout status deployment/<deployment-name>
    ```
    When you run this command, Kubernetes will provide information about the status of the deployment rollout, including whether the rollout is complete or still in progress, and any relevant events or errors. This is useful for monitoring the progress of deployments and ensuring that updates are applied successfully.

- View the rollout history, including previous revisions:
    ```bash
    kubectl rollout history deployment/<deployment-name>
    ```

- Pause a deployment:
    ```bash
    kubectl rollout pause deployment/<deployment-name>
    ```
    Kubernetes will pause the rollout of updates for the specified deployment. This means that no further changes or updates will be applied to the deployment until you resume the rollout. Pausing a rollout can be useful if you need to investigate issues or prevent unintended changes from being applied to your application.

- Resume a deployment:
    ```bash
    kubectl rollout resume deployment/<deployment-name>
    ```

- Rolling Back:
    ```bash
    kubectl rollout undo deployment/<deployment-name> --to-revision=<Revision-Number>
    ```
    Kubernetes will initiate a rollback of the specified deployment to the desired revision. Kubernetes will then handle the process of reverting the deployment to the previous state specified by the revision number. This can be useful if an update introduces issues or if you need to revert to a known good state.

## Deployment Strategy

In Kubernetes, the deployment strategy defines how updates are applied to your application's Pods. There are several deployment strategies available, each with its own approach to managing the update process and ensuring the availability of your application during updates.

![Deployment Strategy](./images/03_07%20Deployment%20Strategy.png)

**Rolling Update Strategy:**
- This is the default deployment strategy in Kubernetes.
- With rolling updates, Kubernetes gradually replaces old Pods with new ones, ensuring that your application remains available throughout the update process.
- Each new Pod is started before the corresponding old Pod is terminated, minimizing downtime and ensuring a smooth transition between versions.

**Recreate Strategy:**
- In this strategy, all existing Pods are terminated before new Pods are created.
- While this approach may result in more downtime compared to rolling updates, it ensures that all Pods are replaced simultaneously, providing a clean cut-over to the new version.

**Blue-Green Deployment:**
- Blue-green deployment involves running two identical production environments, one active (blue) and one inactive (green).
- Updates are applied to the inactive environment, allowing thorough testing before switching traffic to the updated version.
- Once testing is complete, traffic is switched from the blue environment to the green environment, making the updated version live.

**Canary Deployment:**
- Canary deployment is similar to blue-green deployment but involves routing a small percentage of traffic to the updated version (the canary) while the majority of traffic continues to be routed to the stable version.
- This allows for real-world testing of the update's performance and reliability before rolling it out to all users.

Each deployment strategy has its own trade-offs in terms of update speed, downtime, and risk mitigation. By understanding these strategies, you can choose the one that best fits your application's requirements and operational needs.

## Exercise 4:  Deployments and Pods

Create a Deployment in Kubernetes for a stateless application using a YAML file. Use Nginx. In the deployment, specify 2 replicas and give a "unique" name to our container as follows: NGINX-LASTNAME

See result in: `./exercise_04/nginx-deployment.yaml`

Create Deployment

```bash
kubectl create -f nginx-deployment.yaml
```

List the deployments:
```bash
kubectl get deployments
```

Inspect the deployment, view the events:
```bash
kubectl describe deployment nginx-deployment
```

Can our application be searched for by its name? How and why?

Yes, your application can be searched for by its name in Kubernetes. This is because Kubernetes uses labels and selectors to associate Pods with Services, Deployments, and other resources.

To search for your application by its name, we can use `kubectl get` command with the appropriate resource type and label selectors. For example,:

```bash
kubectl get pods -l app=nginx
```

This command will list all Pods with the specified label (in this case, the label `app` with the value equal to your application name).

Similarly, we can search for other Kubernetes resources such as Services, Deployments, ReplicaSets, etc., using label selectors based on the application name.
