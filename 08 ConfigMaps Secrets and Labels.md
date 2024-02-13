# ConfigMaps, Secrets and Labels
Table of contents
- [ConfigMaps, Secrets and Labels](#configmaps-secrets-and-labels)
  - [ConfigMaps](#configmaps)
    - [Introduction](#introduction)
    - [Creating ConfigMaps](#creating-configmaps)
    - [How to use ConfigMaps in Kubernetes Pods](#how-to-use-configmaps-in-kubernetes-pods)
  - [Secrets](#secrets)
    - [Introduction](#introduction-1)
    - [Creating a Secret](#creating-a-secret)
      - [Option 1: From a File](#option-1-from-a-file)
      - [Option 2: From the literal on the command line](#option-2-from-the-literal-on-the-command-line)
    - [Exercise 8: Secrets](#exercise-8-secrets)
    - [How to use Secrets in Kubernetes Pods](#how-to-use-secrets-in-kubernetes-pods)
  - [Labels and selectors](#labels-and-selectors)
    - [Labels](#labels)
    - [Label Selectors](#label-selectors)
    - [Exercise 9: Kubectl commands for Labels](#exercise-9-kubectl-commands-for-labels)


## ConfigMaps

### Introduction

ConfigMaps serve as a flexible configuration model within the Kubernetes ecosystem. They are designed to manage configuration data separately from application code, allowing for easier configuration changes without altering the application itself. Here are key points to understand about ConfigMaps:

1. **Flexible Configuration Management**: They enable decoupling configuration from application code, promoting a more modular and maintainable architecture.

2. **Usage of ConfigMaps**: Are particularly suited for storing configuration that **doesn't contain sensitive information**. However, it's important to weigh the pros and cons of using ConfigMaps for different types of configuration data.

3. **String-Based Configuration**: ConfigMaps are designed to conveniently handle string-based configuration data. They support storing and managing key-value pairs or raw text data.

4. **Integration with Pods**: ConfigMaps can be attached to Pods in two primary ways:
     - As a file placed in a volume mounted into the Pod: This method allows configuration files to be injected into the Pod's filesystem at runtime. It's commonly used for configurations like certificates or application configuration files.
     - As environment variables referenced by the Pod: ConfigMap data can be exposed to Pods as environment variables, enabling easy access to configuration values within the application code.

### Creating ConfigMaps

Creating ConfigMaps in Kubernetes can be done through various methods, each serving specific use cases and scenarios. 

1. **Create from Literal**: Use the command with the `--from-literal` flag to create a ConfigMap directly from key-value pairs provided on the command line.
    ```bash
    kubectl create configmap <name> --from-literal=<key>=<value>
    ```

2. **Create from File**: Use the command with the `--from-file` flag to create a ConfigMap from a file. It's useful for creating ConfigMaps from existing configuration files or when dealing with larger sets of configuration data.
    ```bash
    kubectl create configmap <name> --from-file=<path>
    ```

3. **Create from Definition**: We can also create ConfigMaps directly from a YAML file. This approach provides the most flexibility, allowing us to define the ConfigMap's structure and data in a separate file. It's suitable for complex configuration setups or when you need to version control the ConfigMap's definition alongside other Kubernetes resources. 
   
    Command:
    ```bash
    kubectl create -f configmap-definition.yaml
    ```

    config-map.yaml file:
    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: app-config
    data:
      APP_COLOR: blue
      APP_MODE: prod
    ```

### How to use ConfigMaps in Kubernetes Pods
Below are different ways to use ConfigMaps in Kubernetes Pods to inject configuration data into container environments:

![ConfigMaps in Pods](./images/08_01%20ConfigMaps%20in%20Pods.png)


1. **ENV from ConfigMap**: This method allows us to inject all key-value pairs from a ConfigMap as environment variables into the container.
   ```yaml
    envFrom:
    - configMapRef:
        name: app-config
    ```
   - It is achieved using the `envFrom` field with a `configMapRef` object referencing the desired ConfigMap by its name.
   - All key-value pairs present in the referenced ConfigMap will be set as environment variables within the container.

2. **Single ENV from ConfigMap**: This method allows us to inject a single key from a ConfigMap as an environment variable into the container.
   ```yaml
    env:
        - name: APP_COLOR
            valueFrom:
                configMapKeyRef:
                    name: app-config
                    key: APP_COLOR
    ```
   - It is accomplished by specifying the `name` and `key` of the ConfigMap entry from which the value will be retrieved.
   - The `valueFrom` field with `configMapKeyRef` specifies the ConfigMap name (`name`) and the specific key (`key`) whose value will be used as the environment variable.

3. **Volume from ConfigMap**: This method allows you to mount a ConfigMap as a volume into the container filesystem.
    ```yaml
    volumes:
        - name: app-config-volume
            configMap:
                name: app-config
    ```
   - It is done by defining a volume with `configMap` type and specifying the name of the ConfigMap to mount.
   - All key-value pairs from the ConfigMap will be available as files in the mounted volume, allowing the container to read configuration data directly from files.

In summary, these methods provide flexibility in how ConfigMap data can be consumed by Kubernetes Pods. Depending on the requirements of the application, you can choose the appropriate method to inject configuration data as environment variables or mount them as files into the container.




### Exercise 7: ConfigMaps
Create a ConfigMap to specify an environment variable in our previous POD. See `./exercise_07/configmap.yaml`.

View and describe the ConfigMaps:
```bash
kubectl get configmaps
kubectl describe configmaps <name>
```
Use ConfigMaps in our POD. See `./exercise_07/exercise_07.yaml`.

## Secrets

### Introduction
Use Secrets for sensitive information such as API keys, credentials, etc.
- Secret values are base64 encoded and automatically decoded for pods, but it's not strong encryption. You can encode it manually if needed.
  - Base64 encoding is not very secure; anyone who gains access to the POD could decode it. Therefore, in real-life scenarios, vaults are commonly used where secrets are pre-encrypted before being stored.
- Secrets can be attached to a Pod:
  - Similar to a file placed on a volume at runtime (useful for certificates).
  - As an environment variable referenced by the Pod.

### Creating a Secret

#### Option 1: From a File
Command to create a secret using Kubectl, from a file:
```bash
kubectl create secret generic mysql-pass --from-file=./username.txt --from-file=./password.txt
```

The secret is created from two files, `username.txt` and `password.txt`, which contain sensitive information.

Here's a breakdown of the command:

- `kubectl create secret generic mysql-pass`: This part of the command tells Kubernetes to create a generic secret named `mysql-pass`.
- `--from-file=./username.txt --from-file=./password.txt`: This part specifies that the content of the `username.txt` and `password.txt` files should be used to create the secret. The contents of these files will be stored securely within the created secret.

Next, we will apply the created secret to a Pod. To do so, we will use this YAML that defines a Pod named "mysql-pod" with a MySQL container that mounts persistent storage and a secret for the MySQL root password:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysql-pod
spec:
  containers:
  - image: mysql:5.6
    name: mysql
    env: # Define environment variables for the container
    - name: MYSQL_ROOT_PASSWORD # Specify the name of the environment variable
      valueFrom: # Define the source of the value
        secretKeyRef: # Use a reference to a secret key
          name: mysql-pass # Name of the secret containing the value
          key: password # Key within the secret from which to retrieve the value
    ports:
    - containerPort: 3305
      name: mysql
    volumeMounts:
    - name: mysql-persistent-storage
      mountPath: /var/lib/mysql
    - name: secret-storage
      mountPath: /etc/secretStore
  volumes:
  - name: mysql-persistent-storage
    persistentVolumeClaim: # Define a persistent volume claim to be used as storage
      claimName: mysql-pv-claim
  - name: secret-storage  # Name of the volume used for secret storage
    secret: # Define a volume that gets its content from a secret
      secretName: mysql-pass # Name of the secret containing the data to mount
```

#### Option 2: From the literal on the command line
```bash
kubectl create secret generic mysql-pass --from-literal=password=YOUR_PASS
```

And we would apply it to a Deployment in this way:
```yaml
apiVersion: apps/v1beta2 # for versions before 1.8.0 use apps/v1beta1
kind: Deployment
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
      - image: mysql:5.6
        name: mysql
        env: # Define environment variables for the container
        - name: MYSQL_ROOT_PASSWORD # Specify the name of the environment variable
          valueFrom: # Define the source of the value for the environment variable
            secretKeyRef: # Use a reference to a secret key
              name: mysql-pass # Name of the secret containing the value
              key: password # Key within the secret from which to retrieve the value
```



### Exercise 8: Secrets
**1) Create a Secret** to be used as username/password for the database in the NGINX deployment:
```bash
kubectl create secret generic secret-backend-user --from-literal=backend-username='backend-admin'
kubectl create secret generic secret-db-user --from-literal=db-username='db-admin'
```

**2) List the secrets using `kubectl`:**

```
kubectl get secrets
```
We can also list secrets from a specific namespace. To specify the namespace, we can use the `-n` flag:

```
kubectl get secrets -n <namespace>
```

**3) View the values of the secrets**:

To view the values of the secrets using `kubectl`, you can use the `kubectl get secret` command with the `-o yaml` flag to output the secret information in YAML format. This will include the base64-encoded data of each secret.

```
kubectl get secret <secret-name> -o yaml
```
Output:

![Output](./images/08_02%20Get%20secret.png)

We can also ask for a description of the secrets:
```
kubectl describe secret
```
Output:

![Describe secret](./images/08_03%20Describe%20secret.png)


**4) Use the secrets in the front3 Deployment** and view the values inside the container's bash shell.

See front3 Deployment in `./exercise_08/exercise_08.yaml`.

Apply the deployment:
```bash
kubectl apply -f exercise_08.yaml
```

To view the value inside the container bash shell:
```bash
kubectl exec <pod name> printenv
```
This will print all the environment variables. Since our secrets are set as environment variables, they will be visible too.


### How to use Secrets in Kubernetes Pods
![Secrets in Pods](./images/08_04%20Secrets%20in%20pods.png)

In the same way as with ConfigMaps, there are different ways to use Secrets in Kubernetes Pods to inject configuration data into container environments:


1. **envFrom**: This method allows us to inject all key-value pairs from a Secret as environment variables into the container.

2. **Single ENV**: This method allows us to inject a single key from a Secret as an environment variable into the container.

3. **Volumes**: This method allows you to mount a Secret as a volume into the container filesystem.


## Labels and selectors

### Labels

Labels, also known as tags, are key-value pairs attached to objects such as Pods. 

**Label features:**
- Arbitrary metadata.
- Attached to any API object.
- Generally represent **identity**.
- Queryable by selectors.
  - Similar to SQL 'select ... where ...'
- The only grouping mechanism:
  - Pods under a ReplicationController
  - Pods in a Service
  - Node capabilities (constraints)

### Label Selectors
**Label Selectors** are a way to express how objects are selected based on their labels. They help maintain organization within Kubernetes and aid in identifying deployed resources.


**Field Selectors vs Label Selectors:**

In Kubernetes, both Field Selectors and Label Selectors are used to filter resources, but they operate differently:

1. **Field Selectors:** are used to filter resources based on specific fields within the resource's metadata.
   - They allow you to select resources based on the values of their fields, such as name, namespace, creation timestamp, etc.
   - For example, you can use a Field Selector to filter pods based on their namespace or status phase.

2. **Label Selectors:** are used to filter resources based on the labels attached to them.
   - Labels are key-value pairs attached to Kubernetes resources, and label selectors allow you to select resources based on the presence or absence of specific labels or their values.
   - For example, you can use a Label Selector to filter pods based on the labels assigned to them, such as app=frontend or environment=production.

In summary, Field Selectors filter resources based on specific fields within the resource's metadata, while Label Selectors filter resources based on the labels attached to them.

### Exercise 9: Kubectl commands for Labels
**1) Create a label on Pod X with a value of the label 'environment' set to 'dev'.**
```bash
kubectl label pods <pod_name> environment=dev --namespace=default
```

Replace `<pod_name>` with the name of your pod. This command adds a label named 'environment' with the value 'dev' to the specified pod in the 'default' namespace.

**2) Update Pod X with a new value for the 'environment' label.**
We can use the `kubectl label` command with the `--overwrite` flag:

```bash
kubectl label pods <pod_name> environment=new_value --overwrite --namespace=default
```

Replace 'new_value' with the updated value for the 'environment' label. This command will overwrite the existing value of the 'environment' label for the specified pod in the 'default' namespace.

**3) Update all pods in the 'default' namespace with the 'status' label set to 'unhealthy'.**
```bash
kubectl label pods --all status=unhealthy --namespace=default
```

This command sets the 'status' label to 'unhealthy' for all pods in the 'default' namespace.

**4) Remove the 'environment' label from Pod X.**
```bash
kubectl label pods <pod_name> environment- --namespace=default
```

This command removes the 'environment' label from the specified pod in the 'default' namespace.

**5) View all labels of pods in the 'default' namespace.**
```bash
kubectl get pods --namespace=default --show-labels
```

This command lists all pods in the 'default' namespace along with their labels. The `--show-labels` flag ensures that the labels are displayed in the output.

**6) List all resources with a particular label.**

```bash
kubectl get <resource_type> --selector=<label_selector> --namespace=<namespace>
```

Replace:
- `<resource_type>` with the type of resource you want to list (e.g., pods, deployments, services)
- `<label_selector>` with the label selector you want to filter by (e.g., key=value)
- `<namespace>` with the namespace where the resources are located. If you want to search across all namespaces, you can omit the `--namespace` flag.

For example, to list all pods with the label 'app=nginx' in the 'default' namespace, you can use:

```bash
kubectl get pods --selector=app=nginx --namespace=default
```