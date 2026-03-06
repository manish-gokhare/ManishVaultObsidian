# 1️⃣ How kubectl Reaches the API Server

kubectl does NOT magically know the cluster.

It uses a config file called:

👉 **kubeconfig**

## kubeconfig File

Default location:

~/.kube/config

This file tells kubectl:

- Which cluster to talk to
- API Server address
- Authentication method
- Certificates / tokens




## Inside kubeconfig

It contains 3 key things:

##### 1. Cluster

API Server endpoint
Example:
server: https://10.0.0.1:6443
###### 2. User

Authentication details

Example:

- Client certificate
- Token
- OIDC login
##### 3. Context

Links:

Cluster + User + Namespace

kubectl uses context to decide:

👉 Where to send request  
👉 Who is sending request

If there are two API server then context will get updated.

## Flow

When you run:

kubectl get pods

kubectl:

1. Reads `~/.kube/config`
2. Finds API Server URL
3. Authenticates
4. Sends HTTPS request

kubectl → API Server:6443

So kubectl can run:

✔ From laptop  
✔ From jump host  
✔ From CI/CD  
✔ From inside cluster

As long as kubeconfig exists.



# What is Context in kubeconfig?

👉 **Context = Cluster + User + Namespace**

It tells kubectl:

> Which cluster should I talk to?  
> Which identity should I use?  
> Which namespace should I work in?

Instead of typing all this every time.

# Why Context Exists?

You may have:

- Multiple clusters (dev, staging, prod)
- Multiple users (admin, readonly)
- Multiple namespaces

Context lets you switch easily.

# Structure of kubeconfig

A kubeconfig file has 3 main sections:

```
clusters  
users  
contexts
```
Context connects the first two.

Example kubeconfig

```
clusters:  
- name: dev-cluster  
  cluster:  
    server: https://10.0.0.1:6443  
  
users:  
- name: dev-admin  
  user:  
    client-certificate: admin.crt  
    client-key: admin.key  
  
contexts:  
- name: dev-context  
  context:  
    cluster: dev-cluster  
    user: dev-admin  
    namespace: default

```

# What Context Means

Here:

dev-context =  
   dev-cluster  
 + dev-admin  
 + default namespace

So when kubectl uses this context:

👉 It knows everything it needs.

# Current Context

kubeconfig also has:

current-context: dev-context

kubectl always uses the current context.


# Example Usage

Run:

kubectl get pods

kubectl internally uses:

Cluster → dev-cluster  
User → dev-admin  
Namespace → default

No need to specify manually.


# Switching Context

To see contexts:

kubectl config get-contexts

To switch:

kubectl config use-context prod-context

Now kubectl talks to prod cluster.

---

# Without Context

You would need to type:

- API Server
- Certificate
- Namespace

every time.

Context saves you from that.


# Real-Life Analogy

Context = Login Profile

Like:

Browser profile decides:
- which account
- which settings
- which environment

# One-Line Summary

Context tells kubectl **which cluster, which user, and which namespace to use**.


# 2️⃣ How to Install kubectl on Linux

## Step 1 — Download kubectl

Run:

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

## Step 2 — Make Executable

chmod +x kubectl

## Step 3 — Move to PATH

sudo mv kubectl /usr/local/bin/


## Step 4 — Verify

kubectl version --client


# 3️⃣ Setup kubeconfig

To connect to cluster:

Copy admin kubeconfig (from control plane):

mkdir -p ~/.kube  
sudo cp /etc/kubernetes/admin.conf ~/.kube/config  
sudo chown $(id -u):$(id -g) ~/.kube/config

Now test:

kubectl get nodes


```
kubectl  
↓  
reads ~/.kube/config  
↓  
gets API Server endpoint  
↓  
authenticates  
↓  
HTTPS call to :6443
```



# 🚨 Reality First (Important)

RHEL 10 uses:

- Kernel 6.x
    
- nftables (NOT legacy iptables bridge mode)
    
- No `/proc/sys/net/bridge/*`
    

Because of this:

❌ Flannel = unreliable  
⚠️ Many old kubeadm guides break

So we will use:

✅ containerd  
✅ kubeadm  
✅ Calico CNI (modern + RHEL10 compatible)

This combination works without kernel hacks.

---

# 🧱 Architecture

|Node|Role|
|---|---|
|k8s-master|Control Plane|
|k8s-worker|Worker|

---

# 🔐 AWS Security Group

Create **one SG** → attach to both nodes

Inbound rules:

|Type|Port|Source|
|---|---|---|
|SSH|22|Your IP|
|TCP|6443|SG itself|
|TCP|10250|SG itself|
|TCP|30000-32767|Your IP|
|All TCP|0-65535|SG itself|
|All UDP|0-65535|SG itself|

---

# 🟢 STEP 1 — Node Preparation (Run on BOTH nodes)

## Set Hostname

Master:

```
hostnamectl set-hostname k8s-master
```


Worker:

```
hostnamectl set-hostname k8s-worker
```


---

## Update Hosts

Edit:

vi /etc/hosts

Add:

```
172.31.35.222 k8s-master.example.com k8s-master
172.31.39.82  k8s-worker.example.com k8s-worker
```



## Disable Swap

```
swapoff -a  
sed -i '/swap/d' /etc/fstab
```



## Enable IP Forwarding

```
cat <<EOF > /etc/sysctl.d/k8s.conf  
net.ipv4.ip_forward=1  
EOF  
  
sysctl --system
```
We enable: IP forwarding.

