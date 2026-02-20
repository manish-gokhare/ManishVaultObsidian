
- Pod communicate with each other with their IP address but not with their name. 
- Name of the Pod is not resolve to IP address.
- If Pod IP changes in case of restart, communcation will not happen. Need to change the IP adress while accessing the pod.
- IP address of Pod is dynamic. There is no way to assign the static IP. So Pod communication using IP is not recommended.
- To Solve this problem we need service.


```
manish@MacBook-Pro-va-FOX k8s % kubectl get pod -o wide

NAME                   READY   STATUS    RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES
first-pod              1/1     Running   0          30m   10.244.0.84   minikube   <none>           <none>

httpd-pod-with-label   1/1     Running   0          24m   10.244.0.86   minikube   <none>           <none>
nginx-pod             1/1     Running   0          28m   10.244.0.85   minikube   <none>           <none>

# Go inside the nginx-pod and access httpd-pod-with-label using its IP (10.244.0.86)

manish@MacBook-Pro-va-FOX k8s % kubectl exec -it nginx-pod -- /bin/bash

root@nginx-pod:/# curl 10.244.0.86

<html><body><h1>It works!</h1></body></html>
```


**Default Service in k8s:**
- ClusterIP is default service (svc) in k8s.

```
manish@MacBook-Pro-va-FOX k8s % kubectl get service                               

NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE

kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   113d
```


ClusterIP services are useful for:

- Communication between different services or applications within the same cluster.
- Exposing an application only within the cluster for debugging or testing purposes.
- Providing a stable IP and DNS name for a set of pods, even as individual pods are created and terminated.

If you need to expose your application outside the cluster, you should use other types of services like NodePort or LoadBalancer.


**How to check which selector is used in the service?**

```
manish@MacBook-Pro-va-FOX k8s % kubectl describe service my-service         
Name:                     my-service
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=my-app
Type:                     ClusterIP
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.111.154.208
IPs:                      10.111.154.208
Port:                     <unset>  80/TCP
TargetPort:               8080/TCP
Endpoints:                
Session Affinity:         None
Internal Traffic Policy:  Cluster
Events:                   <none>
```

or
```
manish@MacBook-Pro-va-FOX k8s % kubectl get service my-service -o jsonpath='{.spec.selector}'
{"app":"my-app"}
```


- Based on the selector criteria service takes the pod under control.
- Service will be assigned IP address from internal range of k8s IP address. This IP will not come from CNI (Pod network)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-clusterip-svc
spec:
  selector:
    app: my-app
  ports:
  - port: 801  # Svc listen on port 801
    targetPort: 80  # container port 80
  type: ClusterIP
```

- Service gets the IP address (10.105.176.80) from k8s network
- Service should listen on port , here 801 port
- K8s is going to create DNS entry for the service (innternal k8s) . Name of the service resolves to IP address of the service.
- Inside kube-sytem namespace, DNS pod will be running there.
- DNS entry for the service will be created in DNS pod resides in kube-system

```
my-cluster-ip is ClusterIP service type annd listen on TCP Pord 801.

NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes         ClusterIP   10.96.0.1       <none>        443/TCP   113d
my-clusterip-svc   ClusterIP   10.105.176.80   <none>        **801/TCP**   4h7m
```

DNS entry for the service will be created in DNS pod which runs is kube-system ns.
```
manish@MacBook-Pro-va-FOX k8s % kubectl get pods -n kube-system 
NAME                               READY   STATUS    RESTARTS         AGE
coredns-668d6bf9bc-nh89k           1/1     Running   5 (25h ago)      113d
```

- Get the service endpoints. Service endpoint is >> podIP : containerPort.

- Get the SVC Name and Service Port (service listen to which port). Service listen to Port 801 as per yaml file and kubectl get svc  (here we can see the port)
```
manish@MacBook-Pro-va-FOX k8s % kubectl get ep my-clusterip-svc                        

NAME               ENDPOINTS        AGE

my-clusterip-svc   10.244.0.87:80   4h21m

manish@MacBook-Pro-va-FOX k8s % kubectl get svc                
NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes         ClusterIP   10.96.0.1       <none>        443/TCP   113d
my-clusterip-svc   ClusterIP   10.105.176.80   <none>        801/TCP   4h22m

