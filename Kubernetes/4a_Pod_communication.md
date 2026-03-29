

# Kubernetes Pod Networking – Clear Notes

---

# 1. Pod Networking Goal in Kubernetes

Kubernetes networking follows **three main rules**:

1. **Every Pod gets its own IP address**
    
2. **Pods can communicate with other Pods directly**
    
3. **Pods on different nodes can communicate without NAT**
    

Example:

```
Pod1 → Pod2
Pod1 → Pod on another node
```


# 2. Components Involved in Pod Networking

When a Pod is created, several Linux networking components are used.

|Component|Purpose|
|---|---|
|Pod Network Namespace|Isolated network stack for Pod|
|eth0|Network interface inside Pod|
|veth pair|Virtual cable connecting Pod to node|
|cni0 bridge|Virtual switch on node|
|Node routing|Routes packets between nodes|
|Node NIC (eth0)|Physical network interface|

---

# 3. Pod Network Namespace

Each Pod runs in its **own Linux network namespace**.

Inside a Pod you have:

```
lo
eth0
routing table
```

Example Pod IP:

```
10.244.0.144
```

From inside the Pod it looks like a **normal Linux machine**.

---

# 4. veth Pair (Virtual Ethernet Pair)

A **veth pair acts like a virtual network cable**.

```
Pod side            Host side
---------           ----------
eth0   <--------->  veth1234
```

Important property:

```
Packets entering one end exit the other end
```

So when Pod sends traffic:

```
Pod eth0 → veth1234
```

The packet enters the **host network namespace**.

---

# 5. CNI Plugin

The **CNI plugin configures networking for Pods**.

Examples of CNI plugins:

- Flannel
    
- Calico
    
- Cilium
    
- Weave
    

When Kubernetes creates a Pod, the **CNI plugin performs these tasks**:

1. Create network namespace
    
2. Create veth pair
    
3. Move one end to Pod
    
4. Attach host end to bridge
    
5. Assign Pod IP
    
6. Configure routing
    

---

# 6. Pod IP Assignment

The **CNI plugin assigns the Pod IP**.

Example Pod network range:

```
10.244.0.0/16
```

Example Pod:

```
Pod IP = 10.244.0.144
```

The IP is assigned to:

```
eth0
```

Inside the Pod this command is executed internally:

```
ip addr add 10.244.0.144/24 dev eth0
```

---

# 7. Linux Bridge (cni0)

The **CNI plugin creates a Linux bridge** called:

```
cni0
```

The bridge acts like a **virtual switch inside the node**.

It connects all Pods running on that node.

Example node:

```
Node
--------------------------------

           cni0
        /    |    \
     veth1 veth2 veth3
       |     |     |
     Pod1  Pod2  Pod3
```

Each Pod's **host-side veth interface is attached to the bridge**.

Command internally:

```
ip link set veth1234 master cni0
```

---

# 8. Bridge Gateway

The bridge usually has an IP address.

Example:

```
cni0 = 10.244.0.1
```

This acts as the **gateway for Pods on that node**.

Inside the Pod routing table:

```
default via 10.244.0.1 dev eth0
```

Meaning:

```
Unknown traffic → send to bridge
```

---

# 9. Pod to Pod Communication (Same Node)

Example:

```
Pod1 → Pod2
```

Packet path:

```
Pod1 eth0
   ↓
veth pair
   ↓
cni0 bridge
   ↓
veth pair
   ↓
Pod2 eth0
```

The bridge switches packets based on **MAC addresses**.

---

# 10. Pod to Pod Communication (Different Nodes)

Example:

```
Pod1 (Node1) → Pod2 (Node2)
```

Packet path:

```
Pod1 eth0
   ↓
veth
   ↓
cni0 bridge
   ↓
Node1 routing
   ↓
Node1 eth0
   ↓
Network
   ↓
Node2 eth0
   ↓
Node2 routing
   ↓
cni0 bridge
   ↓
veth
   ↓
Pod2 eth0
```

---

# 11. Role of the CNI Bridge

The **CNI bridge has two main purposes**:

1. Connect Pods on the **same node**
    
2. Connect Pods to the **node network**
    

It behaves like a **virtual Ethernet switch**.

---

# 12. Number of Bridges Per Node

Usually:

```
1 bridge per node
```

Example:

```
cni0
```

If a node runs **20 Pods**:

```
1 bridge
20 veth interfaces
```

---

# 13. Real Node Network Layout

Example worker node:

```
Node
------------------------------------------------

cni0 (10.244.0.1)
 │
 ├── vethA → Pod1 (10.244.0.10)
 ├── vethB → Pod2 (10.244.0.11)
 └── vethC → Pod3 (10.244.0.12)
```

---

# 14. Important Clarification

The **bridge does NOT assign Pod IPs**.

|Component|Responsibility|
|---|---|
|CNI plugin|Assign Pod IP|
|Pod eth0|Holds Pod IP|
|veth pair|Connect Pod to node|
|cni0 bridge|Connect Pods together|

---

# 15. Simple Mental Model

Think of this:

```
Pods = computers
cni0 = network switch
node routing = router
cluster network = internet
```

Flow:

```
Pod → cable → switch → router → network → router → switch → cable → Pod
```

---