because:

👉 Kubernetes nodes must **route traffic between Pods**

Without IP forwarding, the Linux kernel behaves like a host — not a router.

Kubernetes needs the node to behave like a router.

# When IP Forwarding Becomes Required

IP forwarding is required when traffic must **leave one network interface and go to another**.

That happens when:

|Scenario|Needs IP Forwarding?|
|---|---|
|Pod → Pod (same node)|❌ No|
|Pod → Pod (different node)|✅ Yes|
|Pod → Service (ClusterIP)|✅ Yes|
|NodePort → Pod|✅ Yes|
|External → Pod|✅ Yes|

---


## Load Required Kernel Module

```
modprobe overlay  
echo overlay > /etc/modules-load.d/k8s.conf
```

(No br_netfilter needed on RHEL 10)

# 🧠 What is a Kernel Module?

A **kernel module** is like a plugin for the Linux kernel.

It adds features **without rebooting** or rebuilding the kernel.

Examples:

| Feature     | Module    |
| ----------- | --------- |
| Filesystems | overlay   |
| Networking  | vxlan     |
| Storage     | nfs       |
| Firewall    | nf_tables |
Modules live in:
```
/lib/modules/<kernel-version>/
```

The `overlay` module enables:

👉 **OverlayFS**

OverlayFS is a special filesystem that:

✔ Combines multiple directories  
✔ Creates a layered filesystem

This is the foundation of **containers**.

Containers don’t copy full OS filesystems.

Instead they use layers:

Example image:

```
nginx image
 ├ base OS
 ├ libraries
 ├ nginx
```

Each layer is read-only.

When container runs:

A writable layer is added on top.

OverlayFS merges them into one view.

# 🧱 Without OverlayFS

Container runtime (containerd) cannot:

❌ mount container filesystem  
❌ start containers  
❌ run pods

Kubernetes would fail with errors like:

overlayfs not supported

modprobe overlay - Loads the overlay filesystem into the running kernel.

# 📂 How OverlayFS Works (Conceptually)

Container runtime creates:

Lowerdir = image layers (read-only)  
Upperdir = container writable layer  
Workdir  = internal processing

Overlay merges them:

Merged View = Container Filesystem

# 🟢 STEP 2 — Install Container Runtime

## Install Dependencies

```
dnf install -y yum-utils device-mapper-persistent-data lvm2
```

---

## Add Docker Repo

```
dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
```

---

## Install containerd

```
dnf install -y containerd.io
```

---

## Configure containerd

mkdir -p /etc/containerd  
containerd config default > /etc/containerd/config.toml

Edit:

vi /etc/containerd/config.toml

Change:

SystemdCgroup = false

to:

SystemdCgroup = true

---

## Start containerd

systemctl enable --now containerd

---

# 🟢 STEP 3 — Install Kubernetes

## Add Repo

cat <<EOF > /etc/yum.repos.d/kubernetes.repo  
[kubernetes]  
name=Kubernetes  
baseurl=https://pkgs.k8s.io/core:/stable:/v1.30/rpm/  
enabled=1  
gpgcheck=1  
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.30/rpm/repodata/repomd.xml.key  
EOF

---

## Install

dnf install -y kubelet kubeadm kubectl  
systemctl enable --now kubelet

---

# 🟢 STEP 4 — Initialize Master

Run ONLY on master:

kubeadm init \  
--pod-network-cidr=192.168.0.0/16

(No ignore flags needed)

---

# 🟢 STEP 5 — Configure kubectl

mkdir -p $HOME/.kube  
cp /etc/kubernetes/admin.conf $HOME/.kube/config  
chown $(id -u):$(id -g) $HOME/.kube/config

---

# 🟢 STEP 6 — Install Calico Network

Run ONLY on master:

kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/calico.yaml

---

# 🟢 STEP 7 — Join Worker

On master:

kubeadm token create --print-join-command

Run output on worker.

---

# 🟢 STEP 8 — Verify

kubectl get nodes

Expected:

k8s-master   Ready  
k8s-worker   Ready

---

# 🟢 STEP 9 — Test Pod

kubectl run nginx --image=nginx  
kubectl get pods -o wide

You should now see:

IP = 192.168.x.x

---

# 🟢 STEP 10 — Test Service

kubectl expose pod nginx --type=NodePort --port=80  
kubectl get svc

Access:

http://<worker-public-ip>:NodePort

---

# 🎯 Why This Will NOT Break

|Component|Reason|
|---|---|
|containerd|upstream supported|
|Calico|nftables compatible|
|overlay only|no legacy kernel needed|

---

# 🟢 Master vs Worker

kubectl get nodes -o wide

Master shows:

control-plane

Worker shows:

<none>


- Pod will be sceduled on worker node.
- master node is tainted.

# 🧠 What Does “Master is Tainted” Mean?

In Kubernetes, a **taint** is a rule applied to a node that says:

> “Do NOT schedule Pods here — unless they explicitly tolerate this rule.”


# 🔎 What Exactly Is Applied?

On your master node, this taint exists:

node-role.kubernetes.io/control-plane:NoSchedule

You can verify:

kubectl describe node k8s-master | grep Taints

---

# 🧱 What is a Taint?

A taint has 3 parts:

key=value:effect

In your case:

|Part|Value|
|---|---|
|Key|node-role.kubernetes.io/control-plane|
|Effect|NoSchedule|

Effect = `NoSchedule` means:
