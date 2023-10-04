# Managing Objects with Labels, Annotations and Namespaces

## Annotations

- Used to add extra information about cluster resources directly to its metadata
- Most used (by either people or tooling) to make decisions about what to do with a particular resource based on the Annotation
- Build, release and image information exposed in easily accessible areas
- Saves from having to write integrations to retrieve data from external data sources
- Non-hierarchical, key/value pair
- Can't be used to query/select Pods or other resources

## Adding and Editing Annotations

### Declaratively

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  annotation: owner=Anthony        # Declaration of an Annotation and its key/value pair
spec:
  containers:
  - name: nginx
```

### Imperatively

- `kubectl annotate pod nginx-pod owner=Anthony` to add the `owner=Anthony` annotation to the exising `nginx-pod` Pod
- `kubectl annotate pod nginx-pod owner=NotAnthony --overwrite` to edit it

###### Return to [Summary](README.md)