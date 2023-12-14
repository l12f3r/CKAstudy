# Logging and Monitoring in Kubernetes Clusters

## Accessing object data with JSONPath
- API requests can have their outputs as JSONPath, allowing to write expressions to access, filter, sort and format object data
    - Inlining expressions to control the output from kubectl before writing to console
    - Enables to perform precise operations on the data in the object
- In order to compose a correct JSONPath command, it is recommended to previously check the JSON of the resource by outputting it
    - Appending `-o json > file.json` to the command would do the trick
- `kubectl get pods -o jsonpath='{.items[*].metadata.name}{"\n"}'`, for instance, would list all Pods, but displaying just their names
    - `-o jsonpath` defines the output type to be returned, in this case, JSONPath
    - `.items[*]` is a list of objects returned from the API Server, with the * wildcard representing an operation on all items in that list
        - To return a specific item from the list, enter the numeric index representing its position between the brackets (`.items[0]` would return the first item, for instance)
    - `.metadata.name` are the data fields accessed for every object in `items`
    - `{"\n"}` adds a line break, to separate results in different lines
- `kubectl get pods --all-namespaces -o jsonpath='{.items[*].spec.containers[*].image}'` returns all container images in use by all Pods in all namespaces

## Filtering objects with JSONPath
- `kubectl get nodes -o jsonpath="{.items[*].status.addresses[?(@.type=='InternalIP')].address}" --sort.by=.metadata.name` returns a list of all internal IP addresses of nodes in a cluster
    - For each node returned (`.items[*]`), access the `.status` field,
    - which has an `.addresses` list defined in it
    - The `?()` represents a filter expression, with the expression content within the parenthesis
    - `@` represents filtering on the current object
    - For every object on that list, this command tests to see if the `.type` field is equal (`==`) to the `InternalIP` string
    - After closing the parenthesis and brackets, this command accesses the `.address` field
    - Object data is properly sorted by using the `--sort-by` parameter

###### Return to [Summary](README.md)