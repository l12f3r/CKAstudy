# Configuration as Data - Environment Variables, Secrets and ConfigMaps

## ConfigMaps
- API object that consists of key/value pairs exposed into a Pod
    - Used to configure applications/environment specific settings (daemons running inside of containers like web servers, caching servers, connection strings, URLs, hostnames, credentials etc)
- Aims to decouple application and Pod configurations
    - Maximizes container image's portability
- Can be exposed into the container as environment variables or files
- Must exist before Pod creation, or the container referencing the ConfigMap won't start up (it'll keep a `Pending` status)

### Defining ConfigMaps
- Imperatively:
    - `kubectl create configmap appconfigprod --from-literal=DATABASE_SERVERNAME=sql.example.local --from-literal=BACKEND_SERVERNAME=be.example.local` would create the ConfigMap straight from a literal key/value
    - `kubectl create configmap appconfigqa --from-file=appconfigqa` would create the ConfigMap from a file
- Declaratively:
```
apiVersion: v1
kind: ConfigMap                                 # where the ConfigMap is defined
metadata:
  name: appconfigprod
data:                                           # data is defined in key/value pairs
  BACKEND_SERVERNAME: be.example.local
  DATABASE_SERVERNAME: sql.example.local
```

### Using ConfigMaps in Pods
- Using **Environment Variables**
    - `valueFrom` to read an explicit key defined inside of the ConfigMap
    - `envFrom` to read the entire ConfigMap and all of its keys, creating environment variables for each one
```
...
spec:                                   # the Pod's spec
  containers:
  - name: hello-world
  ...
  env:
  - name: DATABASE_SERVERNAME
    valueFrom:
      configMapKeyRef:                  # reference from where it gets its value from
        name: appconfigprod             # must match to the ConfigMap's metadata.name
        key: DATABASE_SERVERNAME
  - name: BACKEND_SERVERNAME
    valueFrom:
      configMapKeyRef:
        name: appconfigprod             # must match to the ConfigMap's metadata.name
        key: BACKEND_SERVERNAME
```
```
spec:
  containers:
  - name: hello-world
  ...
  envFrom:                              # shorter way of declaring
    - configMapRef:
        name: appconfigprod
```
- Using **Volumes and Files**
    - Information contained is mounted using tmpfs
    - Can be single or many files or directories, depending on the structure of the ConfigMap and number of ConfigMaps referred to inside of the Pod's spec
    - Volume ConfigMaps can be updated - environment variables, no
```
...
spec:                                   # the Pod's spec
  volumes:
    - name: appconfig
      configMap:                        # volume type
        name: appconfigqa               # ConfigMap selected to read from
  containers:
  - name: hello-world
    ...
    volumeMounts:                       # where the volume specified above is referenced
      - name: appconfig
        mountPath: "/etc/appconfig"     # location where it should be mounted
```
- Can be marked as immutable

###### Return to [Summary](README.md)