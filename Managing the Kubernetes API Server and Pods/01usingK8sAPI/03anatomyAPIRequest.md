# Using the Kubernetes API

## Anatomy of an API request

1. A command is entered in `kubectl`, which converts this request from YAML into JSON, and submits it to the API Server (depending on the request)
    - Commands are based on RESTful actions - `GET`, `POST`, `DELETE`
2. An API resource location (or API path) must be specified
    - The [API group](02APIGroupsVersioning.md) is encoded in this path, along with information if it is Core or Named, the version of the API to work with, and the object type
3. If the request is valid, the API Server commits it to etcd with the `WRITE` operation and responds with the proper HTTP response code (200, 404 etc)
    - It the request is a query, the information is `READ` from etcd and the proper HTTP response code is replied

## RESTful API verbs

| Verb | Description |
| --- | --- |
| `GET`  | Gets data for a specified resource  | 
| `POST`  | Creates a resource |
| `DELETE`  | Deletes a resource |
| `PUT`  | Creates/updates entire existing resource |
| `PATCH`  | Modifies the specified fields of a resource  |
| `LOG`  | Restrieves logs from a container in a Pod  |
| `EXEC` | Runs (executes) a command in a container  |
| `WATCH` | Listens to changes notifications on a resource with streaming output[^1]  |

[^1]: Each resource in Kubernetes has a `resourceVersion` associated with it. When a `WATCH` is requested on an object or resource, notifications will be sent to the client on every `resourceVersion` change from that request onwards.

## API resource location (API Paths)

- Split across two API groups: 
    - Core API group (legacy), like the following
        - `http://apiserver:port/api/$VERSION/$RESOURCE_TYPE`
        - namespace: `http://apiserver:port/api/$VERSION/namespaces/$NAMESPACE/$RESOURCE_TYPE/$RESOURCE_NAME`
    - and Named API group
        - `http://apserver:port/apis/$GROUPNAME/$VERSION/$RESOURCE_TYPE`
        - `http://apiserver:port/apis/$GROUPNAME/$VERSION/namespaces/$NAMESPACE/$RESOURCE_TYPE/$RESOURCE_NAME`

## Response Codes from the API Server

| Success (2xx) | Client Errors (4xx) | Server Errors (5xx)
| --- | --- | --- |
| 200 - OK | 401 - Unauthorized | 500 - Internal Server Error |
| 201 - Created | 403 - Access Denied | --- |
| 202 - Accepted | 404 - Not Found | --- |

## A closer look

| **CLIENT REQUEST** | Connection -> | Authentication -> | Authorization -> | Admission Control | **SERVER RESPONSE** |
| --- | --- | --- | --- | --- | --- |
| --- | First step: identify if a connection to the API Server is possible | Determine if it's a valid user | Can this user perform the requested action? | Admin control over request | --- |
| --- | If possible, it'll be HTTP over TCP | Validation via authentication plugin | Can the user execute the API verb on the resource/API path? | Additional code [^3] | --- |
| --- | More commonly, TLS encrypted (HTTPS) | Modular[^2] | Actions are denied by default | May modify object | --- |
| --- | --- | If fails, gets 401 response code | If fails, gets 403 response code | Validation | --- |

[^2]: A modular plugin is used to authenticate user via various authentication types, like certificates, passwords, tokens etc
[^3]: An admission controller is a piece of code that intercepts requests to the API server prior to persisting that object to etcd

###### Return to [Summary](README.md)