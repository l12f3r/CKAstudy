# Configuration as Data - Environment Variables, Secrets and ConfigMaps

## Secrets
- API objects that allow to store sensitive information as objects for retrieval (for later use)
    - Passwords, API tokens, keys and certificates
- Allows safer and more flexible configurations (than Pod's specs and container images), so such data is not stored along with those resources

### Properties
- base64 encoded (not encrypted - that's the default setting)
    - Encryption can be configured, if required
- Information is stored inside of etcd
    - Backups and transport around this information needs to be secured
- Namespaced
- Can only be referenced by Pods in the same Namespace
- Unavailable Secrets prevents Pods from starting up 
    - The same occur for Secrets in a different Namespace from the Pod
    - If a Secret is referenced inside of a Pod but is unavailable, the Pod will be on a `Pending` status until the Secret is available

### Creating Secrets (imperatively)
- `kubectl create secret generic app1 --from-literal=USERNAME=app1login --from-literal=PASSWORD='S0methingS@Strong!'`
    - Three different types of Secrets: `docker-registry`, `generic` and `tls`
        - `generic` sources data from a local directory, file or literal value specified on the CLI
            - Literal key/value defined in `--from-literal`
            - File would be defined as `--from-file`
            - Directory would be defined as `--from-directory`

### Using Secrets in Pods
- Use as **Environment Variables**: a Secret can be stored into an Environment Variable, being exposed to applications
```
...
spec:                               # the Pod's spec
  containers:                       # each container should have its own Environment Variables and Secrets set
  - name: hello-world
    ...
    env:
    - name: app1username
      valueFrom:                    # source for the Environment Variable            
        secretKeyRef:               # Secret key reference
          name: app1
          key: USERNAME
    - name: app1password
      valueFrom:                    # another Environment Variable source
        secretKeyRef:
          name: app1
          key: PASSWORD
```
```
...
spec:
  containers:
  - name: hello-world
    ...
    envFrom:                        # shorter way of declaring
    - secretRef:
        name: app1
```
- Use as **Volumes or Files**: tmpfs mounted files exposed into the filesystem inside of the container
    - Volumes or Files can be updated - environment variables, no
```
...
spec:
  volumes:                                  # volume declaration
    - name: appconfig
      secret:                               # volume type
        secretName: app1
  containers:
    ...
    volumeMounts:                           # where the volume will be exposed
      - name: appconfig
    mountPath: "/etc/appconfig"             # location where it should be mounted
```
- Secrets can be mark as **Immutable** - that is, it cannot be changed after its creation

## Accessing a private container registry
- Sometimes, Secrets (in for of credential information) must be used to access a private container registry that requires authentication
    - Common scenario for container images not publicly available, but accessible over the internet
- Container registries: Docker Hub, or any cloud-based container registries (Azure Container Registry, AWS ECR etc)
- `docker-registry` Secret type
- `ctr` commands must be used to download container images - such CLI is part of containerd package

###### Return to [Summary](README.md)