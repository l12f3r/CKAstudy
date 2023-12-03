# Configuring and Managing Application Access with Services

## Service discovery

### DNS
- `ClusterIP`s, `NodePort`s snd `Load Balancer`s will get an `A` (when IPv4) or `AAAA` (when IPv6) DNS record in ClusterDNS
    - Format: `<svcname>.<ns>.svc.<clusterdomain>`
        - Example: `hello-world.default.svc.cluster.local`: `hello-world` is the Service name, `default` is the namespace, and `cluster.local` is the cluster domain
- When a namespace is created, a DNS subdomain is created for it (`<ns>.svc.<clusterdomain>`)

#### Environment variables
- Defined in Pods for each Service available at Pod startup
    - To query for it, a Pod that was just provisioned must be terminated (so the Service Pods will be at startup)

###### Return to [Summary](README.md)