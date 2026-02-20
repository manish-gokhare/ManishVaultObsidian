
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

nginx-pod              1/1     Running   0          28m   10.244.0.85   minikube   <none>           <none>

# Go inside the nginx-pod and access httpd-pod-with-label using its IP (10.244.0.86)

manish@MacBook-Pro-va-FOX k8s % kubectl exec -it nginx-pod -- /bin/bash

root@nginx-pod:/# curl 10.244.0.86

<html><body><h1>It works!</h1></body></html>

```

