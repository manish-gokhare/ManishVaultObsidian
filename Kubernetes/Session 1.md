
K8s Architecture:
![[Pasted image 20260224192823.png]]


```
kubectl run firstpod --image=httpd.  # Create Pod
```

K8s says that  I am going to manage my own resource  and one of the smallest resource is Pod. Pod is the resource I know.  You can use any container run time interface (CRI) (containerd, docker, crio) to create the container inside the Pod. 
Creation of container is a responsibility of container runtime.

K8s does that to extract the backend container definition.

From Kubernetes' perspective: The smallest deployable thing is NOT a container — it is a Pod.

So Kubernetes says:

👉 “You give me a Pod spec.”  
👉 “I will make sure it runs.”



# 🏗 Kubernetes Architecture Principle

Kubernetes only understands:

Pod
 ├── Containers
 ├── Volumes
 ├── Network
 └── Metadata


# 🔄 Flow on Worker Node

When a Pod is assigned to a node:

## Step 1 — Kubelet receives Pod defination

From API server - kubelet on worker node  pulls the Pod defination from API server.

Now kubelet must prepare the Pod environment.
Kubelet reads Pod spec.

```
containers:
  - image: nginx
```

## Step 2 — Kubelet talks via CRI to Runtime to create Pod Sandbox

Via CRI:

```
kubelet → CRI → container runtime
CreatePodSandbox()
```


Runtime creates:

👉 Pause container

This represents:

- Network namespace
- IPC namespace
Pod now has a sandbox.

## Step 3 — Runtime triggers CNI

After sandbox creation, runtime calls:

👉 CNI plugin

to setup:
- Pod IP
- Routing
- veth pair
- Network namespace

So:
```
container runtime → CNI # This is networking only.
```

Now:

👉 Pod has an IP

## Step 4 — Kubelet sets up Storage (Kubelet → CSI)

Volumes are NOT runtime responsibility.

Before containers start, storage must be ready.

Kubelet directly calls CSI driver

```
kubelet → CSI driver
```

Kubelet directly calls CSI driver to:

- Attach volume
- Mount volume
- Prepare it

## Step 5 — Kubelet asks Runtime to create containers

Now everything is ready.

Kubelet does NOT talk runtime directly.

It calls CRI methods like:
```
CreatePodSandbox()
CreateContainer()
StartContainer()
```

```
kubelet → CRI → runtime
CreateContainer()
StartContainer()
```

Runtime receives the request because:

👉 It implements CRI


Then runtime:
- pulls image
- creates container 
- starts process

# 🧠 First — Pod is NOT Just Containers

A Pod is actually an **environment** in which containers run.

Think of it as:

> A mini logical machine.

Inside that environment, Kubernetes must prepare:

- Network
- Storage
- Identity (metadata)
- Then run containers

Containers are just the last step.


# 🧩 What is CRI?

CRI is:

> A **standard API (interface)** that Kubernetes uses to talk to container runtimes.

It defines:

- How to create a container
- How to start/stop it   
- How to create a Pod sandbox

It does NOT run containers.

It only defines:

👉 _How kubelet should communicate_


# ⚙️ What is Container Runtime?

Container Runtime is:

> The actual software that runs containers.

Examples:

- containerd
- CRI-O

They:

✔ Pull images  
✔ Create containers  
✔ Start processes  
✔ Manage namespaces

Important:

CRI is NOT a separate tool you install.

It is:

👉 Implemented by the runtime.

Example:

- containerd has CRI plugin
    
- CRI-O is built for CRI

👉 **CRI is a part of the Runtime**

# 🏗 Real Structure

A container runtime (like containerd or CRI-O) contains:

Container Runtime  
 ├── CRI implementation  👈 (for Kubernetes)  
 ├── Image handling  
 ├── Container execution  
 ├── Snapshotting  
 ├── Storage mgmt  
 ├── Networking hooks  
 └── Low-level runtime (runc)

So CRI is just:

👉 The Kubernetes-facing API layer inside the runtime

# 🔌 Runtime Does Much More Than CRI

CRI only defines how Kubernetes talks to runtime.

But runtime also:

- Pulls images
    
- Unpacks layers
    
- Manages filesystem
    
- Talks to runc
    
- Creates namespaces
    
- Runs processes
    

All of this is outside CRI.

# 📌 Example: containerd

containerd has:

- Core engine
    
- Image manager
    
- Snapshotter
    
- Task manager
    
- Runtime (runc)
    
- **CRI plugin**
    

So:

CRI = plugin inside runtime


Kubernetes understands the **Pod abstraction**  
and uses:

- CRI to run containers
    
- CNI to provide networking
    
- CSI to attach storage





# Kubernetes Pod Creation Flow (kubectl → kubelet)

## 1. kubectl

- `kubectl` is a CLI (Command Line Interface)
- It can run **from anywhere** (inside or outside cluster)
- Only requirement → connectivity to **API Server**

Example:

```bash
kubectl run first-pod --image=httpd
```

kubectl sends an **HTTPS REST request** to:

```
API Server :6443
```


## 2. API Server

- API Server is the **entry point** of Kubernetes
- All cluster communication goes through it
- Runs on **Control Plane (Master) Node**
- Default port: **6443**

When request comes:

API Server:

- Authenticates
- Authorize
- Validates
- Runs Admission Controllers

Then stores the **Pod object in etcd**

## 3. etcd

- etcd is the **database of Kubernetes**
- Stores desired state of cluster


Example stored Pod object:

```
Pod Name: first-pod
Image: httpd
NodeName: ""
Status: Pending
Namespace: default
Pod IP: ""
```

