# Logging and Monitoring in Kubernetes Clusters

## Monitoring in Kubernetes
- Systems need to be monitored in order to **observe** and understand what's occurring with cluster and workload
- Gives the ability to **measure** impact of changes (if it's running better or worse)
- Enables understanding **resource limits** (if nodes are running out of resources, or if there are enough resources available to run the workload without degrading performance or having an outage)
- Many monitoring solutions available, such as **Prometheus** (metric data collection) and **Grafana** (data visualization)

### Kubernetes Metrics Server
- Provides resource point-in-time performance metrics for Pods and Nodes running on a cluster
- Collects resource metrics (including CPU and RAM usage) from kubelets, exposing it via API Server
- `kubectl top pods` / `kubectl top nodes`
    `--sort-by` parameter can also be applied
    `--containers` return performance for containers inside a Pod

###### Return to [Summary](README.md)