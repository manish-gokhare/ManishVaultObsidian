Below is a **90-Day Kubernetes Mastery Roadmap used by many senior Kubernetes administrators and SREs**.  
It focuses on **real operational skills**, not just theory. The roadmap progresses from **foundations → administration → production → expert-level troubleshooting**.

Since you already work as a **Hadoop Admin**, you will likely progress faster because you already understand:

- Distributed systems
    
- Cluster resource management
    
- Linux operations
    
- Networking basics
    

Your focus will be learning **Kubernetes abstractions and cluster internals**.

---

# 90-Day Kubernetes Mastery Roadmap (Administrator Track)

## Phase 1 — Core Foundations (Days 1–15)

Goal: Understand containers, kubectl, and basic workloads.

### Day 1 — Containers Fundamentals

Learn:

- Containers vs VMs
- Container runtime
- Images and layers
    

Practice

```bash
docker pull nginx
docker run -d -p 8080:80 nginx
docker ps
docker logs <container_id>
```

---

	### Day 2 — Kubernetes Overview
	
	Learn:
	
	- Control plane
	    
	- Worker nodes
	    
	- Kubernetes architecture
	    

Practice

Install kubectl

```bash
sudo apt install kubectl
```

Check version

```bash
kubectl version --client
```

---

### Day 3 — Local Cluster Installation

Use **Kind** or **Minikube**

Example using Kind:

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin
```

Create cluster

```bash
kind create cluster
```

Verify

```bash
kubectl cluster-info
kubectl get nodes
```

---

### Day 4 — kubectl Basics

Practice

```bash
kubectl get nodes
kubectl get pods
kubectl get namespaces
kubectl get all
```

Describe resources

```bash
kubectl describe node
```

---

### Day 5 — Pods

Create pod YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```

Apply

```bash
kubectl apply -f pod.yaml
```

---

### Day 6 — Labels & Selectors

Add labels

```yaml
metadata:
  labels:
    app: nginx
    env: prod
```

Query with label

```bash
kubectl get pods -l app=nginx
```

---

### Day 7 — ReplicaSets

Example

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
```

Apply

```bash
kubectl apply -f rs.yaml
```

---

### Day 8 — Deployments

Deployment example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
```

Scale

```bash
kubectl scale deployment nginx-deploy --replicas=5
```

---

### Day 9 — Rolling Updates

Update image

```bash
kubectl set image deployment/nginx-deploy nginx=nginx:1.25
```

Check rollout

```bash
kubectl rollout status deployment nginx-deploy
```

Rollback

```bash
kubectl rollout undo deployment nginx-deploy
```

---

### Day 10 — Services

Types:

- ClusterIP
    
- NodePort
    
- LoadBalancer
    

Create service

```bash
kubectl expose deployment nginx-deploy --type=NodePort --port=80
```

Check

```bash
kubectl get svc
```

---

### Day 11 — Namespaces

Create namespace

```bash
kubectl create namespace dev
```

Run pod inside namespace

```bash
kubectl run nginx --image=nginx -n dev
```

---

### Day 12 — ConfigMaps

Create configmap

```bash
kubectl create configmap app-config --from-literal=env=prod
```

View

```bash
kubectl get configmap
```

---

### Day 13 — Secrets

Create secret

```bash
kubectl create secret generic db-secret \
--from-literal=password=admin123
```

---

### Day 14 — Jobs & CronJobs

Job example

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: test-job
spec:
  template:
    spec:
      containers:
      - name: busybox
        image: busybox
        command: ["echo", "Hello Kubernetes"]
      restartPolicy: Never
```

---

### Day 15 — Resource Management

Set CPU & Memory

```yaml
resources:
  requests:
    cpu: "200m"
    memory: "256Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"
