# Day 1 – Containers Fundamentals

## 1. What is a Container?

A **container** packages:

- Application
- Runtime
- Libraries
- Dependencies
- OS components

So the application runs **the same everywhere**.

Example structure:

```text
Container
 ├── Application
 ├── Libraries
 ├── Runtime
 └── Minimal OS
```

---

# Containers vs Virtual Machines

| Feature         | VM      | Container |
| --------------- | ------- | --------- |
| OS per instance | Yes     | No        |
| Boot time       | Minutes | Seconds   |
| Size            | GB      | MB        |
| Performance     | Slower  | Faster    |

Architecture comparison:

```text
VM

Hardware
   ↓
Hypervisor
   ↓
Guest OS
   ↓
App
```

```text
Containers

Hardware
   ↓
Host OS
   ↓
Container Runtime
   ↓
Containers
```

---

# Container Runtime

A **container runtime** runs containers.

Common runtimes:

- containerd (used by Kubernetes)
- CRI-O
- Docker

For learning we use **Docker**.

---

# Step 1 — Install Docker

On Ubuntu:

```bash
sudo apt update
sudo apt install docker.io -y
```

Enable Docker

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

Verify installation

```bash
docker version
```

Expected output:

```text
Client:
 Version: 24.x
Server:
 Engine: Docker
```

---

# Step 2 — Run Your First Container

Run a test container.

```bash
docker run hello-world
```

Output:

```text
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

What happened internally:

```text
Docker Client
      ↓
Docker pulls image
      ↓
Creates container
      ↓
Runs container
```

---

# Step 3 — Pull Container Images

Containers run from **images**.

Image = blueprint.

Example:

```bash
docker pull nginx
```

Check images

```bash
docker images
```

Example output

```text
REPOSITORY   TAG       IMAGE ID
nginx        latest    605c77e6
```

---

# Step 4 — Run Nginx Container

Run a web server container.

```bash
docker run -d -p 8080:80 nginx
```

Explanation:

```text
-d        run container in background
-p        map port
8080      host port
80        container port
```

Check running containers

```bash
docker ps
```

Example output

```text
CONTAINER ID   IMAGE   PORTS
abc123         nginx   0.0.0.0:8080->80/tcp
```

Open browser

```
http://localhost:8080
```

You will see **NGINX welcome page**.

---

# Step 5 — Explore Container

Get container ID

```bash
docker ps
```

Enter container

```bash
docker exec -it <container_id> bash
```

Example

```bash
docker exec -it abc123 bash
```

Inside container:

```bash
ls
```

Check OS

```bash
cat /etc/os-release
```

Exit container

```bash
exit
```

---

# Step 6 — Container Logs

View logs

```bash
docker logs <container_id>
```

Example

```bash
docker logs abc123
```

---

# Step 7 — Stop Container

Stop container

```bash
docker stop <container_id>
```

Example

```bash
docker stop abc123
```

Check again

```bash
docker ps
```

---

# Step 8 — Remove Container

Remove container

```bash
docker rm <container_id>
```

Example

```bash
docker rm abc123
```

---

# Step 9 — Run Interactive Container

Run Ubuntu container

```bash
docker run -it ubuntu bash
```

Inside container

```bash
apt update
apt install curl
```

Exit

```bash
exit
```

Container stops automatically.

---

# Step 10 — Container Lifecycle

```text
docker run
     ↓
Container Created
     ↓
Running
     ↓
Stopped
     ↓
Removed
```

Commands:

```bash
docker run
docker stop
docker start
docker rm
```

---

# Step 11 — Container Networking

Run nginx again

```bash
docker run -d -p 8080:80 nginx
```

Check networking

```bash
docker inspect <container_id>
```

---

# Step 12 — Clean Environment

Remove all containers

```bash
docker rm $(docker ps -aq)
```

Remove images

```bash
docker rmi nginx
```

---

# Your Practical Tasks (Important)

Complete these today.

### Task 1

Pull images

```bash
docker pull nginx
docker pull redis
docker pull alpine
```

---

### Task 2

Run nginx

```bash
docker run -d -p 8080:80 nginx
```

Check

```bash
docker ps
```

---

### Task 3

Enter container

```bash
docker exec -it <container_id> bash
```

---

### Task 4

Check logs

```bash
docker logs <container_id>
```

---

### Task 5

Stop and remove container

```bash
docker stop <container_id>
docker rm <container_id>
```

---

# Real Industry Insight

In Kubernetes:

```text
Pod
 └── Container
```

Kubernetes does **not run applications directly**.

It runs **containers inside pods**.

# Docker Containers – Administration Notes

## 1. Docker Architecture

Docker is a containerization platform used to build, ship, and run applications in isolated environments called **containers**.

### Components of Docker Architecture

```
Client
  │
  ▼
Docker CLI (docker commands)
  │
  ▼
Docker Daemon (dockerd)
  │
  ├── Container Management
  ├── Image Management
  └── Network & Volume Management
  │
  ▼
Container Runtime (containerd / runc)
  │
  ▼
Containers
```

### Key Components

**Docker Client**

- Command-line interface used by users.
- Sends commands to the Docker daemon.

Example:

```bash
docker run nginx
docker build .
docker ps
```

---

**Docker Daemon (dockerd)**

- Background service that manages Docker objects.
    
- Responsible for:
    
    - images
        
    - containers
        
    - networks
        
    - volumes
        

---

**Docker Images**

- Read-only templates used to create containers.
    

Example:

```bash
docker pull nginx
```

---

**Docker Containers**

- Running instance of an image.
    

Example:

```bash
docker run nginx
```

---

**Container Runtime**

Docker internally uses:

- containerd
    
- runc
    

These runtimes actually create the container processes.

---

## 2. Virtual Machines vs Containers

|Feature|Virtual Machine|Container|
|---|---|---|
|Virtualization Level|Hardware|OS Level|
|Boot Time|Minutes|Seconds|
|Size|GB|MB|
|Performance|Slower|Faster|
|OS Per Instance|Yes|No|

### VM Architecture

```
Hardware
   │
