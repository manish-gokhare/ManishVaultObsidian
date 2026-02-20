
```
kubectl create deployment test-deploy --image=nginx --replicas=3

kubectl get deploy,rs,pod --show-labels
```

- Deployment (test-deploy) will be created. 
- ReplicaSet will be created 
- Pod will be created.
- There will be 3 copy of the nginx pod.
- Deployment calls ReplicaSet and ReplicaSet is going to create the Pod.
- If Pod is deleted then ReplicaSet will create the Pod.
- If ReplicaSet is deleted. Deployment will create RS and RS creates new Pods.
- If we delete the deployment - all objects created by deployment like RS, Pod will be deleted.
- test-deploy deployment will be created with label test-deploy. This lablel of deployment will be test-deploy.
- ReplicaSet will be created by deployment . RS lable will be test-deploy.
- Pod will be created with lable test-deploy. RS creates the Pod. 
- If we delete the RS, Deployment will create new RS and RS creates new pods.
- Deployment controls RS and RS control Pod.

Deployment
  └─ manages ReplicaSet(s)
       └─ ReplicaSet ensures desired number of Pods running
            └─ Pods (multiple replicated instances of containers)

- Deployment is the top-level controller. When you create or update a Deployment, it automatically creates or updates a ReplicaSet.
- ReplicaSet is responsible for keeping the specified number of Pod replicas up and running.
- Each Pod is an instance of your application container(s).
- If a Pod fails, the ReplicaSet creates a new Pod to maintain the desired count.
- Deployment also handles rollout of updates by creating new ReplicaSets with updated Pod templates and scaling down old ReplicaSets.

[Deployment] → controls → [ReplicaSet] → controls → [Pods]


```
manish@MacBook-Pro-va-FOX deployment % kubectl get deploy,rs,pod --show-labels
NAME                          READY   UP-TO-DATE   AVAILABLE   AGE   LABELS
deployment.apps/test-deploy   3/3     3            3           17s   app=test-deploy

  

NAME                                     DESIRED   CURRENT   READY   AGE   LABELS
replicaset.apps/test-deploy-848c855d5f   3         3         3       17s   app=test-deploy,pod-template-hash=848c855d5f

  

NAME                               READY   STATUS    RESTARTS   AGE   LABELS
pod/test-deploy-848c855d5f-6n8qm   1/1     Running   0          17s   app=test-deploy,pod-template-hash=848c855d5f
pod/test-deploy-848c855d5f-7m4df   1/1     Running   0          17s   app=test-deploy,pod-template-hash=848c855d5f
pod/test-deploy-848c855d5f-rj22p   1/1     Running   0          17s   app=test-deploy,pod-template-hash=848c855d5f
```
YAML way equivalent to above.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: test-deploy
  name: test-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: test-deploy
  template:
    metadata:
      labels:
        app: test-deploy
    spec:
      containers:
      - image: nginx
        name: nginx
```

**Manual way to scale up or scale down the pod**

```
**kubectl scale deploy test-deploy --replicas=5** 

manish@MacBook-Pro-va-FOX deployment % kubectl scale deploy test-deploy --replicas=5 
deployment.apps/test-deploy scaled
manish@MacBook-Pro-va-FOX deployment % kubectl get deploy,rs,pod --show-labels

NAME                          READY   UP-TO-DATE   AVAILABLE   AGE   LABELS
deployment.apps/test-deploy   5/5     5            5           14m   app=test-deploy

NAME                                     DESIRED   CURRENT   READY   AGE   LABELS
replicaset.apps/test-deploy-848c855d5f   **5**         5         5       14m   app=test-deploy,pod-template-hash=848c855d5f

NAME                               READY   STATUS    RESTARTS   AGE   LABELS

pod/test-deploy-848c855d5f-bw25n   1/1     Running   0          14m   app=test-deploy,pod-template-hash=848c855d5f

