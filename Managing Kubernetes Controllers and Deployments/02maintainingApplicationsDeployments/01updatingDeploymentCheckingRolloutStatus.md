# Maintaining applications with Deployments

## Updating a Deployment
- [Deployments](../01usingControllersDeploymentBasics/02deployment.md) can be updated:
    - By rolling out new versions of a container image on Pods
        - Imperatively: `kubectl set image deployment hello-world hello-world=hello-app:2.0` changes the `hello-world=` container image of the `hello-world` `Deployment` to version `hello-app:2.0`
            - The `--record` flag can be passed to capture metadata information about the operation executed. Such information is stored in a CHANGE-CAUSE annotation associated with the `Deployment` object
    - Triggered by changing the Pod `template`
        - Declaratively: `kubectl edit deployment hello-world` opens the YAML code for the `hello-world`, allowing to edit the Pod `template`. To apply such changes, `kubectl apply -f hello-world-deployment.yaml --record`
        - Other fields on the deployment `spec` (such as number of replicas) can be changed without triggering a rollout/update

### Controller operations on Deployment updates (at the cluster level...)
1. Upon execution, a `Deployment` creates a [`ReplicaSet`](../01usingControllersDeploymentBasics/03replicaSet.md) with matching labels/Selectors (**R1**, for instance)
2. `ReplicaSet` creates Pods for those workloads as defined on desired state, with matching labels/Selectors (**R1**)
3. To update an application, a change on Pod's `template` excerpt must be in place (let's say, changing the previous label/Selector to **R2**). Such change triggers Kubernetes to start a new version of the `ReplicaSet`
4. This new `ReplicaSet` (under **R2**), with a new [`pod-template-hash`](https://github.com/l12f3r/CKAstudy/blob/main/Managing%20the%20Kubernetes%20API%20Server%20and%20Pods/02managingObjectsLabelsAnnotationsNamespaces/02workingWithLabels.md#controller-operations-deployments), creates new Pods (also under the new rule, **R2**), which will transition into  receiving the workload from **R1**
5. The **R1** version of the `ReplicaSet` is kept (in case of a necessary rollback), but its Pods are terminated

## Checking Deployment rollout status 
- `kubectl rollout status deployment [name]`: interactive view of rollout status on the CLI, outputting to stdout new and terminating Pods
- `kubectl describe deployment [name]`: deep-dive on metadata, including:
    - Deployment statuses: **Complete** (all update work is finished), **Progressing** (update in course), **Failed** (update could not be completed)

###### Return to [Summary](README.md)