Most fields are empty initially.

## 4. Kubernetes is Event-Based (Pull Model)

Other components do NOT get pushed updates.

They **watch API Server** continuously.

Components watching:

- Scheduler
- Controller Manager
- Kubelete

They use:

```
Watch API
```

---

## 5. Scheduler

Scheduler watches for:

```
Pods where spec.nodeName = ""
```

Meaning → Pod not assigned to any node.

Scheduler selects best node based on:

- CPU
- Memory
- Node health
- Constraints

Example decision:

```
worker-2 selected
```

Scheduler updates Pod via API Server.

```
spec.nodeName = worker-2
```

API Server stores update in etcd.


## 6. Kubelet

- Kubelet runs on every worker node
- It watches API Server

Kubelet on `worker-2` sees:

```
Pod assigned to me
```

Now kubelet becomes responsible for execution.

## 7. Pod Creation on Worker Node

Kubelet orchestrates the creation.

### Step 1 — Create Pod Sandbox

Kubelet calls Runtime via CRI:

```
CreatePodSandbox()
```

This creates:

- Pause container
- Network namespace
### Step 2 — Networking (CNI)

Runtime invokes CNI plugin.

CNI:

- Assigns Pod IP
- Configures networking

### Step 3 — Storage (CSI)

Kubelet calls CSI driver.

CSI:

- Attaches volume
- Mounts volume

### Step 4 — Create Containers

Kubelet again uses CRI:

```
CreateContainer()
StartContainer()
```

Runtime pulls image and runs container.

## 8. Pod is Running

Now Pod is alive with:

- IP assigned
- Container running

## 9. Kubelet Updates Pod Status

Kubelet sends status back to API Server:

```
Pod Status = Running
Pod IP assigned
Container state updated
```

API Server stores this in etcd.


## 10. Final Flow

```
kubectl
   ↓
API Server
   ↓
etcd
   ↓
Scheduler (assign node)
   ↓
API Server updates etcd
   ↓
Kubelet watches
   ↓
Create Sandbox
   ↓
CNI assigns IP
   ↓
CSI mounts storage
   ↓
CRI → Runtime creates container
   ↓
Kubelet updates status
   ↓
API Server stores in etcd
```

## Key Concepts

|Component|Role|
|---|---|
|kubectl|Sends request|
|API Server|Gateway|
|etcd|Source of truth|
|Scheduler|Assigns node|
|Kubelet|Executes Pod|
|CNI|Networking|
|CSI|Storage|
|CRI|Runtime interface|

## Golden Rule

API Server = Gateway  
etcd = Truth  
Scheduler = Decision  
Kubelet = Executor


# Kubernetes Controller Manager — Role in Reliability

Yes ✅ — Controller Manager is the component responsible for:

👉 Fault tolerance  
👉 High Availability  
👉 Scaling  
👉 Upgrades

But it does this through **different controllers inside it**

## What is Controller Manager?

Controller Manager is the **brain that maintains the desired state** of the cluster.

It runs on the **Control Plane**.

While:

- Scheduler decides _where_
    
- Kubelet executes _how_
    

👉 Controller Manager ensures:

> What you asked for is continuously maintained.

# What is a Controller?

A Controller is a **control loop** that:

1. Watches API Server
    
2. Compares:
    

Desired State vs Current State

3. Takes action if mismatch exists
    

This is called:

👉 Reconciliation Loop

# Controller Manager Runs Many Controllers

Controller Manager is not one controller.

It is a collection of controllers.

## 1. ReplicaSet Controller

Ensures: Required number of Pods are running

Example:

Desired: replicas = 3

Current: only 2 running

Action:

👉 Creates 1 new Pod


## 2. Deployment Controller

Manages:

- Rolling updates
    
- Rollbacks
    
- ReplicaSets
    

Ensures application versioning works.
## 3. Node Controller

Monitors node health.

If node stops responding:

- Marks node NotReady
    
- Evicts Pods after timeout


## 4. Job Controller

Ensures:

Task runs to completion

Example:

Backup job

Runs once and stops.

## 5. StatefulSet Controller

Maintains:

- Stable identity
    
- Ordered deployment
    

Used for:

- Databases

## 6. Service Account Controller

Creates:

- Default service accounts

## 7. Namespace Controller

Ensures:

- Namespace lifecycle is maintained


## What is kube-proxy?

kube-proxy is a **network component** that runs on **every worker node**.

Its job is to make:

👉 **Services reachable**

Pods are dynamic.  
They die and get recreated.

But Services provide:

> Stable access to Pods

kube-proxy makes that possible.

# Why Do We Need kube-proxy?

Pods:

- Have dynamic IPs
    
- Come and go
    

If a Pod dies → IP changes

But applications need stable communication.

```
Frontend → Backend
```
Frontend should not care:

- Which Pod?
    
- Which IP?
It should call:

```
backend-service
```
kube-proxy makes this routing happen.

# What kube-proxy Does

kube-proxy:

✔ Watches API Server  
✔ Detects Service & Endpoint changes  
✔ Updates node networking rules

It creates:

👉 Traffic forwarding rules

So:

Service IP → Pod IP

# kube-proxy Works in Pull Model

Like kubelet, it: Watches API Server

for:

- Services
- Endpoints

# Traffic Flow Example

You create:

Service: backend-svc  
ClusterIP: 10.96.10.5

Pods behind it:

Pod1 → 192.168.1.10  
Pod2 → 192.168.1.11

kube-proxy creates rules like:

10.96.10.5 → 192.168.1.10  
10.96.10.5 → 192.168.1.11

Now requests to Service IP get load-balanced.





