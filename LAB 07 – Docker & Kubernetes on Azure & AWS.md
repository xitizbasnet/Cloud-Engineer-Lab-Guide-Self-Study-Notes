# LAB 07 – Docker & Kubernetes on Azure & AWS

> [!IMPORTANT]
> This lab demonstrates how to build, run, and manage containerized applications using **Docker**, **Podman**, **Buildah**, and **LXC**. It also covers deploying workloads to **Azure Kubernetes Service (AKS)** and configuring **Container Network Interface (CNI)** and **IP Address Management (IPAM)** using **Calico**.

---

# 📋 Lab Information

| Property | Value |
|----------|-------|
| **Lab ID** | LAB 07 |
| **Title** | Docker & Kubernetes on Azure/AWS |
| **Estimated Duration** | 3–4 Hours |
| **Difficulty** | Advanced |

---

# 🎯 Objective

Build and deploy containerized applications using **Docker**, **Podman**, **Buildah**, and **LXC**. Manage Kubernetes clusters on **Azure Kubernetes Service (AKS)** and **Amazon Elastic Kubernetes Service (EKS)**. Configure **Container Network Interface (CNI)** and **IP Address Management (IPAM)** for container networking. Deploy Docker and Kubernetes IPAM.

At the end of this lab, you will:

- Build and run Docker containers.
- Create rootless container images using Buildah and Podman.
- Manage Linux containers with LXC.
- Deploy applications to Azure Kubernetes Service (AKS).
- Configure Kubernetes networking using Calico CNI.
- Configure Docker custom bridge networking with IPAM.

---

# 📚 Prerequisites

Before starting this lab, ensure that you have:

- An active Azure or AWS subscription.
- A Linux (Ubuntu) virtual machine.
- Docker installation permissions (sudo access).
- Access to an AKS or Amazon EKS cluster.
- `kubectl` configured for cluster access.
- Internet connectivity for downloading packages and container images.

---

# 🛠️ Lab Tasks

---

# Step 1 — Docker: Build and Run Containers

Install Docker on Ubuntu.

## Install Docker

```bash
sudo apt update && sudo apt install docker.io -y
sudo systemctl enable docker && sudo usermod -aG docker $USER
```

---

## Build a Custom Docker Image

Create a Dockerfile.

```bash
cat > Dockerfile << 'EOF'
FROM ubuntu:22.04
RUN apt update && apt install nginx -y
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
EOF
```

Build the Docker image.

```bash
docker build -t mywebapp:v1 .
```

---

## Run the Container

```bash
docker run -d -p 8080:80 --name webapp mywebapp:v1
```

Verify the deployment.

```bash
docker ps && curl http://localhost:8080
```

> [!NOTE]
> The container exposes **port 80** internally and maps it to **port 8080** on the host.

---

# Step 2 — Podman & Buildah (Rootless Containers)

Install Podman and Buildah.

```bash
sudo apt install podman buildah -y
```

---

## Build an Image with Buildah (No Dockerfile Required)

```bash
buildah from ubuntu:22.04
buildah run ubuntu-working-container -- apt update
buildah run ubuntu-working-container -- apt install nginx -y
buildah commit ubuntu-working-container myapp-podman:v1
```

---

## Run the Image Using Podman

```bash
podman run -d -p 8081:80 myapp-podman:v1
```

Verify the running container.

```bash
podman ps
```

> [!TIP]
> Podman runs containers without requiring a central daemon and supports rootless execution for improved security.

---

# Step 3 — LXC Container Management

Install LXC.

```bash
sudo apt install lxc lxc-utils -y
```

---

## Create an LXC Container

```bash
sudo lxc-create -n webserver -t ubuntu
```

---

## Start the Container

```bash
sudo lxc-start -n webserver
```

---

## Install Nginx Inside the Container

```bash
sudo lxc-attach -n webserver -- apt install nginx -y
```

---

## List Running Containers

```bash
sudo lxc-ls --fancy
```

