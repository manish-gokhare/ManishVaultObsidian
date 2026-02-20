
**To launch the Pod:**
```k8s
kubectl run first-pod --image=nginx
```

**To get the infomation about the Pod:**
```k8s
kubectl get pods -o wide
```

```output
manish@MacBook-Pro-va-FOX k8s % kubectl get pods -o wide           

NAME        READY   STATUS    RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES

first-pod   1/1     Running   0          3s    10.244.0.72   minikube   <none>           <none>
```

**To describe the Pod:**
```k8s
kuebectl describe pod first-pod
```

```output
kubectl describe pod first-pod

Name:             first-pod

Namespace:        default

Priority:         0

Service Account:  default

Node:             minikube/192.168.64.3

Start Time:       Tue, 11 Nov 2025 13:34:25 +0100

Labels:           run=first-pod

Annotations:      <none>

Status:           Running

IP:               10.244.0.72

Containers:

  first-pod:

    Container ID:   docker://347500177e1ef0bef71ec9c3edb7b4e280381a4434aeb781e15ec3f6885f7941

    Image:          nginx

    Image ID:       docker-pullable://nginx@sha256:1beed3ca46acebe9d3fb62e9067f03d05d5bfa97a00f30938a0a3580563272ad

    Port:           <none>

    Host Port:      <none>

    State:          Running
```

**How to check the logs of the container:**
```k8s
kubectl logs first-pod
```

**How to go inside the container:**
```k8s
manish@MacBook-Pro-va-FOX k8s % kubectl exec -it first-pod -- /bin/sh          

#
```

**How to delete the Pod:**
```k8s
kubectl delete pod first-pod
```
**How to define the yaml file for Pod:**
```APIDoc
Kubernetes API documentation:
https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.31/#pod-v1-core
```


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx-container
    image: nginx:latest
    ports:
    - containerPort: 80
      hostPort: 8080
```

```Description
The container listens on port 80.
The host machine maps its port 8080 to the container’s port 80.
“Bind the node’s or hosts  port 8080 directly to this container’s port.”

You can access nginx via http://<host-ip>:8080.

As I am using minikube 
minkubeip is 192.168.64.3

http://192.168.64.3:8080/
SUCCESS.

```

**Pod communication :** 
Pod communicate with each other with Pod IP address. IP 10.244.0.74 is IP address of httpd pod.

```yaml
#Create httpd-pod
manish@MacBook-Pro-va-FOX k8s % kubectl run httpd-pod --image=httpd

pod/httpd-pod created

#Create nginx-example-pod
manish@MacBook-Pro-va-FOX k8s % kubectl run nginx-example-pod --image=nginx

pod/nginx-example-pod created

#Go inside the nginx-example-pod and access httpd-pod with httpd-pod IP.
manish@MacBook-Pro-va-FOX k8s % kubectl exec -it nginx-example-pod -- /bin/bash

root@nginx-example-pod:/# curl http://10.244.0.74

<html><body><h1>It works!</h1></body></html>

root@nginx-example-pod:/# curl http://10.244.0.74:80

<html><body><h1>It works!</h1></body></html>
```

**What is initContainer?**
✅ What is an Init Container?
An Init Container is a special container in a Pod that:

- Runs before the main containers.
- Performs initialization tasks.
- Must complete successfully before the main containers start.
- Can have different images, tools, or configurations than the main container.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-with-init
spec:
  volumes:
  - name: config-volume
    emptyDir: {}

  initContainers:
  - name: init-config
    image: busybox
    command: ['sh', '-c', 'echo "Welcome to Nginx!" > /config/index.html']
    volumeMounts:
    - name: config-volume
      mountPath: /config

  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: config-volume
      mountPath: /usr/share/nginx/html
    ports:
    - containerPort: 80
```
  
🔍 What Happens Here:


Init Container (init-config):

Runs first.
Writes a simple HTML file to /config/index.html.
Mounts a shared volume (config-volume).

Main Container (nginx):
Starts only after the init container finishes.
Mounts the same volume at /usr/share/nginx/html.
Serves the HTML file created by the init container.

```Do the port-forward to see the application which is runnning in the container.
  kubectl port-forward pod/nginx-with-init 8090:80

Forwarding from 127.0.0.1:8090 -> 80
Forwarding from [::1]:8090 -> 80
Handling connection for 8090
Handling connection for 8090

Login to Webbrowser:
http://127.0.0.1:8090


```login to container ```
Main container (nginx) serves the file from /usr/share/nginx/html/
manish@MacBook-Pro-va-FOX k8s % kubectl exec -it nginx-with-init -- /bin/sh
Defaulted container "nginx-container" out of: nginx-container, init-config (init)
# ls

# cat /usr/share/nginx/html/index.html
Welcome to Nginx!


```Debug the container
kubectl logs nginx-with-init

As initContainer do not stay running after their task in complete. Once it is complete k8s remove them from running state.
'''

