# Using the Kubernetes API

## Anatomy of an API request

1. A command is entered in `kubectl`, which converts this request from YAML into JSON, and submits it to the API Server (depending on the request)
    - Commands are based on RESTful actions - `GET`, `POST`, `DELETE`
2. A URL resource location or API path must be specified
    - The [API group](02APIGroupsVersioning.md) is encoded in this path, along with information if it is Core or Named, the version of the API to work with, and the object type
3. If the request is valid, the API Server commits it to etcd with the `WRITE` operation and responds with the proper HTTP response code (200, 404 etc)
    - It the request is a query, the information is `READ` from etcd and the proper HTTP response code is replied

## RESTful API verbs

| ------------- | ------------- |
| `GET`  | Gets data for a specified resource  | 
| `POST`  | Creates a resource |
| `DELETE`  | Deletes a resource |
| `PUT`  | Creates/updates entire existing resource |
| `PATCH`  | Modifies the specified fields of a resource  |
| `LOG`  | Restrieves logs from a container in a Pod  |
| `EXEC` | Runs (executes) a command in a container  |
| `WATCH` | Listens to changes notifications on a resource with streaming output[^1]  |

[^1]: Each resource in Kubernetes has a `resourceVersion` associated with it. When a `WATCH` is requested on an object or resource, notifications will be sent to the client on every `resourceVersion` change from that request onwards.

###### Return to [Summary](README.md)