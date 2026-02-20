A NodePort service in Kubernetes allows a service to be accessed both internally within the cluster and externally from outside the cluster.

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: httpd-pod-with-label
  namespace: default
  labels:
    app: my-app
    environment: development
spec:
  containers:
  - name: httpd
    image: httpd:latest
    ports:
    - containerPort: 80
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nodeport-svc
spec:
  selector:
    app: my-app
  ports:
  - port: 801  # Service listens on port 801 inside the cluster
    targetPort: 80  # Pod container port 80
    nodePort: 31284   # Static NodePort exposed on each node
  type: NodePort
```

- A NodePort service in Kubernetes allows a service to be accessed both internally within the cluster and externally from outside the cluster.
- When a service’s type is set to NodePort, Kubernetes assigns a port number from the default range 30000–32767. This is the standard port range for NodePort services. The port is selected dynamically if not specified manually.
- In this example, the assigned NodePort is 31284. This port is used to expose the service on each node (master or worker) in the cluster. Internally, the service forwards traffic to the target Pods.

In summary, with a NodePort service:
	•	Kubernetes exposes the service on a static port across all nodes.
	•	External users can access the service via  '<NodeIP>:<NodePort>`.
	•	The service then routes the request to the appropriate Pod inside the cluster.




```
manish@MacBook-Pro-va-FOX k8s % kubectl get service
NAME              TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)         AGE
kubernetes        ClusterIP   10.96.0.1      <none>        443/TCP         115d
my-nodeport-svc   NodePort    10.105.76.26   <none>        801:31284/TCP   8m53s

manish@MacBook-Pro-va-FOX k8s % kubectl get ep my-nodepoer-svc
NAME              ENDPOINTS        AGE
my-nodepoer-svc   10.244.0.97:80   9m4s
```

```
manish@MacBook-Pro-va-FOX k8s % kubectl get pod -o wide
NAME                   READY   STATUS    RESTARTS   AGE     IP            NODE       NOMINATED NODE   READINESS GATES

httpd-pod-with-label   1/1     Running   0          4m11s   10.244.0.97   minikube   <none>           <none>
nginx-pod              1/1     Running   0          86m     10.244.0.93   minikube   <none>           <none>

manish@MacBook-Pro-va-FOX k8s % kubectl get nodes
NAME       STATUS   ROLES           AGE    VERSION
minikube   Ready    control-plane   115d   v1.32.0

manish@MacBook-Pro-va-FOX k8s % kubectl get nodes -o wide

NAME       STATUS   ROLES           AGE    VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE              KERNEL-VERSION   CONTAINER-RUNTIME

minikube   Ready    control-plane   115d   v1.32.0   192.168.64.3   <none>        Buildroot 2023.02.9   5.10.207         docker://27.4.0



manish@MacBook-Pro-va-FOX k8s % minikube ip

192.168.64.3
```

How NodePort Works

- The Service gets an internal ClusterIP for communication inside the cluster.
 - Additionally, Kubernetes opens the NodePort on every node (master or worker) in the cluster.
-  External clients access the service by sending requests to any node’s IP address at the assigned NodePort. Your example URL `http://192.168.64.3:31284` correctly illustrates this.
- Traffic received on the NodePort is forwarded to the Service, which listens on the specified service port (in your example, port 801). This service port routes traffic to the Pod(s) on their target ports.
- The NodePort is a static port on every node, ensuring consistent access through any node IP in the cluster.

The traffic flow is from NodePort → Service port → Pod target port.

nodeip:NodePort --> serviceIP:Serviceport (port) --> PodIP:targetPort
192.168.64.3:31284 --> 10.105.76.26:801 --> 10.244.0.97:80


- Traffic enters the cluster via the Node IP and the assigned NodePort (e.g., 192.168.64.3:31284).
- This traffic is then forwarded to the internal Service IP at the Service port (e.g., 10.105.76.26:801).
- Finally, the Service routes the traffic to the Pod IP at the Pod’s target port (e.g., 10.244.0.97:80).

![[Pasted image 20251117212507.png]]


It can also be accessible internally .

```
manish@MacBook-Pro-va-FOX k8s % kubectl exec -it nginx-pod -- /bin/bash  

root@nginx-pod:/# curl my-nodepoer-svc:801

<html><body><h1>It works!</h1></body></html>
```

