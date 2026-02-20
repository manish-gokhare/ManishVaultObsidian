This service used to expose the pod internally as well as externally.

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
  name: my-loadbalancer-svc
spec:
  selector:
    app: my-app
  ports:
  - port: 801  # Svc listen on port 801
    targetPort: 80  # container port 80
  type: LoadBalancer
```
	
- Endpoint of the service is going to be PodIP:targetPort.

- targetPort should be match with containerPort of the Pod to expose via service.
- When we create LoadBalancer service, NetworkLoad will be created TCP LB.
- this LB will have external IP. LB sends the traffic to worker node
- You need to configure LB in front of a NodePort service, and it becomes an entry point to your Kubernetes cluster that forwards traffic to your pods.
- Traffic arrives at the external load balancer’s IP and is forwarded to the nodes in the cluster on a NodePort assigned to the Service. From the node on that NodePort, traffic is directed via kube-proxy through iptables rules (or IPVS) to the ClusterIP Service inside the node. The ClusterIP Service then load balances traffic across the Pods selected by the Service selectors, which may be running on any node.


```
manish@MacBook-Pro-va-FOX k8s % kubectl get svc
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)         AGE
kubernetes   ClusterIP      10.96.0.1       <none>          443/TCP         117d
my-lb-svc    LoadBalancer   10.97.247.223   **10.97.247.223**   801:30598/TCP   21h

manish@MacBook-Pro-va-FOX k8s % kubectl get pod
NAME           READY   STATUS    RESTARTS   AGE
httpd-lb-pod   1/1     Running   0          21h

manish@MacBook-Pro-va-FOX k8s % kubectl get ep my-lb-svc
NAME        ENDPOINTS        AGE
my-lb-svc   10.244.0.99:80   21h
```

How to access the pod using LB?
We need to access using external IP:servicePort
http://10.97.247.223:801

Traffic Flow:
external IP : ServicePort (Port) --> NodeIP:NodePort --> ServiceIP:ServicePort (Port)  --> PodIP:TargetPort

The traffic flow for a Kubernetes LoadBalancer Service follows this path:
	1.	External client sends traffic to the external IP assigned to the LoadBalancer on the ServicePort (the port specified in the Service’s `spec.ports[].port`).
	2.	The cloud load balancer forwards this traffic to one of the cluster Node IPs at the NodePort assigned to the Service (`spec.ports[].nodePort`).
	3.	On the node, kube-proxy routes the traffic arriving at the NodePort to the ClusterIP of the Service.
	4.	The ClusterIP Service load balances the traffic among the Pods selected by the Service label selector.
	5.	Traffic reaches the Pod at its TargetPort configured in the Service (`spec.ports[].targetPort`).

So the flow is:

**ExternalIP:ServicePort → NodeIP:NodePort → ClusterIP:ServicePort → PodIP:TargetPort**

This means the traffic goes through NodePort and ClusterIP within the cluster before reaching the Pod. The LoadBalancer Service automatically manages this routing setup by provisioning the external load balancer and configuring nodes and Services accordingly.


```
manish@MacBook-Pro-va-FOX k8s % kubectl get service my-lb-svc -o jsonpath='{.spec.ports[0].port}'
801

manish@MacBook-Pro-va-FOX k8s % kubectl get service my-lb-svc -o jsonpath='{.spec.ports[0].nodePort}'
30598
```

How to delete all the pod?

```
kubectl delete pod
```