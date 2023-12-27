# Kubernetes Security Fundamentals

## Securing the API Server

#### Steps to take before a request is accepted and processed by the API Server
1. **Authentication** 👤: validates the identity of who/what submitted the request
    - Different methods, such as certificates or basic HTTP authentication
2. **Authorization** 📋: determines if the requester is allowed to perform the action on the request submitted 
    - Role-based Access Control (RBAC) 🧢
3. **Admission Control** ✅: enables administrative control over API requests 
    - Has admission controllers intercepting requests prior to persisting the object on the cluster
    - Configuration of additional validation, modifying incoming objects
    - Resource limits on Pods

## API Server - Authentication
- Each API request needs to be authenticated upon coming into the API Server
- Requests come from various sources (**Users** enter via kubectl, **Service Accounts** from applications running inside Pods, internally from **Control Plane 🧠 services** like the Scheduler and Controller Manager or even from kubelets/kube-proxys of other nodes)

## Authentication plugins
| Client Certificates  | Authentication Tokens | Basic HTTP | OpenID Connect |
| ------------- | ------------- | ------------- | ------------- |
| Most commonly used, either on cloud or not | Encoded information in the authorization header of an HTTP request | User account information and passwords stored on a static file | Enables external ID/auth providers/services |
| Default when building cluster via kubeadm | Used mostly in Service Accounts, Bootstrap Tokens and Static Token Files | -------------  | ------------- |
| Common Name (CN) is the username | Administration can be challenging[^2] | Also are read only on API Server startup | Requires additional setup integration with the external provider/service |
| Adds administrative/maintenance burden[^1] | ------------- | Simpler to set up and use on development clusters | Centralized user database, facilitates SSO |

[^1]: Certificates expire, and username changes require reissue.
[^2]: Static token files' contents are only read when the API Server starts up; changes require restarting the API Server

## Authorization plugins
| Role-based Access Control (RBAC)[^1] 🧢  | Node [^1] | Attribute-based Access Control (ABAC) |
| ------------- | ------------- | ------------- |
| Access to resources is controlled based on roles granted to individual users and functions such roles can perform | Used to grant API access and permissions to kubelets on nodes | Access rights are granted to users through policies + attributes |
| Role with functions is created, then bound to user | Restricts node access to resources that kubelets need to access | Enables granular access control based on users, resources and environment  |

[^1]: Default authorization modules on kubeadm-based clusters

###### Return to [Summary](README.md)