```

---

# Phase 2 — Cluster Administration (Days 16–40)

Goal: Become a **real Kubernetes administrator**

### Day 16 — Kubernetes Architecture Deep Dive

Learn:

- API server
    
- etcd
    
- scheduler
    
- controller manager
    

---

### Day 17 — kube-apiserver

Check pods

```bash
kubectl -n kube-system get pods
```

---

### Day 18 — etcd

Backup etcd

```bash
ETCDCTL_API=3 etcdctl snapshot save snapshot.db
```

Restore

```bash
etcdctl snapshot restore snapshot.db
```

---

### Day 19 — Scheduler

Check scheduler logs

```bash
kubectl logs -n kube-system kube-scheduler-<node>
```

---

### Day 20 — Controller Manager

Understand controllers:

- Replica controller
    
- Node controller
    
- Endpoint controller
    

---

### Day 21 — kubelet

Check service

```bash
systemctl status kubelet
```

---

### Day 22 — kube-proxy

View

```bash
kubectl -n kube-system get pods | grep proxy
```

---

### Day 23 — Node Maintenance

Drain node

```bash
kubectl drain node1 --ignore-daemonsets
```

Uncordon

```bash
kubectl uncordon node1
```

---

### Day 24 — Cluster Upgrade

Example

```bash
kubeadm upgrade plan
kubeadm upgrade apply v1.xx
```

---

### Day 25 — TLS & Certificates

Check certificates

```bash
kubeadm certs check-expiration
```

---

### Day 26 — RBAC

Create role

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: dev
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```

---

### Day 27 — Service Accounts

Create

```bash
kubectl create serviceaccount app-sa
```

---

### Day 28 — Network Policies

Example

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: allow-nginx
spec:
  podSelector:
    matchLabels:
      app: nginx
```

---

### Day 29 — Pod Security

Understand:

- privileged containers
    
- security contexts
    

Example

```yaml
securityContext:
  runAsUser: 1000
```

---

### Day 30–40 — Networking

Learn deeply:

- Kubernetes networking model
    
- CNI plugins
    
- Calico
    
- Flannel
    
- Cilium
    
- DNS
    
- Ingress
    

---

# Phase 3 — Storage (Days 41–55)

Learn storage architecture.

Topics:

- Persistent Volumes
    
- Persistent Volume Claims
    
- Storage Classes
    
- CSI drivers
    

Example PV

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-test
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
```

---

# Phase 4 — Production Operations (Days 56–75)

Learn real production operations.

Topics:

- Monitoring (Prometheus)
    
- Grafana
    
- Logging (ELK / Loki)
    
- Alerting
    
- Backup
    
- Disaster recovery
    

Example install Prometheus

```bash
helm install prometheus prometheus-community/prometheus
```

---

# Phase 5 — Expert Level (Days 76–90)

This is where **senior Kubernetes admins differentiate themselves**.

Topics:

### Advanced Troubleshooting

```bash
kubectl describe pod
kubectl logs
kubectl exec
```

---

### Debug containers

```bash
kubectl exec -it pod-name -- /bin/sh
```

---

### Node debugging

```bash
journalctl -u kubelet
```

---

### Control plane troubleshooting

```bash
kubectl get componentstatuses
```

---

### Performance tuning

Learn:

- scheduler tuning
    
- resource quotas
    
- autoscaling
    

---

# Final Skills After 90 Days

You should be comfortable with:

- Cluster installation (kubeadm)
    
- Troubleshooting nodes
    
- RBAC security
    
- Networking
    
- Storage
    
- Monitoring
    
- Production operations
    

This is basically **CKA + real-world SRE skills**.

---

# Recommended Daily Routine (Very Important)

Each day:

**20 min**

Concept reading

**40 min**

Hands-on labs

**30 min**

Break cluster intentionally

Example:

```bash
delete pod
delete service
kill kubelet
restart node
```

Learn how to fix.

---

# If you want, I can also give you the **Senior Kubernetes Administrator Skill Map (used by companies like Netflix, Google, and RedHat)** that shows exactly what separates:

- Beginner
    
- Kubernetes Engineer
    
- Senior Kubernetes Administrator
    
- Kubernetes Architect.