A ReplicaSet controller in Kubernetes is a core controller responsible for maintaining a stable set of replica Pods running at any given time. Its primary role is self-healing and ensuring the desired number of Pod replicas specified in the ReplicaSet manifest is always running and available.
How ReplicaSet Controller Works:
	•	The ReplicaSet defines a desired number of replicas and a label selector to identify the Pods it manages.
	•	The controller continuously monitors Pods matching its selector.
	•	If the actual number of Pods is fewer than the desired replicas, the controller creates new Pods based on the Pod template.
	•	If there are more Pods than desired, it deletes the excess Pods.
	•	It tracks ownership of Pods via metadata ownerReferences, allowing it to manage the lifecycle of its Pods effectively.
	•	The controller performs scaling operations automatically based on updates to the ReplicaSet’s `spec.replicas` field.
	•	It ensures availability by replacing Pods that have crashed, been deleted, or lost due to node failure.
Why use ReplicaSet?
	•	Guarantees that a specified number of identical Pods are always running.
	•	Provides a simple self-healing mechanism for application Pods.
	•	Useful for workloads where you want multiple copies of a Pod to run simultaneously.
Limitations:
	•	ReplicaSet does not support rolling updates; for advanced deployment strategies, a Deployment controller should be used which manages ReplicaSets under the hood.
	•	ReplicaSets work with label selectors and pod templates but do not handle updates to the pod specification elegantly—that’s the Deployment’s job.


```
apiVersion: apps/v1
kind: ReplicaSet
metadata: 
  name: rs-nginx
  labels:				# label on RS 
    app: nginx
spec: 
  replicas: 3   # number of pod's that controller will MANAGING
  selector: # selector is used to define that if there is any STATIC pod thatmatches below  expression should be taken undercontrol first 
    matchExpressions:
      - {key: app, operator: In, values: [ nginx_1, nginx_2]}
  template:		# definition of pod's that controller will be CREATING
    metadata:
      name: nginx		# no importance, because names of pod willbe aut-generated
      labels:
        app: nginx_2 
    spec:
      containers:
        - image: nginx:latest
          name: nginx
```

**Scenario:-** How many new pod will be created considering there is already one pod is running with label nginx_1.

Define standalone pod for testing:
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-standalone-pod
  labels:
    app: nginx_1
    env: devlopment
spec:
  containers:
  - name: nginx-container
    image: nginx:latest
    ports:
    - containerPort: 80
```

```
manish@MacBook-Pro-va-FOX k8s % kubectl apply -f ngnix-second-pod.yml
manish@MacBook-Pro-va-FOX replicaset % kubectl get po --show-labels

NAME                       READY   STATUS    RESTARTS   AGE     LABELS
pod/nginx-standalone-pod   1/1     Running   0          5m40s   app=nginx_1,env=devlopment
```

Execute the ReplicaSet. 
Two new Pod will be created.

```
manish@MacBook-Pro-va-FOX replicaset % kubectl apply -f my-rs.yml 
replicaset.apps/rs-nginx created

manish@MacBook-Pro-va-FOX replicaset % kubectl get po,rs
NAME                       READY   STATUS    RESTARTS   AGE
pod/nginx-standalone-pod   1/1     Running   0          80s
pod/rs-nginx-f9m77         1/1     Running   0          22s
pod/rs-nginx-lkqv6         1/1     Running   0          22s

NAME                       DESIRED   CURRENT   READY   AGE
replicaset.apps/rs-nginx   3         3         3       22s
```

- The ReplicaSet selector uses the `matchExpressions` to identify which Pods it manages. In your example, it matches Pods that have the label `app` with a value either `nginx_1` or `nginx_2`:

```
selector:
  matchExpressions:
    - {key: app, operator: In, values: [nginx_1, nginx_2]}
```

- We already have one standalone Pod running with label `app=nginx_1`.
- ReplicaSet controller sees this Pod matches its selector and will adopt it under its control.
So, ReplicaSet - rs-nginx  takes standalone pod under control if that pod has mathing selector with replcaSet. Standalone pod has two labels and one of them is app=nginx_1 which matches with Replicaset selector matchExpressions. 
```
NAME                       READY   STATUS    RESTARTS   AGE     LABELS
pod/nginx-standalone-pod   1/1     Running   0          5m40s   app=nginx_1,env=devlopment
```

So 1 pod is already there, so what will be the shortfall of the pods that will be created by ReplicaSet controller manager. ReplicaSet controller manager creates the pod according to the pod template which we have specified in Replicaset yaml file. The new pod will be created with nginx_2 label (as this is label definned in template for pod in replicaset yaml file)

- Your Pod template for the ReplicaSet has `app=nginx_2`.
- The ReplicaSet desires a total of 3 replicas, so since 1 Pod already matches (nginx_1), it needs to create 2 more Pods as per template (with label `app=nginx_2`).
- The new Pods are created with labels defined in the ReplicaSet template (`app=nginx_2`).
- So total Pods after creation are 3: 1 existing with `app=nginx_1` + 2 new with `app=nginx_2`.
So there will be two new pod got created with label nginx_2, so total pod now are 3.


**Scenario:-** If we delete the standalone-pod then will the replicaset creates the pod and what will be the label?

Yes, we can delete the pod. New label will be nginx_2 as per template defined in ReplicaSet.


**Scenario:-** If there are already 3 pod running. If I am going to create the pod with nginx_1 or nginx_2 label what will happen?

As per the controller defination at a time only 3 pod will be runnign with RS selector match expression. So new pod will be terminated if it has label wither nginix_1 or nginx_2.


**How to expose the ReplicaSet with LB service?**

A Service does not point “at the ReplicaSet object” but at Pods via labels, so the only requirement is that your ReplicaSet’s Pod template has stable labels (for example `app: nginx_2 ) and your Service `spec.selector` matches those labels. The Service then load-balances across all matching Pods, regardless of their individual names.

