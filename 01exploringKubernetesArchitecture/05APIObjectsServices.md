# Exploring the Kubernetes Architecture

## Services
- Network abstraction for access to [Pods](03APIObjectsPods.MD) and its provided services
    - Persistently allocates IP addresses and a DNS names for services provided by Pods front-ending the application
        - Adds persistency
    - Leverages services by adding/removing [Pods](03APIObjectsPods.MD) depending on demand/health
    - Provides a load balancer to distribute application load across [Pods](03APIObjectsPods.MD)

- Services identifies which ephemeral [Pods](03APIObjectsPods.MD) are served using labels

###### Return to [Summary](https://github.com/l12f3r/CKAstudy/tree/main/01exploringKubernetesArchitecture#readme)