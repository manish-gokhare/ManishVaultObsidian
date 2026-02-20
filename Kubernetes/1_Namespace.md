Namespace is a way to isolate the resources in k8s cluster.

hadoop-ns
db-ns
kafka-ns
monitoring-ns

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: development
```

```output
manish@MacBook-Pro-va-FOX k8s % kubectl apply -f ns.yml 

namespace/development created

manish@MacBook-Pro-va-FOX k8s % kubectl get ns

NAME                   STATUS   AGE

default                Active   113d
development            Active   6s
kube-node-lease        Active   113d
kube-public            Active   113d
kube-system            Active   113d
kubernetes-dashboard   Active   113d
```

**Delete development namespace:**
```command
kubectl delete -f ns.yaml
```
**command: kubectl create ns web-ns**
```command
manish@MacBook-Pro-va-FOX k8s % kubectl create ns web-ns
namespace/web-ns created
```

**How to deploy the Pod in the namespace:**

- Create namespace development 
- Deploy the Pod in development namespace
- Delete the namespace (all pod in the namspace will be deleted)

```yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: development
---

apiVersion: v1
kind: Pod
metadata:
  name: httpd
  namespace: development
spec:
  containers:
  - name: httpd
    image: httpd:latest
    ports:
    - containerPort: 80
```

```info

ContainerPort - It is the port where container listens to. Apache httpd listens to port 80 always. It only documents and maps metadata.

Kubernetes does NOT change the actual listening port.

For example:

If your container listens on port 8080 but you write:
    ports:
    - containerPort: 80
      
Kubernetes does NOT change the actual listening port.

The container will still listen on 8080.    


```



**Execute the yaml ns.yml and ns-pod.yml to create ns and pod in ns**
```command
manish@MacBook-Pro-va-FOX k8s % kubectl apply -f ns.yml 

namespace/development created

manish@MacBook-Pro-va-FOX k8s % kubectl apply -f ns-pod.yml 

pod/httpd created

manish@MacBook-Pro-va-FOX k8s % kubectl get pods 

NAME              READY   STATUS    RESTARTS     AGE

nginx-with-init   1/1     Running   1 (8h ago)   4d3h

manish@MacBook-Pro-va-FOX k8s % kubectl get pods -n development

NAME    READY   STATUS    RESTARTS   AGE

httpd   1/1     Running   0          17s

#By default - default ns will be considered.

manish@MacBook-Pro-va-FOX k8s % kubectl exec -it httpd -- /bin/bash

Error from server (NotFound): pods "httpd" not found

manish@MacBook-Pro-va-FOX k8s % kubectl exec -it httpd -n development -- /bin/bash

root@httpd:/usr/local/apache2#
```

**Delete namespace -  Pod will be deleted inside the namespace - development**

```command
manish@MacBook-Pro-va-FOX k8s % kubectl delete -f ns.yml 

namespace "development" deleted

manish@MacBook-Pro-va-FOX k8s % kubectl get pods -n development                   

No resources found in development namespace.
```

``` command
# We have web-ns already.
manish@MacBook-Pro-va-FOX k8s % kubectl run httpd-pod --image=httpd -n web-ns 

pod/httpd-pod created

manish@MacBook-Pro-va-FOX k8s % kubectl get pods -n web-ns

NAME        READY   STATUS    RESTARTS   AGE

httpd-pod   1/1     Running   0          11s

manish@MacBook-Pro-va-FOX k8s % kubectl describe pod httpd-pod -n web-ns


manish@MacBook-Pro-va-FOX k8s % kubectl delete ns web-ns

namespace "web-ns" deleted

manish@MacBook-Pro-va-FOX k8s % kubectl get pods -n web-ns

No resources found in web-ns namespace.

manish@MacBook-Pro-va-FOX k8s %

```

**To see the pod from all namespaces**
``` command
kubectl get pods -A
```

**How to delete the pod forcefully from namespace** 
``` command

manish@MacBook-Pro-va-FOX k8s % kubectl delete pod httpd-pod -n web-ns --force --grace-period=0

**Warning:** Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.

pod "httpd-pod" force deleted

manish@MacBook-Pro-va-FOX k8s % kubectl get pods -n web-ns                                     
No resources found in web-ns namespace.
```
**Set the custom namespace as default**
```command
kubectl config set-context --current --namespace=web-ns

kubectl config get-contexts
```

**Pod is running in development namespace. How to connect?**


```

manish@MacBookPro K8s % kubectl get pods -n development

NAME    READY   STATUS    RESTARTS   AGE

httpd   1/1     Running   0          14m

kubectl exec -n development -it httpd -- /bin/bash

manish@MacBookPro K8s % kubectl exec -n development -it httpd -- /bin/bash

root@httpd:/usr/local/apache2# ls

bin  build  cgi-bin  conf  error  htdocs  icons  include  logs modules

```
