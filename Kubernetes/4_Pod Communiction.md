
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


# 1️⃣ Kubernetes Networking Rule

Kubernetes guarantees:

> **Every Pod can communicate with every other Pod using its Pod IP without NAT.**

Example cluster:

|Pod|Node|IP|
|---|---|---|
|Pod1|Worker1|192.168.254.129|
|Pod2|Worker2|192.168.126.5|

Pod1 wants to talk to Pod2.

```
# Go inside the pod
#execute 
curl http://192.168.126.5
```



**NAT (Network Address Translation)** means:

A device changes the source or destination IP address of a packet while forwarding it.

Example outside Kubernetes:

```
Laptop (192.168.1.10)  
  │  
Router performs NAT  
  │  
Internet sees: 203.0.113.25
```

The router replaces:

Source IP: 192.168.1.10

with

Source IP: 203.0.113.25

So the original IP is **hidden**.


**Kubernetes avoids NAT between Pods.**

If Pod1 talks to Pod2:

Pod1 IP: 192.168.254.129  
Pod2 IP: 192.168.126.5

The packet is sent exactly like this:

Source IP: 192.168.254.129  
Destination IP: 192.168.126.5

And **these addresses remain unchanged** across nodes.

Kubernetes requires:

1️⃣ Pods have unique IPs  
2️⃣ Pods can reach each other directly  
3️⃣ No NAT between Pods

**Without NAT means:**

> The Pod’s original IP address is preserved during communication.

Pods talk to each other **directly using their real IPs**.


# 1️⃣ Kubernetes Creates a Pod

When Kubernetes creates a Pod:

1. Scheduler selects a node.
    
2. Kubelet starts the container using a container runtime (containerd / CRI-O).
    

At this moment the container **does not yet have networking**.

---

# 2️⃣ Kubernetes Uses CNI to Configure Networking

Kubernetes calls a **CNI plugin (Container Network Interface)**.

Examples:

- Calico
- Flannel
- Cilium
- Weave

The CNI plugin sets up Linux networking objects.

---

# 3️⃣ A Network Namespace is Created

Every Pod runs inside its own **Linux network namespace**.

A **network namespace** is like a **separate network stack**.

Inside the Pod namespace you get:
```
eth0  
lo  
routing table  
iptables rules
```

So every Pod behaves like a **small Linux machine**.

# 4️⃣ Kubernetes Creates a veth Pair

The CNI plugin creates a **veth pair**.

A **veth pair = two virtual Ethernet cables connected together**.

```
Pod side        Host side
--------        ---------
eth0  <------->  vethXYZ
```

Flow:

```
Pod namespace
   eth0
     │
     │ veth pair
     │
Host namespace
   vethXYZ
```
Packets leaving the Pod go through this cable.

So Kubernetes connects Pod network namespace with host network namespace using a **veth pair**.

Each node has Pod network namespace and host network namespace.

```
Node (minikube)
│
├── Pod network namespace
│     └── eth0
│
└── Host network namespace
      └── veth1234
```


The plugin runs a script that does something like:

```
ip link add veth1234 type veth peer name eth0
```

Then it:

1️⃣ moves `eth0` into the Pod namespace  
2️⃣ keeps `veth1234` in the host namespace

```
Pod namespace             Host namespace
---------------           ----------------
eth0  <──────────────>   veth1234
```

But `veth1234` is just sitting on the host or the node.

It needs to connect to other Pods. 
This is where the **Linux bridge** comes in.

The **CNI plugin creates a Linux bridge** called:

```
cni0
```

A **Linux bridge behaves exactly like a network switch**.

Think of it like this: Virtual switch.
It connects multiple interfaces together.

The **CNI bridge acts like a virtual network switch inside the worker node**.

Its job is to **connect all Pods running on the same node**.

Without the bridge, each Pod would have an isolated veth connection with nowhere to send packets.

If If Pod1 sends a packet to Pod2 on the same worker node:

```
Pod1 eth0
   ↓
vethA
   ↓
cni0 bridge
   ↓
vethB
   ↓
Pod2 eth0
```


When Pod Talks to a Pod on a DIFFERENT Node

```
Pod1 (Node1) → Pod2 (Node2)
```

Packet Flow:

```
Pod1 eth0
   ↓
veth pair
   ↓
cni0 bridge
   ↓
Node1 routing
   ↓
Node1 eth0 (physical NIC)
   ↓
Network
   ↓
Node2 eth0
   ↓
Node2 routing
   ↓
cni0 bridge
   ↓
veth pair
   ↓
Pod2 eth0
```

The **bridge is not responsible for cross-node networking**
The **node routing + CNI networking handles that**

# What Actually Sends Traffic Between Nodes?

That depends on the **CNI plugin**.

Examples:

|CNI Plugin|Method|
|---|---|
|Flannel|VXLAN overlay|
|Calico|BGP routing|
|Cilium|eBPF|
|AWS VPC CNI|VPC routing|

These systems move packets **between nodes**.

The bridge only connects Pods **to the node network**.


# 5️⃣ Pod Gets an IP Address

The CNI plugin performs several tasks:

1. Create the **veth pair**
    
2. Move one end to the **Pod network namespace**
    
3. Attach the other end to the **bridge (`cni0`)**
    
4. **Assign the Pod IP**
    
5. Configure routing

The CNI plugin assigns the Pod an IP.

```
Pod IP: 10.244.1.5
```

Important Kubernetes rule:

**Every Pod gets a unique IP across the cluster.**

This means:

- Pods communicate directly
    
- No NAT between Pods

# 6️⃣ Host Connects Pod to Cluster Network

Now the host must connect the Pod to other Pods.

This depends on the CNI plugin.

Common approaches:

### Bridge networking

Linux bridge is created:

```
cni0
```

```
Pod eth0
   │
veth pair
   │
Linux Bridge (cni0)
   │
Host network
   │
Other nodes
```

A Pod gets an IP like 10.244.0.144

But the Pod lives inside a **network namespace**,isolated from the host.

How can packets leave the Pod and reach the rest of the cluster?

- Pod has its own interface (eth0)

```
Pod namespace
------------------
eth0 → 10.244.0.144
lo
------------------
```

But **eth0 is not connected to anything yet**.

When Pod Talks to a Pod on a DIFFERENT Node

```
Pod1 (Node1) → Pod2 (Node2)
```

Packet Flow:

```
Pod1 eth0
   ↓
veth pair
   ↓
cni0 bridge
   ↓
Node1 routing
   ↓
Node1 eth0 (physical NIC)
   ↓
Network
   ↓
Node2 eth0
   ↓
Node2 routing
   ↓
cni0 bridge
   ↓
veth pair
   ↓
Pod2 eth0
```

The **bridge is not responsible for cross-node networking**
The **node routing + CNI networking handles that**