Create a LoadBalancer Service  to expose the pod with lable as nginx_2

```
apiVersion: v1
kind: Service
metadata:
  name: my-lb-svc
spec:
  selector:
    app: nginx_2
  ports:
  - port: 801  # Svc listen on port 801
    targetPort: 80  # container port 80
  type: LoadBalancer

~
```

or via command line
```
kubectl expose rs rs-nginx \
  --type=LoadBalancer \
  --port=801 \
  --target-port=80 \
  --name=my-lb-svc

```

```
manish@MacBook-Pro-va-FOX replicaset % kubectl apply -f lb-svc.yml
service/my-lb-svc created

manish@MacBook-Pro-va-FOX replicaset % kubectl get svc
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)         AG
my-lb-svc    LoadBalancer   10.105.121.8   10.105.121.8   801:30594/TCP   44s

manish@MacBook-Pro-va-FOX replicaset % kubectl get ep my-lb-svc

NAME        ENDPOINTS         AGE
my-lb-svc   10.244.0.100:80   59s


manish@MacBook-Pro-va-FOX replicaset % kubectl get pod,rs,svc --show-labels -o wide
NAME.         READY   STATUS    RESTARTS AGE   IP             NODE       NOMINATED NODE   READINESS GATES   LABELS
pod/httpd-pod  1/1     Running   0       24m   10.244.0.103   minikube   <none>           <none>            run=httpd-pod

pod/nginx-standalone-pod   1/1     Running   0          71m   10.244.0.100   minikube   <none>           <none>            app=nginx_1,env=devlopment

pod/rs-nginx-f9m77         1/1     Running   0          70m   10.244.0.102   minikube   <none>           <none>            app=nginx_2

pod/rs-nginx-lkqv6         1/1     Running   0          70m   10.244.0.101   minikube   <none>           <none>            app=nginx_2

  

NAME                       DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES         SELECTOR                   LABELS
replicaset.apps/rs-nginx   3         3         3       70m   nginx        nginx:latest   app in (nginx_1,nginx_2)   app=nginx

  
NAME                 TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)         AGE    SELECTOR      LABELS

service/my-lb-svc    LoadBalancer   10.105.121.8   10.105.121.8   801:30594/TCP   26m    app=nginx_1   <none>
```

Load balancer serice is created and access via extrnal-ip:serviceport


**How to scale up  and scale down the pod using ReplicaSet?**

```
manish@MacBook-Pro-va-FOX replicaset % kubectl scale rs rs-nginx --replicas=5

NAME                       DESIRED   CURRENT   READY   AGE

replicaset.apps/rs-nginx   5         5         5       26h

NAME                       READY   STATUS    RESTARTS   AGE

pod/httpd-pod              1/1     Running   0          25h
pod/nginx-standalone-pod   1/1     Running   0          26h
pod/rs-nginx-f9m77         1/1     Running   0          26h
pod/rs-nginx-fj4tj         1/1     Running   0          20s
pod/rs-nginx-lkqv6         1/1     Running   0          26h
pod/rs-nginx-wqgz2         1/1     Running   0          20s

manish@MacBook-Pro-va-FOX replicaset % kubectl scale rs rs-nginx --replicas=2
replicaset.apps/rs-nginx scaled

manish@MacBook-Pro-va-FOX replicaset % kubectl get rs,pod --show-labels
NAME                       DESIRED   CURRENT   READY   AGE   LABELS
replicaset.apps/rs-nginx   2         2         2       26h   app=nginx


NAME                       READY   STATUS    RESTARTS   AGE   LABELS
pod/httpd-pod              1/1     Running   0          25h   run=httpd-pod
pod/nginx-standalone-pod   1/1     Running   0          26h   app=nginx_1,env=devlopment
pod/rs-nginx-f9m77         1/1     Running   0          26h   app=nginx_2
```