#Execute curl command usnig service-name:service-port
manish@MacBook-Pro-va-FOX k8s % kubectl exec -it nginx-pod -- /bin/bash
root@nginx-pod:/# curl my-clusterip-svc:801
<html><body><h1>It works!</h1></body></html>
```

**How the service takes the pod under control?**

- By selecting the pod on the basis of their label. Pod label should be match with service selector.
- Service & Pod should be in the same ns.
pod.yaml

``` yaml 
apiVersion: v1
kind: Pod
metadata:
  name: httpd-pod
  labels:
   app: my-app 
spec:
  containers:
  - name: httpd-container
    image: httpd:latest
    ports:
    - containerPort: 80
```


service.yaml
```clusterIp-svc.yml
apiVersion: v1
kind: Service
metadata:
  name: my-clusterip-svc
spec:
  selector:
    app: my-app
  ports:
  - port: 801  # Svc listen on port 801
    targetPort: 80  # container port 80
  type: ClusterIP
```


By selecting the pod on the basis of pod labels.
``` yaml
spec:           # Service selector from servic yaml
  selector:
    app: my-app
```

Pod should have matching label with selector of the service.

```yaml
metadata:
  name: httpd-pod
  labels:    # pod labels from pod yaml (it should match with selector from service.yml
   app: my-app 

```

**Endpoint of Service:**

Endpoint of a service will be >>>  Pod IP : ContainerPort (TargetPort) 
```
manish@MacBook-Pro-va-FOX k8s % kubectl get ep my-clusterip-svc
NAME               ENDPOINTS        AGE
my-clusterip-svc   10.244.0.89:80   4h52m
```

If we delete the pod - what will be the end point of the service:
Endpoint- None

```
manish@MacBook-Pro-va-FOX k8s % kubectl delete -f httpd-pod.yml 
pod "httpd-pod" deleted

manish@MacBook-Pro-va-FOX k8s % kubectl get ep my-clusterip-svc
NAME               ENDPOINTS   AGE
my-clusterip-svc   <none>      4h55m
```

- While accessing the pod using the service. We need to provide the service-name:servicePort .
- my-clusterip-svc listens on port 801 so we need to provide curl my-clusterip-svc:801
- Service endpoint is PodIP:ContainerPort so it connects the the Pod. Port mapping is defined in the Pod. What is service port and TargetPort(container port). This is called port binding.
- If service is deleted then service entry will be deleted from the DNS. If we try to connect using the name of the service and we get " Could not resolve host: my-clusterip-svc".
```
manish@MacBook-Pro-va-FOX k8s % kubectl delete -f clusterIp-svc.yml
service "my-clusterip-svc" deleted

manish@MacBook-Pro-va-FOX k8s % kubectl exec -it nginx-pod -- /bin/bash
root@nginx-pod:/# curl my-clusterip-svc:801

curl: (6) Could not resolve host: my-clusterip-svc
```

- If we create the service , IP will be changed and DNS entry will be updated. Service will take the pod under control.  As we access the service via service name so it connects to the expected pod. Service expose the pod using pod label as selector.

```
manish@MacBook-Pro-va-FOX k8s % kubectl apply  -f clusterIp-svc.yml
service/my-clusterip-svc created

manish@MacBook-Pro-va-FOX k8s % kubectl get ep my-clusterip-svc        
NAME               ENDPOINTS        AGE
my-clusterip-svc   10.244.0.90:80   2m27s

manish@MacBook-Pro-va-FOX k8s % kubectl get svc
NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes         ClusterIP   10.96.0.1       <none>        443/TCP   114d
my-clusterip-svc   ClusterIP   10.101.57.110   <none>        801/TCP   2m56s

manish@MacBook-Pro-va-FOX k8s % kubectl exec -it nginx-pod -- /bin/bash
root@nginx-pod:/# curl my-clusterip-svc:801
<html><body><h1>It works!</h1></body></html>
```

**What if we have two pods with same label as (app: my-app)?**

- We have httpd-pod with label as app:web-app
```
manish@MacBook-Pro-va-FOX k8s % kubectl get pod --show-labels
NAME                   READY   STATUS    RESTARTS   AGE     LABELS
httpd-pod              1/1     Running   0          7m56s   app=my-app
```

- Create one more pod with label app:web-app using yaml or command line. Check the endpoint of the service. There will be two ep.
- I have added two labels app: my-app and env: development. Service selector checks the label app=myapp and it selects this pod.
```yaml
manish@MacBook-Pro-va-FOX k8s % cat ngnix-second-pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-second-pod
  labels:
    app: my-app
    env: development
