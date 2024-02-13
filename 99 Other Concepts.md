# Other Concepts

## Commands: Field Selectors

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

  
## Service


## Ingress controller


## Special Pod

