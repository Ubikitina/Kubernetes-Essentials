# Configurations
Table of contents:
- [Configurations](#configurations)
  - [Commands and Arguments](#commands-and-arguments)
  - [Environment variables](#environment-variables)
    - [Exercise 6: Environment variables](#exercise-6-environment-variables)
  - [Security Context](#security-context)
    - [Security Contenxt in Docker](#security-contenxt-in-docker)
    - [Security Context in Kubernetes: Pod Level](#security-context-in-kubernetes-pod-level)


## Commands and Arguments
Having this Dockerfile:
```dockerfile
FROM Ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]
```
The `ENTRYPOINT` instruction sets the primary command to be executed when the container starts. Here, it's set to "sleep".
The `CMD` instruction provides default arguments to the entrypoint command. In this case, it specifies "5" as the default argument.


And also, having this `pod-definition.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
  - name: ubuntu-sleeper
    image: ubuntu-sleeper
```

We can include the commands and arguments in the YAML. To include the commands and arguments from the Dockerfile in the YAML, we can use the `commands` and `args` fields within the container specification.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
  - name: ubuntu-sleeper
    image: ubuntu-sleeper
    commands: ["sleep 2.0"]
    args: ["10"]
```

After updating the pod definition YAML with the desired commands and arguments, we can create the pod using the following command:
```bash
kubectl create -f pod-definition.yaml
```

Expected outcome:
- When the pod is created, it will start a single container using the specified image ("ubuntu-sleeper").
- The container will execute the command "sleep 2.0" with the argument "10". This means the container will sleep for 10 seconds, as **overridden** by the arguments provided in the pod definition YAML.
- The container will then exit after sleeping for the specified duration.


## Environment variables
Define the variables in pod spec using the `env` section. 

For example, if we want to include these variables:
```bash
docker run -e APP_COLOR=pink simple-webapp-color
```

We can create the following `pod-definition.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
    - containerPort: 8080
    env:
    - name: APP_COLOR
      value:pink
```



### Exercise 6: Environment variables

- Create environment variables named DEMO_GREETING and DEMO_FAREWELL with test values from a file for the image `gcr.io/google-samples/node-hello:1.0`.

- See the environment variables using the command:
  ```bash
  kubectl exec <pod> -- printenv
  ```

- Use the environment variables with the `bash` image using the `echo` command and as arguments the variables we are going to create.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: envar-dhack2
spec:
  containers:
  - name: env-print-dh
    image: bash
    command: ["echo"]
    args:
      - "DEMO_GREETING: $(echo $DEMO_GREETING)"
      - "DEMO_FAREWELL: $(echo $DEMO_FAREWELL)"
    env:
      - name: DEMO_GREETING
        value: "test_greeting_value"
      - name: DEMO_FAREWELL
        value: "test_farewell_value"
```

Now let's play with the variables to display information from our POD and containers:
https://kubernetes.io/docs/tasks/inject-data-application/environment-variable-expose-pod-information/

## Security Context

### Security Contenxt in Docker
In Docker, use another user, not Root. Using a non-root user instead of the root user in Docker improves security by reducing the potential impact of security vulnerabilities or malicious attacks. It is a security best practice (Least Privilege Principle) that helps reduce attack surface, minimize the impact of security incidents and reduces the risk of compromise to both the containerized application and the host system (Container Escape Prevention). Here's why:

**How to check the users in Linux:**

When you run ps aux, you get a comprehensive list of all processes running on the system, along with detailed information about each process, regardless of whether they are associated with your current terminal session or not.


![ps aux](./images/07_01%20ps%20aux.png)

**How to set the user and/or permissions in `docker run`:**
```bash
docker run --user=1001 ubuntu sleep 3600
```
   - This command runs a Docker container using the `ubuntu` image.
   - The `--user=1001` flag sets the user ID inside the container to `1001`.
   - `sleep 3600` is the command that the container will run. In this case, it's the `sleep` command, which pauses execution for the specified number of seconds (3600 seconds or 1 hour).
  
```bash
docker run --cap-add MAC_ADMIN ubuntu
```
   - This command runs a Docker container using the `ubuntu` image.
   - The `--cap-add MAC_ADMIN` flag adds the `MAC_ADMIN` capability to the container.
   - Capabilities are permissions that control what a process can do. Adding the `MAC_ADMIN` capability allows the container to perform certain networking-related tasks that are typically restricted.



### Security Context in Kubernetes: Pod Level

We can include the `securityContext` settings in the YAML configuration:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    # Sets the user ID under which the container should run.
    runAsUser: 1000
    # Sets the group ID for the file system group of the container.
    fsGroup: 2000
  volumes:
    - name: sec-ctx-vol
      # Defines an emptyDir volume, which is a temporary directory that shares its data across containers in the same Pod.
      emptyDir: {}
  containers:
    - name: sec-ctx-demo
      image: ignite.azurecr.io/nginx-demo
      volumeMounts:
        - name: sec-ctx-vol
          mountPath: /data/demo
      securityContext:
        # Sets the user ID under which the container should run. This overrides the Pod-level setting.
        runAsUser: 2000
        # Specifies whether the container is allowed to escalate privileges. In this case, privilege escalation is disabled.
        allowPrivilegeEscalation: false
        # Specifies additional capabilities to add to the container. Here, NET_ADMIN and SYS_TIME capabilities are added.
        capabilities:
          add: ["NET_ADMIN", "SYS_TIME"]
        # Specifies SELinux security options.
        seLinuxOptions:
          # Sets the SELinux MLS/MCS level.
          level: "s0:c123,c456"
```

Explanation of `securityContext` settings:

1. **Pod-level `securityContext`**:
   - `runAsUser`: Sets the user ID under which all containers in the Pod should run to `1000`.
   - `fsGroup`: Sets the group ID for the file system group of all containers in the Pod to `2000`.

2. **Container-level `securityContext`** (inside the `containers` field):
   - `runAsUser`: Overrides the Pod-level setting and sets the user ID for the specific container to `2000`.
   - `allowPrivilegeEscalation`: Specifies whether the container is allowed to escalate privileges. In this case, it's set to `false`, meaning privilege escalation is disabled.
   - `capabilities`: Specifies additional Linux kernel capabilities to add to the container. Here, `NET_ADMIN` and `SYS_TIME` capabilities are added.
     - `NET_ADMIN`: This capability allows a process to perform various network-related operations that are typically restricted to privileged users.
     - `SYS_TIME`: This capability allows a process to set the system clock or modify its behavior.
   - `seLinuxOptions`: Specifies SELinux security options. Here, the SELinux Multi-Level Security (MLS) or Multi-Category Security (MCS) level is set to `"s0:c123,c456"`.
     - `s0`: This is the SELinux sensitivity level. It represents the sensitivity of the data handled by the process or container. In this case, s0 indicates the default sensitivity level.
     - `c123,c456`: These are the SELinux categories. They represent different security categories or compartments that data can be classified into.