---

## Stop the Container

```bash
sudo lxc-stop -n webserver
```

---

# Step 4 — Kubernetes: Deploy on Azure Kubernetes Service (AKS)

Connect to your AKS cluster and deploy an application.

---

## Create a Namespace

```bash
kubectl create namespace production
```

---

## Deploy the Application

```bash
kubectl apply -f deployment-manifest.yaml
```

### Deployment Manifest

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: cloudlabregistry.azurecr.io/mywebapp:v1
        ports:
        - containerPort: 80
EOF
```

---

## Verify the Deployment

```bash
kubectl get pods -n production
```

---

## Expose the Deployment

```bash
kubectl expose deployment webapp -n production --type=LoadBalancer --port=80
```

> [!NOTE]
> Using the **LoadBalancer** service type creates an external endpoint for accessing the application.

---

# Step 5 — Docker & Kubernetes IPAM & CNI Configuration

Configure Kubernetes networking using **Calico**.

---

## Install Calico CNI

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

---

## Configure the Calico IPAM Pool

Apply the IP pool configuration.

```bash
calicoctl apply -f ippool.yaml
```

### IP Pool Configuration

```yaml
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
  name: default-ipv4-ippool
spec:
  cidr: 192.168.0.0/16
  ipipMode: Always
  natOutgoing: true
EOF
```

---

## Configure a Custom Docker Bridge Network

Create a custom bridge network with a defined subnet.

```bash
docker network create --driver bridge \
--subnet 172.20.0.0/16 \
--ip-range 172.20.10.0/24 \
--gateway 172.20.10.1 \
my-custom-net
```

> [!NOTE]
> Custom bridge networks allow containers to communicate within an isolated subnet while providing predictable IP address allocation.

---

# 🌐 Networking Overview

```text
Docker
└── Bridge Network
      ├── Subnet: 172.20.0.0/16
      ├── IP Range: 172.20.10.0/24
      └── Gateway: 172.20.10.1

Kubernetes
└── Calico CNI
      ├── Network Policies
      ├── IPAM
      └── CIDR: 192.168.0.0/16
```

---

# ✅ Verification Checklist

After completing this lab, verify that:

- [ ] Docker is installed successfully.
- [ ] Custom Docker image builds successfully.
- [ ] Docker container is running.
- [ ] Podman successfully runs the Buildah image.
- [ ] LXC container starts successfully.
- [ ] Nginx is installed inside the LXC container.
- [ ] Kubernetes namespace has been created.
- [ ] Deployment contains three running replicas.
- [ ] LoadBalancer service is created successfully.
- [ ] Calico CNI is installed.
- [ ] Calico IP Pool has been configured.
- [ ] Docker custom bridge network has been created.

---

# 💡 Best Practice Tips

> [!TIP]
> Always use **non-root users** inside containers. Add `USER appuser` in your Dockerfile after installing dependencies.

---

> [!TIP]
> Scan Docker images with `docker scout cves` or **Trivy** before pushing them to production registries.

---

> [!TIP]
> Configure **resource requests** and **resource limits** for every Kubernetes Pod to prevent noisy-neighbor issues within the cluster.

---

# 📖 Summary

In this lab, you completed the following tasks:

- Installed Docker on Ubuntu.
- Built and deployed a custom Docker container.
- Built container images using Buildah.
- Executed rootless containers with Podman.
- Managed Linux containers using LXC.
- Deployed an application to Azure Kubernetes Service (AKS).
- Configured Kubernetes networking with Calico CNI.
- Configured Calico IP Address Management (IPAM).
- Created a custom Docker bridge network.

---

# 📚 References

- Docker
- Podman
- Buildah
- Linux Containers (LXC)
- Kubernetes
- Azure Kubernetes Service (AKS)
- Amazon Elastic Kubernetes Service (EKS)
- Calico CNI
- IP Address Management (IPAM)
- Container Networking Interface (CNI)
