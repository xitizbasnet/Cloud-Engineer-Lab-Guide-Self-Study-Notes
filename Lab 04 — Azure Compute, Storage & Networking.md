# Lab 04 — Azure Compute, Storage & Networking

> **Difficulty:** 🟡 Intermediate

> **Estimated Duration:** ⏱️ 3–4 Hours

---

# 📖 Overview

This lab provides hands-on experience provisioning and managing core Microsoft Azure infrastructure services.

You will deploy and configure:

* Azure Virtual Machines (VMs)
* Azure Virtual Networks (VNets)
* Network Security Groups (NSGs)
* Azure Storage Accounts
* Azure Blob Storage
* Azure File Shares
* Azure Container Registry (ACR)
* Azure Container Instances (ACI)
* Azure Kubernetes Service (AKS)

You will also perform Azure subscription capacity planning, quota analysis, and budget management.

---

# 🎯 Objective

Provision and manage:

* Azure Virtual Machines
* Storage Accounts
* Virtual Networks
* Network Security Groups (NSGs)
* Azure Container Registry (ACR)
* Azure Container Instances (ACI)
* Azure Kubernetes Service (AKS)

Perform:

* Capacity Planning
* Azure Subscription Quota Analysis

---

# Exercise 1 — Create a Virtual Network and Subnets

## Step 1: Create the Virtual Network

Create a Virtual Network with an application subnet.

```bash
az network vnet create \
--resource-group cloud-lab-rg \
--name cloud-lab-vnet \
--address-prefix 10.10.0.0/16 \
--subnet-name app-subnet \
--subnet-prefix 10.10.1.0/24
```

---

## Step 2: Create the Database Subnet

```bash
az network vnet subnet create \
--resource-group cloud-lab-rg \
--vnet-name cloud-lab-vnet \
--name db-subnet \
--address-prefix 10.10.2.0/24
```

> [!NOTE]
> Separating application and database workloads into dedicated subnets improves network segmentation and security.

---

# Exercise 2 — Deploy a Linux Virtual Machine and Configure NSGs

## Step 1: Deploy an Ubuntu Virtual Machine

```bash
az vm create \
--resource-group cloud-lab-rg \
--name cloud-linux-vm \
--image UbuntuLTS \
--admin-username azureuser \
--generate-ssh-keys \
--vnet-name cloud-lab-vnet \
--subnet app-subnet \
--size Standard_B2s
```

---

## Step 2: Open SSH Access

```bash
az vm open-port \
--port 22 \
--resource-group cloud-lab-rg \
--name cloud-linux-vm
```

---

## Step 3: Configure Network Security Group Rules

Create custom **NSG rules** to allow SSH (TCP 22) access **only from known or trusted IP addresses**.

> [!IMPORTANT]
> Avoid exposing SSH to the public internet. Restrict inbound management access whenever possible.

---

# Exercise 3 — Azure Storage Account: Blob Storage and File Shares

## Step 1: Create a Storage Account

```bash
az storage account create \
--name cloudlabstorage2024 \
--resource-group cloud-lab-rg \
--sku Standard_LRS \
--kind StorageV2
```

---

## Step 2: Create a Blob Container

```bash
az storage container create \
--name app-data \
--account-name cloudlabstorage2024
```

---

## Step 3: Create an Azure File Share

```bash
az storage share create \
--name shared-files \
--account-name cloudlabstorage2024 \
--quota 50
```

---

# Exercise 4 — Azure Container Registry (ACR) and Azure Container Instances (ACI)

## Step 1: Create an Azure Container Registry

```bash
az acr create \
--resource-group cloud-lab-rg \
--name cloudlabregistry \
--sku Basic
```

---

## Step 2: Authenticate with ACR

```bash
az acr login --name cloudlabregistry
```

---

## Step 3: Tag and Push the Docker Image

```bash
docker tag myapp cloudlabregistry.azurecr.io/myapp:v1

docker push cloudlabregistry.azurecr.io/myapp:v1
```

---

## Step 4: Deploy an Azure Container Instance

```bash
az container create \
--resource-group cloud-lab-rg \
--name myapp-container \
--image cloudlabregistry.azurecr.io/myapp:v1 \
--cpu 1 \
--memory 1.5 \
--dns-name-label myapp-lab
```

---

# Exercise 5 — Deploy Azure Kubernetes Service (AKS)

## Step 1: Create an AKS Cluster

```bash
az aks create \
--resource-group cloud-lab-rg \
--name cloud-lab-aks \
--node-count 2 \
--node-vm-size Standard_B2s \
--enable-managed-identity \
--attach-acr cloudlabregistry
```

---

## Step 2: Configure kubectl Access

```bash
az aks get-credentials \
--resource-group cloud-lab-rg \
--name cloud-lab-aks
```

---

## Step 3: Verify Cluster Nodes

```bash
kubectl get nodes
```

---

## Step 4: Deploy the Application

```bash
kubectl apply -f deployment.yaml
```

> [!NOTE]
> The `deployment.yaml` manifest defines the Kubernetes resources that will be deployed to the AKS cluster.

---

# Exercise 6 — Azure Budget Planning and Quota Analysis

## Step 1: Review Azure Subscription Quotas

```bash
az vm list-usage \
--location eastus \
--output table
```

---

## Step 2: Create a Monthly Budget

```bash
az consumption budget create \
--budget-name MonthlyBudget \
--amount 500 \
--time-grain Monthly \
--start-date 2024-01-01 \
--end-date 2024-12-31
```

---

## Step 3: Review Cost Analysis

Open the Azure portal and navigate to:

```text
Cost Management → Cost Analysis
```

Use the Cost Analysis dashboard to review:

* Resource costs
* Service utilization
* Spending trends
* Budget status

---

# ✅ Best Practices

> [!TIP]
> Always use **Managed Identities** instead of service principal secrets for authentication between **Azure Kubernetes Service (AKS)** and **Azure Container Registry (ACR)**.

---

> [!TIP]
> Enable **Microsoft Defender for Containers** on AKS to detect runtime threats, vulnerable container images, and security misconfigurations.

---

> [!TIP]
> Configure Azure budget alerts at multiple thresholds, such as:
>
> * **50%** of the monthly budget
> * **80%** of the monthly budget
> * **100%** of the monthly budget
>
> Early notifications help prevent unexpected cloud spending.

---

# ✔️ Lab Summary

In this lab, you completed the following tasks:

* Created Azure Virtual Networks and subnets.
* Deployed an Ubuntu virtual machine.
* Configured Network Security Group (NSG) rules.
* Created an Azure Storage Account.
* Provisioned Blob Storage and Azure File Shares.
* Created an Azure Container Registry (ACR).
* Pushed a Docker image to ACR.
* Deployed an Azure Container Instance (ACI).
* Created and configured an Azure Kubernetes Service (AKS) cluster.
* Connected to the AKS cluster using `kubectl`.
* Deployed a Kubernetes workload.
* Reviewed Azure subscription quotas.
* Configured a monthly Azure budget.
* Explored Azure Cost Management for spending analysis.
