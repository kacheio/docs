# Cluster

Kache supports the synchronization of services in a cluster environment (e.g. Kubernetes).

A typical use case is invalidating in-memory cache keys when Kache is running in a cluster with 
replicated services and with "multi-tier" ([`layered`](./provider.md#layered-cache)) caching enabled. 
Thus, when a pod receives a PURGE request, it performs the purge operation and sends a signal to
all the other pods in the cluster to invalidate the corresponding keys in their local in-memory 
caches as well.

=== "YAML"
  ``` yaml
  cluster:
    # Cluster provider (kubernetes).
    discovery: kubernetes
    # Kubernetes namespace where the cluster is running.
    namespace: default
    # Service name as specified in the service configuration.
    service: kache-service
  ```

### Reference

| Directive     | Type        | Description                          |
| -----------   | ----------- | ------------------------------------ |
| `discovery`   | `string`    | The cluster provider in which endpoints and services can be discovered. At the moment only `kubernetes` is supported. |
| `namespace`   | `string`    | The Kubernetes namespace where the cluster is running. |
| `service`     | `string`    | The service name as specified in the service config. |
