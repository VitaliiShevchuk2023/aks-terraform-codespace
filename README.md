
# 🚀 AKS Cluster Deployment with Terraform in GitHub Codespaces

> Deploy an Azure Kubernetes Service (AKS) cluster using Terraform in GitHub Codespaces and run a multi-container retail store demo application.

---

## 📋 Table of Contents

- [Project Overview](#project-overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [GitHub Codespace Setup](#github-codespace-setup)
- [Infrastructure Deployment](#infrastructure-deployment)
- [Application Deployment](#application-deployment)
- [Verification and Access](#verification-and-access)
- [Cleanup](#cleanup)
- [Terraform Configuration Files](#terraform-configuration-files)
- [Important Disclaimers](#important-disclaimers)
- [Troubleshooting](#troubleshooting)
- [Additional Resources](#additional-resources)

---

## 📖 Project Overview

This project demonstrates how to deploy an **Azure Kubernetes Service (AKS)** cluster using **Terraform** inside **GitHub Codespaces** and run a multi-container demo application that simulates a retail store scenario (Azure Store Demo).

### What You Get

| Component | Description |
|-----------|-------------|
| AKS Cluster | Managed Kubernetes cluster in Azure |
| Terraform IaC | Infrastructure as Code for reproducible deployments |
| Store Front | Web UI for browsing products and placing orders |
| Product Service | Microservice displaying product information |
| Order Service | Microservice for processing orders |
| RabbitMQ | Message queue for asynchronous order processing |

> ⚠️ **Notice:** This project is intended **for evaluation and learning purposes only** — not for production use.

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────┐
│                  Azure Kubernetes Service                │
│                                                         │
│  ┌─────────────┐    ┌───────────────┐                  │
│  │ Store Front │───▶│ Product Svc   │                  │
│  │  (port 80)  │    │  (port 3002)  │                  │
│  └──────┬──────┘    └───────────────┘                  │
│         │                                               │
│         │           ┌───────────────┐   ┌───────────┐  │
│         └──────────▶│  Order Svc    │──▶│ RabbitMQ  │  │
│                     │  (port 3000)  │   │(port 5672)│  │
│                     └───────────────┘   └───────────┘  │
│                                                         │
└─────────────────────────────────────────────────────────┘
         │
         │ LoadBalancer (External IP)
         ▼
    👤 End User
```

### Terraform Resource Map

```
Azure Subscription
└── Resource Group (random name)
    ├── AKS Cluster
    │   ├── System-Assigned Managed Identity
    │   ├── Default Node Pool (Standard_D2_v2 × 3)
    │   └── Network Plugin: kubenet
    └── SSH Public Key (auto-generated)
```

---

## ✅ Prerequisites

### Azure Account
- An active Azure subscription ([create a free account](https://azure.microsoft.com/free/))
- Sufficient permissions to create resources in your subscription

### Tools (automatically available in GitHub Codespaces)
- [Terraform](https://www.terraform.io/) >= 1.0
- [Azure CLI](https://docs.microsoft.com/cli/azure/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)

### Background Knowledge
- Basic understanding of Kubernetes concepts (Pods, Services, Deployments)
- Basic understanding of Terraform (providers, resources, outputs)
- Familiarity with Azure CLI commands

---

## 📁 Project Structure

```
aks-terraform-codespace/
├── .devcontainer/
│   └── devcontainer.json          # GitHub Codespace configuration
├── terraform/
│   ├── providers.tf               # Terraform providers (azurerm, azapi, random)
│   ├── ssh.tf                     # SSH key pair generation
│   ├── main.tf                    # Core resources (Resource Group, AKS cluster)
│   ├── variables.tf               # Input variables
│   └── outputs.tf                 # Output values
├── k8s/
│   └── aks-store-quickstart.yaml  # Kubernetes application manifest
└── README.md                      # This file
```

---

## 🛠️ GitHub Codespace Setup

### Step 1: Fork and Open the Repository

1. Fork this repository to your GitHub account.
2. Click **Code → Codespaces → Create codespace on main**.
3. Wait for the Codespace environment to build (~2–3 minutes).

### Step 2: devcontainer Configuration

The `.devcontainer/devcontainer.json` file automatically provisions the development environment:

```json
{
  "name": "AKS Terraform Dev",
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "features": {
    "ghcr.io/devcontainers/features/terraform:1": {
      "version": "latest"
    },
    "ghcr.io/devcontainers/features/azure-cli:1": {
      "version": "latest"
    },
    "ghcr.io/devcontainers/features/kubectl-helm-minikube:1": {
      "version": "latest",
      "kubectl": "latest"
    }
  },
  "postCreateCommand": "terraform version && az version && kubectl version --client"
}
```

### Step 3: Sign In to Azure

```bash
# Authenticate with your Azure account
az login

# Verify the active subscription
az account show

# Switch subscription if needed
az account set --subscription "<YOUR_SUBSCRIPTION_ID>"
```

> 💡 **Note:** Terraform authenticates with Azure **only through Azure CLI**. Azure PowerShell authentication is not supported.

---

## 🚀 Infrastructure Deployment

### Step 1: Navigate to the Terraform Directory

```bash
cd terraform
```

### Step 2: Initialize Terraform

```bash
terraform init -upgrade
```

This command downloads the required providers:

| Provider | Version |
|----------|---------|
| `hashicorp/azurerm` | ~> 3.0 |
| `azure/azapi` | ~> 1.5 |
| `hashicorp/random` | ~> 3.0 |
| `hashicorp/time` | 0.9.1 |

### Step 3: Review the Execution Plan

```bash
terraform plan -out main.tfplan
```

You will see a list of resources to be created:
- Azure Resource Group (randomly named)
- SSH key pair
- AKS cluster

### Step 4: Apply the Plan

```bash
terraform apply main.tfplan
```

> ⏱️ **Deployment time:** approximately 5–10 minutes

After completion, Terraform outputs key values:

```
Outputs:
kubernetes_cluster_name = "cluster-..."
resource_group_name     = "rg-..."
```

---

## 📦 Application Deployment

### Step 1: Retrieve the kubeconfig

```bash
# Save the cluster configuration
echo "$(terraform output -raw kube_config)" > ./azurek8s

# Verify the file does not contain EOT markers
cat ./azurek8s

# Set the environment variable for kubectl
export KUBECONFIG=./azurek8s
```

> ⚠️ If the file starts with `<< EOT` and ends with `EOT`, remove those lines manually before proceeding.

### Step 2: Verify Cluster Connectivity

```bash
kubectl get nodes
```

Expected output:

```
NAME                               STATUS   ROLES   AGE   VERSION
aks-agentpool-XXXXXXXX-vmss000000  Ready    agent   5m    v1.XX.X
aks-agentpool-XXXXXXXX-vmss000001  Ready    agent   5m    v1.XX.X
aks-agentpool-XXXXXXXX-vmss000002  Ready    agent   5m    v1.XX.X
```

### Step 3: Deploy the Application

```bash
cd ../k8s
kubectl apply -f aks-store-quickstart.yaml
```

Expected output:

```
deployment.apps/rabbitmq created
service/rabbitmq created
deployment.apps/order-service created
service/order-service created
deployment.apps/product-service created
service/product-service created
deployment.apps/store-front created
service/store-front created
```

---

## 🔍 Verification and Access

### Check Pod Status

```bash
kubectl get pods
```

All Pods should reach `Running` status:

```
NAME                               READY   STATUS    RESTARTS   AGE
rabbitmq-XXXXXXXXXX-XXXXX          1/1     Running   0          2m
order-service-XXXXXXXXXX-XXXXX     1/1     Running   0          2m
product-service-XXXXXXXXXX-XXXXX   1/1     Running   0          2m
store-front-XXXXXXXXXX-XXXXX       1/1     Running   0          2m
```

### Get the External IP Address

```bash
kubectl get service store-front --watch
```

Wait for `EXTERNAL-IP` to change from `<pending>` to a real IP address:

```
NAME          TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)        AGE
store-front   LoadBalancer   10.0.100.10   20.62.159.19    80:30025/TCP   5m
```

Stop watching: `Ctrl+C`

### Open the Application

Open your browser and navigate to:
```
http://<EXTERNAL-IP>
```

### Useful Monitoring Commands

```bash
# List all cluster resources
kubectl get all

# View logs for a specific service
kubectl logs -l app=order-service

# Inspect a specific Pod
kubectl describe pod <pod-name>

# List all services and their endpoints
kubectl get services
```

---

## 🧹 Cleanup

> ⚠️ Always clean up resources after finishing to avoid unnecessary Azure charges.

### Step 1: Remove the Kubernetes Application

```bash
kubectl delete -f k8s/aks-store-quickstart.yaml
```

### Step 2: Destroy the Terraform Infrastructure

```bash
cd terraform

# Create a destruction plan
terraform plan -destroy -out main.destroy.tfplan

# Apply the destruction plan
terraform apply main.destroy.tfplan
```

### Step 3: Delete the Service Principal (if created)

```bash
sp=$(terraform output -raw sp)
az ad sp delete --id $sp
```

### Verify Cleanup

```bash
# Confirm the Resource Group has been removed
az group list --query "[].name" -o table
```

---

## 📄 Terraform Configuration Files

### `providers.tf` — Providers and Versions

```hcl
terraform {
  required_version = ">=1.0"

  required_providers {
    azapi = {
      source  = "azure/azapi"
      version = "~>1.5"
    }
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~>3.0"
    }
    random = {
      source  = "hashicorp/random"
      version = "~>3.0"
    }
    time = {
      source  = "hashicorp/time"
      version = "0.9.1"
    }
  }
}

provider "azurerm" {
  features {}
}
```

### `variables.tf` — Input Variables

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `resource_group_location` | string | `eastus` | Azure region for all resources |
| `resource_group_name_prefix` | string | `rg` | Prefix for the resource group name |
| `node_count` | number | `3` | Initial number of nodes in the node pool |
| `username` | string | `azureadmin` | Admin username for the cluster nodes |
| `msi_id` | string | `null` | Managed Service Identity ID (optional) |

### `outputs.tf` — Output Values

| Output | Description | Sensitive |
|--------|-------------|-----------|
| `resource_group_name` | Name of the created Resource Group | No |
| `kubernetes_cluster_name` | Name of the AKS cluster | No |
| `kube_config` | Raw kubeconfig for kubectl | **Yes** |
| `host` | Kubernetes API server address | **Yes** |
| `client_certificate` | Client certificate for authentication | **Yes** |
| `client_key` | Client key for authentication | **Yes** |
| `cluster_ca_certificate` | Cluster CA certificate | **Yes** |
| `cluster_username` | Cluster admin username | **Yes** |
| `cluster_password` | Cluster admin password | **Yes** |

---

## ⚠️ Important Disclaimers

### 1. For Demonstration Only
This project is **not production-ready**. Before deploying to production, review the [AKS baseline reference architecture](https://learn.microsoft.com/azure/architecture/reference-architectures/containers/aks/baseline-aks) to understand how to align it with your business requirements.

### 2. RabbitMQ Without Persistent Storage
RabbitMQ is deployed without a `PersistentVolume` — all messages are lost on Pod restart. For production workloads, use managed services instead:
- **Azure Service Bus** — fully managed message broker
- **Azure CosmosDB** — for stateful data persistence

### 3. Azure Linux 2.0 End of Support
> 🔴 **Critical:** Azure Linux 2.0 is being retired.
> - From **November 30, 2025** — no more security updates
> - From **March 31, 2026** — node images will be deleted; node pool scaling will be blocked

**Action required:** Migrate your node pools to `osSku: AzureLinux3` or upgrade to a supported Kubernetes version.

### 4. Hardcoded Secrets in Manifest
The demo manifest uses plaintext credentials for RabbitMQ (`username`/`password`). In production environments, always use:
- **Azure Key Vault** for secret storage
- **Kubernetes Secrets** encrypted with Azure Key Management Service (KMS)
- **Workload Identity** instead of static credentials

---

## 🔧 Troubleshooting

### `terraform apply` fails: "insufficient permissions"

```bash
# Check the currently signed-in account
az account show

# List role assignments for your account
az role assignment list \
  --assignee $(az account show --query user.name -o tsv) \
  --output table
```

### kubeconfig error: "yaml: line 2: mapping values are not allowed"

```bash
# Remove EOT markers from the kubeconfig file
grep -v "EOT" ./azurek8s > ./azurek8s.clean
mv ./azurek8s.clean ./azurek8s
export KUBECONFIG=./azurek8s
```

### Pod stuck in `Pending` state

```bash
# Inspect Pod events
kubectl describe pod <pod-name>

# Check node resource availability
kubectl describe nodes

# Check if resource requests exceed node capacity
kubectl get pods -o wide
```

### `EXTERNAL-IP` remains `<pending>` for a long time

```bash
# Check LoadBalancer events
kubectl describe service store-front

# Azure typically assigns an IP within 2–5 minutes
# If it persists, check your subscription's public IP quota
az network public-ip list --output table
```

### Terraform state lock

```bash
# If the state is locked due to a previously failed run
terraform force-unlock <LOCK_ID>
```

### `order-service` Pod keeps restarting

The `order-service` uses an `initContainer` that waits for RabbitMQ to become available. If RabbitMQ is not ready, the order-service Pod will wait.

```bash
# Check RabbitMQ Pod status first
kubectl get pods -l app=rabbitmq

# Check initContainer logs
kubectl logs <order-service-pod-name> -c wait-for-rabbitmq
```

---

## 💰 Estimated Costs

> Costs vary by region and how long the cluster runs.

| Resource | Type | Estimated cost/hour |
|----------|------|---------------------|
| AKS Control Plane | Managed | $0.00 (free tier) |
| 3× Standard_D2_v2 | VM (2 vCPU, 7 GB RAM) | ~$0.30 |
| Standard Load Balancer | Networking | ~$0.02 |
| Public IP Address | Static | ~$0.004 |
| **Total** | | **~$0.32/hour** |

💡 Use the [Azure Pricing Calculator](https://azure.microsoft.com/pricing/calculator/) for a precise estimate in your region.

> 🔔 **Tip:** Set up a budget alert in [Microsoft Cost Management](https://portal.azure.com/#blade/Microsoft_Azure_CostManagement/Menu/overview) to avoid unexpected charges.

---

## 📚 Additional Resources

| Resource | Link |
|----------|------|
| AKS Documentation | [learn.microsoft.com/azure/aks](https://learn.microsoft.com/azure/aks) |
| Terraform AzureRM Provider | [registry.terraform.io](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs) |
| AKS Key Concepts | [AKS Concepts](https://learn.microsoft.com/azure/aks/concepts-clusters-workloads) |
| AKS Baseline Architecture | [AKS Baseline](https://learn.microsoft.com/azure/architecture/reference-architectures/containers/aks/baseline-aks) |
| Azure Store Demo (GitHub) | [azure-samples/aks-store-demo](https://github.com/Azure-Samples/aks-store-demo) |
| Azure Subscription Limits | [Service Limits](https://learn.microsoft.com/azure/azure-resource-manager/management/azure-subscription-service-limits) |
| GitHub Codespaces Docs | [GitHub Codespaces](https://docs.github.com/codespaces) |
| devcontainer Features | [containers.dev/features](https://containers.dev/features) |

---

## 🤝 Contributing

1. Fork the repository.
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Commit your changes: `git commit -m 'Add your feature'`
4. Push the branch: `git push origin feature/your-feature`
5. Open a Pull Request.

Please ensure your changes pass `terraform validate` and `terraform fmt` before submitting.

---


<div align="center">

[![Azure](https://img.shields.io/badge/Azure-AKS-0089D6?logo=microsoft-azure&logoColor=white)](https://azure.microsoft.com)
[![Terraform](https://img.shields.io/badge/IaC-Terraform-7B42BC?logo=terraform&logoColor=white)](https://terraform.io)
[![Kubernetes](https://img.shields.io/badge/Orchestration-Kubernetes-326CE5?logo=kubernetes&logoColor=white)](https://kubernetes.io)
[![GitHub Codespaces](https://img.shields.io/badge/Dev-GitHub_Codespaces-181717?logo=github&logoColor=white)](https://github.com/features/codespaces)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

</div>
