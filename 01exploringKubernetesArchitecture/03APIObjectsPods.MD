# Exploring the Kubernetes Architecture

## Pods
- Represent one or more containers on a cluster
    - That is, one or more container-based microservice(s) that are part of the application/service
- Most basic unit of work within Kubernetes - unit of *scheduling*
    - Upon provisioning a Pod, Kubernetes' [Controllers](04APIObjectsControllers.MD) uses the information manifested on code to *schedule* Pods with the necessary resources to meet the desired state
- Ephemeral - no Pod is redeployed, always a new one is provisioned after destroy
    - Technically speaking, no state is kept or shared between a destroyed Pod and its substitute
- Atomicity - they're there, or not
    - Technically speaking, if a container within a (multicontainer-)Pod dies, the whole Pod dies right away
- Kubernetes tracks the state of Pods and health of its containers to ensure that Pods are running and the app within it is also OK
    - Health is checked by **probes** (ex.: a probe in a web app Pod would check the URL and see how the app responds)

## Return to [Summary](01exploringKubernetesArchitecture/README.md)