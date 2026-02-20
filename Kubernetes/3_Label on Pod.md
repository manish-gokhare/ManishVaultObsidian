
Labels are user defined tag that helps to identify any resource (Pod,service,deployment.node etc) in k8s.

Labels are defined as key:value.

Labels are created by default when we use command line to create the pod.  

```
manish@MacBook-Pro-va-FOX k8s % kubectl run first-pod --image=httpd

pod/first-pod created

manish@MacBook-Pro-va-FOX k8s % kubectl get pods --show-labels

NAME        READY   STATUS    RESTARTS   AGE   LABELS

first-pod   1/1     Running   0          15s   run=first-pod
```


In yaml file we need to define the labels for the k8s resource.

```yaml
apiVersion: v1

kind: Pod

metadata:

  name: httpd-pod-with-label

  namespace: default

  labels:

    app: httpd

    environment: development

spec:

  containers:

  - name: httpd

    image: httpd:latest

    ports:

    - containerPort: 80
```


```output
manish@MacBook-Pro-va-FOX k8s % kubectl get pod --show-labels

NAME                   READY   STATUS    RESTARTS   AGE    LABELS

httpd-pod-with-label   1/1     Running   0          10s    app=httpd,environment=development

```

Labels the Pod using command line

```
manish@MacBook-Pro-va-FOX k8s % kubectl get pod --show-labels

NAME                   READY   STATUS    RESTARTS   AGE     LABELS

first-pod              1/1     Running   0          9m37s   run=first-pod

# Label the Pod

manish@MacBook-Pro-va-FOX k8s % kubectl label pod first-pod app=httpd

pod/first-pod labeled

#Verify the label

manish@MacBook-Pro-va-FOX k8s % kubectl get pod --show-labels        

NAME                   READY   STATUS    RESTARTS   AGE     LABELS

first-pod              1/1     Running   0          10m     app=httpd,run=first-pod

#Remove the specific label (run=first-pod)

manish@MacBook-Pro-va-FOX k8s % kubectl label pod first-pod run-     

pod/first-pod unlabeled

manish@MacBook-Pro-va-FOX k8s % kubectl get pod --show-labels   

NAME                   READY   STATUS    RESTARTS   AGE     LABELS

first-pod              1/1     Running   0          10m     app=httpd

```

