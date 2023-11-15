# Configuration as Data - Environment Variables, Secrets and ConfigMaps

## Configuring Applications in Pods
- **Command Line arguments**
    - Used when starting up a container, to change functionality of an application
- **Environment Variables**
    - During runtime, an application inside of an container can read any of the environment variables and change how it behaves based on the environment variable values
- **ConfigMaps**
    - API object to inject more complicated configurations

## Environment variables inside Pods
- Two types of environment variables:
    - **User-defined**:
        - Specified on the Pod's spec for each container, either by name-value pairs or `valueFrom` (a ConfigMap, among others)
        - Can be defined inside the actual container image (not a good practice, though)
    - **System-defined**:
        - Names of all Services available upon creating the Pod
- Defined at container startup
- Cannot be updated once the Pod is created

```
...
spec:                                                   # the Pod's spec
  containers:
  - name: hello-world
    image: gcr.io/google-samples/hello-app:1.0
    env:                                                # environment variables definition
    - name: DATABASE_SERVERNAME
      value: "sql.example.local"
    - name: BACKEND_SERVERNAME
      value: "be.example.local"
```


###### Return to [Summary](README.md)