pod/test-deploy-848c855d5f-dkq9q   1/1     Running   0          5s    app=test-deploy,pod-template-hash=848c855d5f

pod/test-deploy-848c855d5f-jrcf9   1/1     Running   0          5s    app=test-deploy,pod-template-hash=848c855d5f

pod/test-deploy-848c855d5f-n5d9q   1/1     Running   0          14m   app=test-deploy,pod-template-hash=848c855d5f

pod/test-deploy-848c855d5f-p2f2q   1/1     Running   0          14m   app=test-deploy,pod-template-hash=848c855d5f
```


Expose the deployment:

```
kubectl expose deployment test-deploy --type=NodePort --port=8080 --target-port=80 --name=test-svc --dry-run=client -o yaml

apiVersion: v1
kind: Service
metadata:
  creationTimestamp: nul
  labels:
    app: test-deploy
  name: test-svc
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 80
  selector:
    app: test-deploy
  type: NodePort
```


```
manish@MacBook-Pro-va-FOX deployment % kubectl expose deployment test-deploy --type=NodePort --port=8080 --target-port=80 --name=test-svc
service/test-svc exposed

manish@MacBook-Pro-va-FOX deployment % kubectl get svc

NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE

kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP          120d

test-svc     NodePort    10.101.84.243   <none>        8080:32411/TCP   6s

manish@MacBook-Pro-va-FOX deployment % kubectl get ep test-svc

NAME       ENDPOINTS                                                     AGE
test-svc   10.244.0.115:80,10.244.0.116:80,10.244.0.117:80 + 2 more...   21s

```

- There will be 5 endpoint of the service test-svc.
- This is NodePort service can be access via Node IP & Node PORT 
- http://192.168.64.3:32411


**Reduce the replica from 5 to 2.**

```
manish@MacBook-Pro-va-FOX deployment % kubectl scale deploy test-deploy --replicas=2
deployment.apps/test-deploy scaled

manish@MacBook-Pro-va-FOX deployment % kubectl get deploy,rs,pod,svc --show-labels
NAME                          READY   UP-TO-DATE   AVAILABLE   AGE     LABELS
deployment.apps/test-deploy   2/2     2            2           5h46m   app=test-deploy

NAME                                     DESIRED   CURRENT   READY   AGE     LABELS
replicaset.apps/test-deploy-848c855d5f   2         2         2       5h46m   app=test-deploy,pod-template-hash=848c855d5f

NAME                               READY   STATUS    RESTARTS   AGE     LABELS
pod/test-deploy-848c855d5f-dkq9q   1/1     Running   0          5h31m   app=test-deploy,pod-template-hash=848c855d5f

pod/test-deploy-848c855d5f-jrcf9   1/1     Running   0          5h31m   app=test-deploy,pod-template-hash=848c855d5f

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE     LABELS
service/kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP          121d    component=apiserver,provider=kubernetes

service/test-svc     NodePort    10.101.84.243   <none>        8080:32411/TCP   5h14m   app=test-deploy
```

**How many endpoint for the service test-svc?**

There will be two endpoint. As there are two pods are running. PodIP:TargetPort

```
manish@MacBook-Pro-va-FOX deployment % kubectl get ep test-svc
NAME       ENDPOINTS                         AGE
test-svc   10.244.0.118:80,10.244.0.119:80   5h18m
```

**Delete the deployment**


**Deployment using yaml file**

```
kubectl explain deployment

```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchExpressions:
      - key: app
        operator: In
        values:
          - nginx
      - key: environment
        operator: Not In
        values:
          - dev
  template:
    metadata:
      labels:
        app: nginx
        environment: prod
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        resources:
          limits:     # Maximum guaranted
            cpu: 1m  # 1m = 0.0001 core or 1000m = 1 core
            memory: 40Mi
           requests:  #Minimum guaranteed 
            cpu: 1m
            memory: 30Mi
            
        
        
```


```


