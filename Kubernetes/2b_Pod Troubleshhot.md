**Create a Pod with name which is not present in registry.**

 1) **ErrImagePull**
```
manish@MacBookPro ~ % kubectl run second-pod --image=nginx1
pod/second-pod created

manish@MacBookPro ~ % kubectl get pods --show-labels      
NAME         READY   STATUS         RESTARTS   AGE     LABELS
second-pod   0/1     ErrImagePull   0          9s      run=second-pod
```

We can see STATUS of Pod is  ErrImagePull

**How to troubleshoot?**

```
1) Describe the Pod
2) Check events in the Pod

manish@MacBookPro ~ % kubectl describe pod second-pod

Events:

  Type     Reason     Age                From               Message

  ----     ------     ----               ----               -------

Normal   Scheduled  64s                default-scheduler  Successfully assigned default/second-pod to minikube
Normal   Pulling    25s (x3 over 64s)  kubelet            Pulling image "nginx1"
Warning  Failed     23s (x3 over 63s)  kubelet            Failed to pull image "nginx1": Error response from daemon: pull access denied for nginx1, repository does not exist or may require 'docker login': denied: requested access to the resource is denied
Warning  Failed     23s (x3 over 63s)  kubelet            Error: ErrImagePull
Normal   BackOff    13s (x3 over 62s)  kubelet            Back-off pulling image "nginx1"
Warning  Failed     13s (x3 over 62s)  kubelet            Error: ImagePullBackOff


```

# Why Scheduling Must Happen First (Architecture Logic)

In Kubernetes:

1. **default-scheduler** decides which node to run the pod on
    
2. Scheduler updates Pod spec with `nodeName`
    
3. Then **kubelet on that node**:
    
    - Pulls the image
        
    - Creates container
        
    - Starts container
        

- `default-scheduler` → handles scheduling
    
- `kubelet` → handles image pulling & container runtime


Scheduled → Pulling → Failed → BackOff