Hypervisor
   │
Guest OS
   │
Application
```

### Container Architecture

```
Hardware
   │
Host OS
   │
Container Runtime
   │
Containers
```

Containers share the **host kernel**, making them lightweight.

---

# 3. Build Custom Docker Container

## Step 1 – Create Project Directory

```bash
mkdir nginx-custom
cd nginx-custom
```

---

## Step 2 – Create Custom Web Page

```bash
nano index.html
```

Example content:

```html
<!DOCTYPE html>
<html>
<head>
<title>Custom Container</title>
</head>
<body>

<h1>Hello from my custom NGINX container</h1>

</body>
</html>
```

---

## Step 3 – Create Dockerfile

```bash
nano Dockerfile
```

Dockerfile:

```dockerfile
FROM nginx:latest

COPY index.html /usr/share/nginx/html/index.html
```

Explanation:

|Instruction|Purpose|
|---|---|
|FROM|Base image|
|COPY|Copy file into container|

---

## Step 4 – Build Image

```bash
docker build -t custom-nginx .
```

Verify:

```bash
docker images
```

Example:

```
REPOSITORY      TAG
custom-nginx    latest
nginx           latest
```

---

# 4. Run the Custom Container

Run container:

```bash
docker run -d -p 8080:80 custom-nginx
```

Explanation:

|Option|Meaning|
|---|---|
|-d|run in background|
|-p|port mapping|

---

Check running containers:

```bash
docker ps
```

Example output:

```
CONTAINER ID   IMAGE
123abc         custom-nginx
```

---

# 5. Container Troubleshooting Commands

### Check Running Containers

```bash
docker ps
```

---

### List All Containers

```bash
docker ps -a
```

---

### View Container Logs

```bash
docker logs <container_id>
```

Example:

```bash
docker logs 123abc
```

---

### Inspect Container

```bash
docker inspect <container_id>
```

---

### Check Container Resource Usage

```bash
docker stats
```

---

### Stop Container

```bash
docker stop <container_id>
```

---

### Remove Container

```bash
docker rm <container_id>
```

---

### Force Remove

```bash
docker rm -f <container_id>
```

---

# 6. Access the Container

## Method 1 – Web Browser

If port mapping is:

```bash
docker run -p 8080:80 custom-nginx
```

Access via browser:

```
http://localhost:8080
```

---

## Method 2 – Curl (CLI)

```bash
curl http://localhost:8080
```

---

## Method 3 – Enter Container Shell

```bash
docker exec -it <container_id> bash
```

Example:

```bash
docker exec -it 123abc bash
```

---

## Method 4 – Docker Attach

```bash
docker attach <container_id>
```

---

## Method 5 – Inspect Container Network

```bash
docker inspect <container_id>
```

Look for:

```
IPAddress
Ports
```

---

# 7. Create Docker Repository in DockerHub

Open DockerHub:

[https://hub.docker.com](https://hub.docker.com/)

Steps:

1. Login
    
2. Go to **Repositories**
    
3. Click **Create Repository**
    

Example:

```
Repository Name: custom-nginx
Namespace: manishgokhare
Visibility: Public
```

Repository created:

```
manishgokhare/custom-nginx
```

---

# 8. Tag Image and Push to DockerHub

## Tag Image

```bash
docker tag custom-nginx manishgokhare/custom-nginx:v1
```

Verify:

```bash
docker images
```

---

## Push Image

```bash
docker push manishgokhare/custom-nginx:v1
```

Example output:

```
layer1: pushed
layer2: pushed
```

---

## Verify Image

Pull from registry:

```bash
docker pull manishgokhare/custom-nginx:v1
```

Run:

```bash
docker run -p 8080:80 manishgokhare/custom-nginx:v1
```

---

# 9. Container Internals

Containers are implemented using Linux kernel features.

## 1. Namespaces

Namespaces isolate system resources.

Types:

|Namespace|Purpose|
|---|---|
|PID|process isolation|
|NET|network isolation|
|MNT|filesystem isolation|
|UTS|hostname isolation|
|IPC|interprocess communication|

Example:

```bash
lsns
```

---

## 2. cgroups (Control Groups)

Control resource limits for containers.

Resources controlled:

- CPU
    
- Memory
    
- Disk IO
    
- Network
    

Example:

```bash
docker stats
```

Example limit:

```bash
docker run -m 512m nginx
```

Limits container memory to 512MB.

---

## 3. OverlayFS

OverlayFS manages container filesystems using layers.

Image structure:

```
Base Layer
   │
Application Layer
   │
Writable Container Layer
```

Benefits:

- Faster builds
    
- Layer reuse
    
- Smaller images
    

Check layers:

```bash
docker history nginx
```

---

# Summary

Docker workflow:

```
Dockerfile
   │
docker build
   │
Docker Image
   │
docker run
   │
Container
   │
docker tag
   │
docker push
   │
DockerHub Registry
```

Key container technologies:

- Namespaces
    
- cgroups
    
- OverlayFS
    

These technologies make **containers lightweight and efficient**, forming the foundation for **Kubernetes container orchestration**.