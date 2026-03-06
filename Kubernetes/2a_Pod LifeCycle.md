

We’ll cover:

1. **Kubernetes Pod Lifecycle**
    
2. **How a Pod is allocated in a Production (Pro) Cluster**
    
3. **How kube-apiserver knows a Pod creation request came**
    
4. **How the scheduler knows a Pod needs scheduling**
    

---

# 1️⃣ Kubernetes Pod Lifecycle

A Pod goes through several phases from creation to termination.

## 🔄 Pod Lifecycle Phases

```text
User → API Server → etcd → Scheduler → Kubelet → Running → Terminated
```

### 📌 Main Pod Phases

|Phase|Meaning|
|---|---|
|Pending|Pod accepted, not yet running|
|Running|At least one container running|
|Succeeded|All containers exited successfully|
|Failed|At least one container failed|
|Unknown|Node communication issue|

---

## Detailed Lifecycle Flow

### 1️⃣ Pod Creation

- You run:

    ```bash
    kubectl apply -f pod.yaml
    ```
- Request goes to **kube-apiserver**
- Pod object is stored in **etcd**
- Pod status = `Pending`

### 2️⃣ Scheduling

- Scheduler sees Pod has:
    ```yaml
    spec.nodeName: <empty>
    ```

- Scheduler selects best node

- Updates Pod with:

    ```yaml
    spec.nodeName: worker-node-1
    ```


### 3️⃣ Kubelet Execution

- Kubelet on that node sees the assigned Pod
- Pulls container image
- Creates containers using container runtime (containerd / CRI-O)
- Pod becomes `Running`


### 4️⃣ Termination

- Delete command:
    
    ```bash
    kubectl delete pod mypod
    ```
    
- Pod enters `Terminating`
    
- Graceful shutdown
    
- Removed from etcd
    

---

# 2️⃣ How Pod is Allocated in a Production (Pro) Cluster

In a **Production Cluster**, architecture typically looks like:

- Multiple Control Plane nodes
    
- Multiple Worker nodes
    
- Load balancer in front of API servers
    
- HA etcd cluster
    

---

## 📌 Allocation Process in Production

### Step-by-Step Flow

### 1️⃣ User Sends Request

- Via:
    
    - `kubectl`
        
    - CI/CD pipeline
        
    - GitOps (ArgoCD)
        
    - REST API
        

Request hits:

```
Load Balancer → kube-apiserver
```

---

### 2️⃣ Authentication & Authorization

API Server:

- Authenticates user (cert/token/OIDC)
    
- Authorizes via RBAC
    

---

### 3️⃣ Admission Controllers

- Resource validation
    
- Mutating admission (add sidecars)
    
- Validating admission
    

If valid → stored in **etcd**

---

### 4️⃣ Scheduler Chooses Node

Scheduler considers:

- CPU/Memory availability
    
- Node affinity
    
- Taints & tolerations
    
- Pod affinity/anti-affinity
    
- Resource requests/limits
    
- Topology spread constraints
    

Best node selected.

---

### 5️⃣ Kubelet Creates Pod on Node

- Watches API server
    
- Sees Pod assigned to itself
    
- Pulls image
    
- Creates containers
    

---

### 🏗 Production Cluster Allocation Summary

```text
Client
  ↓
Load Balancer
  ↓
kube-apiserver (HA)
  ↓
etcd (HA)
  ↓
kube-scheduler
  ↓
Worker Node kubelet
  ↓
Container Runtime
```

---

# 3️⃣ How kube-apiserver Knows a Pod Creation Request Came?


The API Server is the **central entry point** of Kubernetes.

### When You Run:

```bash
kubectl apply -f pod.yaml
```

### What Happens Internally:

1. `kubectl` converts YAML to JSON
    
2. Sends HTTPS REST request:
    
    ```
    POST /api/v1/namespaces/default/pods
    ```
    
3. Request reaches:
    
    ```
    https://<api-server>:6443
    ```
    
