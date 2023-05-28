- Namespaces provides a mechanism for isolating groups of resources within a single cluster. 
- Names of resources need to be unique within a namespace, but not across namespaces.
- Namespace-based scoping is applicable only for namespaced [objects](https://kubernetes.io/docs/concepts/overview/working-with-objects/#kubernetes-objects) _(e.g. Deployments, Services, etc)_ and not for cluster-wide objects _(e.g. StorageClass, Nodes, PersistentVolumes, etc)_.

### When to use 
- Namespaces are intended for use to provide logical grouping of resource.
- It provide a way to divide cluster resources between multiple users (via [resource quota](https://kubernetes.io/docs/concepts/policy/resource-quotas/)).
- It can be used with-in the network policy to manage the network access.

### Namespaces and DNS

- When you create a Service, Kubernetes creates DNS entry for it. It means's, You can contact Services with consistent DNS names instead of IP addresses. 
- This DNS entry is of the form 
	`<service-name>.<namespace-name>.svc.cluster.local
- If a container only uses `<service-name>`, it will resolve to the service which is local to a namespace.  By default, a client Pod's DNS search list includes the Pod's own namespace and the cluster's default domain.

