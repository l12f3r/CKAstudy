# Exploring the Kubernetes Architecture

## Services
- Network abstraction for access to Pods and its provided services
    - Persistently allocates IP addresses and a DNS names for services provided by Pods front-ending the application
        - Adds persistency
    - Leverages services by adding/removing Pods depending on demand/health
    - Provides a load balancer to distribute application load across Pods