4. API Server:
    
    - Authenticates
        
    - Authorizes
        
    - Validates schema
        
    - Runs admission controllers
        
    - Writes object into etcd
        

So API Server knows because:

> It exposes REST endpoints and receives HTTP POST request for Pod resource.

Kubernetes is fundamentally a **REST-based control plane**.

---

# 4️⃣ How Scheduler Knows a Pod Needs Scheduling?

This is key to understanding Kubernetes architecture.

## 🔍 Watch Mechanism (Very Important)

The scheduler does NOT get a direct message.

Instead:

👉 It **watches the API server**.

---

### Scheduler Logic:

Scheduler continuously runs:

```text
Watch for Pods where:
spec.nodeName == empty
AND status.phase == Pending
```

Internally it uses:

- Informers
    
- Shared caches
    
- Watch API
    

---

### When New Pod Is Created:

1. Pod stored in etcd
    
2. API server updates its state
    
3. Scheduler sees new unscheduled Pod
    
4. Scheduler picks node
    
5. Updates Pod with nodeName
    
6. API server stores update
    

---

## 🔁 Event-Based Model

Kubernetes uses:

> Watch → Event → Reconcile

Not polling.

---

# 🔥 Complete Flow in One View

```text
kubectl apply
    ↓
API Server (REST call)
    ↓
etcd (store pod)
    ↓
Scheduler watches for unscheduled pods
    ↓
Scheduler assigns node
    ↓
API Server updates pod
    ↓
Kubelet on selected node watches
    ↓
Creates container
    ↓
Pod Running
```

---

# 🚀 Key Concepts to Remember (Interview Focus)

### Kubernetes is:

- Declarative
    
- REST-based
    
- Event-driven
    
- Controller-based
    
- Uses Watch mechanism
    
- Uses reconciliation loop
    

---

# 🧠 Control Plane Components Responsibility

| Component               | Responsibility           |
| ----------------------- | ------------------------ |
| kube-apiserver          | Entry point & validation |
| etcd                    | Persistent storage       |
| kube-scheduler          | Node selection           |
| kube-controller-manager | Reconciliation           |
| kubelet                 | Pod execution            |
| container runtime       | Container creation       |

---

If you want, I can next explain:

- 🔹 What happens if scheduler crashes?
    
- 🔹 How high availability works in Production?
    
- 🔹 What happens when node goes down?
    
- 🔹 Deep dive into scheduling algorithm?
    
- 🔹 Real-time example with logs?
    

Just tell me your level (Beginner / Intermediate / Architect).

# 🔎 Does Scheduler Query etcd?

No.

Only **kube-apiserver talks to etcd**.

Scheduler  →  kube-apiserver  →  etcd

Scheduler never connects to etcd

# 🔄 What Actually Happens?

### Step 1 — Pod Creation

When you run:

kubectl apply -f pod.yaml

1. Request goes to **kube-apiserver**
    
2. API server:
    
    - Authenticates
        
    - Authorizes
        
    - Validates
        
    - Runs admission controllers
        
3. If valid → Pod object stored in **etcd**
    
4. Pod status = `Pending`
    
5. `spec.nodeName = ""` (empty)
    

---

# 🔔 Step 2 — How Scheduler Knows?

Scheduler has a **Watch** open to kube-apiserver.

It uses the Kubernetes Watch API:

GET /api/v1/pods?watch=true

Internally this uses:

- Informers
    
- Shared caches
    
- Event stream
    

---

### Important Concept

Scheduler does NOT:

- Poll etcd
    
- Query etcd directly
    
- Scan database
    

Instead it:

> Subscribes to change events from the API server

---

# 🧠 Event Flow Internally

When Pod is stored in etcd:

1. etcd notifies API server
    
2. API server updates its internal cache
    
3. API server sends event to all watchers
    
4. Scheduler receives "ADDED" event
    
5. Scheduler checks:
    
    - `spec.nodeName == ""`
        
    - Pod is not terminal
        

Then scheduling starts.

