
There are two types of Pods.

- Standalone Pod (manual pod) - If you create a Pod from a manifest (YAML file) and it is not part of a controller (Deployment, ReplicaSet, etc.), deleting the Pod will not result in it being recreated. This is a standard, standalone (yet API server-managed) Pod—not a static Pod, and not a controller-managed (self-healing) Pod.
- Static Pod: Static Pods are managed directly by the kubelet on a specific node, not by the Kubernetes API server or controller manager.
- Static Pods are defined by placing a Pod manifest file on the node in a specific directory (like `/etc/kubernetes/manifests`); the kubelet watches this directory, not the API server
- Controller Manager Pod - HA and Fault Tolerence

There are two controller in K8s
- ReplicaSet
- Deployment