spec:
  containers:
  - name: nginx-container
    image: nginx:latest
    ports:
    - containerPort: 80
```
```
manish@MacBook-Pro-va-FOX k8s % kubectl get svc
NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes         ClusterIP   10.96.0.1       <none>        443/TCP   114d
my-clusterip-svc   ClusterIP   10.101.57.110   <none>        801/TCP   14m

manish@MacBook-Pro-va-FOX k8s % kubectl get ep my-clusterip-svc 
NAME               ENDPOINTS                       AGE
my-clusterip-svc   10.244.0.90:80,10.244.0.92:80   14m

manish@MacBook-Pro-va-FOX k8s % kubectl get pod -o wide -l app=my-app
NAME               READY   STATUS    RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES
httpd-pod          1/1     Running   0          14m   10.244.0.90   minikube   <none>           <none>
nginx-second-pod   1/1     Running   0          71s   10.244.0.92   minikube   <none>           <none>

```

Access the service from test pod(nginx)
first time connection goes to httpd-pod and next time it goes to nginx-second-pod 
```
manish@MacBook-Pro-va-FOX k8s % kubectl exec -it nginx-pod -- /bin/bash
root@nginx-pod:/# curl my-clusterip-svc:801
<html><body><h1>It works!</h1></body></html>
root@nginx-pod:/# curl my-clusterip-svc:801
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
```


If there are two container in the pod. one container is httpd and another container is rabit mq.
Service takes the pod under the control based on the selector.

when we access the service 
curl service-name:Serviceport 
Where the connection goes (in container1 or container 2).

It depends on the port binding.
If we use the 
curl my-clusterip-svc:801 - it connect to httpd container
curl my-clusterip-svc:802 - it connects to rabit-mq.
```yaml
manish@MacBook-Pro-va-FOX k8s % cat clusterIp-svc-multi.yml
apiVersion: v1
kind: Service
metadata:
  name: my-clusterip-svc-multi
spec:
  selector:
    app: my-app
  ports:
  - protocol: TCP
    name: http-port
    port: 801  # Svc listen on port 801
    targetPort: 80  # container port 80
  - protocol: TCP
    name: rabbit-port
    port: 802  # Svc listen on port 801
    targetPort: 5672  # container port 5672
  type: ClusterIP

```

Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
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
  - name: rabbitmq
    image: rabbitmq:latest
    ports:
    - containerPort: 5672
```

Execute yaml files

```
manish@MacBook-Pro-va-FOX k8s % kubectl apply -f multi-conpod.yml 

pod/multi-container-pod created

manish@MacBook-Pro-va-FOX k8s % kubectl apply -f clusterIp-svc-multi.yml

service/my-clusterip-svc-multi created
```

Verify the service listening ports: should listen on 2 ports.
```
manish@MacBook-Pro-va-FOX k8s % kubectl get svc my-clusterip-svc-multi
NAME                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)           AGE
my-clusterip-svc-multi   ClusterIP   10.110.148.142   <none>        801/TCP,802/TCP   21s
```

Verify the endpoint of the service. Also check the IP adddress of POD. There should be two endpoint with same IP with port 80 and with port 5672.

```
manish@MacBook-Pro-va-FOX k8s % kubectl get ep my-clusterip-svc-multi

NAME                     ENDPOINTS                         AGE

my-clusterip-svc-multi   10.244.0.96:80,10.244.0.96:5672   3m17s


manish@MacBook-Pro-va-FOX k8s % kubectl get ep my-clusterip-svc-multi
NAME                     ENDPOINTS                         AGE
my-clusterip-svc-multi   10.244.0.96:80,10.244.0.96:5672   3m17s


manish@MacBook-Pro-va-FOX k8s % kubectl get pod -o wide

NAME                  READY   STATUS    RESTARTS   AGE     IP            NODE       NOMINATED NODE   READINESS GATES

multi-container-pod   2/2     Running   0          5m23s   10.244.0.96   minikube   <none>           <none>

nginx-pod             1/1     Running   0          29m     10.244.0.93   minikube   <none>           <none>
```

There are 3 types of services:
- ClusterIP -- Default service type. To expose the pod internally with k8s cluster.
- NodePort -- This service used to expose the pod internally as well as externally
- LoadBalance -- This service used to expose the pod internally as well as externally

NodePort Service