---

# 📌 About This Condition:

You wrote:

spec.nodeName == empty  
AND status.phase == Pending

That is conceptually correct but internally Kubernetes mostly checks:

spec.nodeName == ""

Because:

- New Pods are automatically `Pending`
    
- Main trigger is empty nodeName
    

So the key scheduling condition is:

> Pod has no assigned node.

---

# 🔁 Full Communication Model

Here is the real architecture:

kubectl  
   ↓  
kube-apiserver  
   ↓  
etcd  
   ↑  
Scheduler (watch)  
   ↑  
kubelet (watch)

Notice:

- Scheduler watches API server
    
- Kubelet watches API server
    
- Controllers watch API server
    
- Nobody except API server talks to etcd
    

---

# 🏗 Why This Design?

If scheduler accessed etcd directly:

❌ Tight coupling  
❌ Security issues  
❌ Multiple clients hitting etcd  
❌ Complex consistency

Instead Kubernetes follows:

> API Server = Single Source of Truth

---

# 🧠 Real Production Detail

Scheduler maintains:

- Local cache of Pods
    
- Local cache of Nodes
    
- Local cache of PVs, PVCs, etc.
    

It updates cache via Watch events.

So scheduling is very fast because it doesn't call API server every time.

---

# 🔄 After Scheduling

Once scheduler selects a node:

It sends PATCH request to API server:

{  
  "spec": {  
    "nodeName": "worker-node-2"  
  }  
}

Then:

1. API server validates update
    
2. Stores in etcd
    
3. Kubelet watching sees pod assigned to itself
    
4. Kubelet creates container
    

---

# 🔥 Clean Mental Model

Think of Kubernetes like this:

- etcd = database
    
- API server = gatekeeper + event broadcaster
    
- Scheduler = event listener + decision maker
    
- Kubelet = event listener + executor


# Pod Creation 

The API Server creates the Pod object, but the kubelet creates the Pod’s runtime environment and starts the containers via the container runtime.


In Kubernetes, **only the API Server creates resources**.

So the **Pod object is created by the API Server**, not by kubelet.

The kubelet **does not create Pods** — it **realizes (implements) the Pod that already exists in the API server**.

# Correct Flow of Pod Creation

When you run:

```bash
kubectl run nginx --image=nginx
```

the sequence is:

### 1️⃣ kubectl sends a request

`kubectl` sends an HTTP request to the **API Server**.


### 2️⃣ API Server creates the Pod object

The API server:

- validates the request
    
- creates the **Pod resource**
    
- stores it in **etcd**
    

Now the Pod exists in the cluster state.

Example:

```
STATUS = Pending
NODE = <none>
```


### 3️⃣ Scheduler assigns a node

The **kube-scheduler** watches for Pods that don't have a node.

It chooses a node and updates the Pod spec:

```
spec.nodeName = k8s-worker
```

---

### 4️⃣ kubelet watches the API Server

Every kubelet constantly watches the API server.

When kubelet on `k8s-worker` sees:

```
Pod assigned to k8s-worker
```

it starts the process of running it.

---

### 5️⃣ kubelet creates the **Pod runtime environment**


The kubelet:

- prepares the Pod sandbox
    
- configures networking (CNI)
    
- mounts volumes
    
- calls container runtime
    

Then it asks container runtime (containerd) to start containers.


### 6️⃣ Container Runtime starts containers

containerd:

- pulls the image
    
- creates container
    
- runs it
    

Now the Pod becomes:

```
STATUS = Running
```

---

# Correct Terminology

|Component|What it does|
|---|---|
|API Server|**Creates Pod resource**|
|etcd|Stores Pod state|
|Scheduler|Assigns node|
|kubelet|**Ensures Pod is running on that node**|
|container runtime|Runs containers|

---

# Key Kubernetes Principle

The kubelet is a **node agent that ensures desired state**.

It does **not create resources in Kubernetes**.

It only makes sure:

```
Pod assigned to this node → actually running
```



