# homelab-k8s-argo-config

## Overview

This repository contains all ArgoCD configurations for the Homelab platform. It manages the installation and configuration of platform tools and infrastructure components across different environments using a GitOps approach.

## 🔗 Related Repositories

This repository is part of a multi-repository GitOps architecture:

- **[homelab-k8s-argo-config](https://github.com/anvaplus/homelab-k8s-argo-config)** (This repository) - ArgoCD configurations for platform tools and the app-of-apps pattern.
- **[homelab-k8s-base-manifests](https://github.com/anvaplus/homelab-k8s-base-manifests)** - Base Helm chart templates for applications.
- **[homelab-k8s-environments](https://github.com/anvaplus/homelab-k8s-environments)** - Environment-specific version declarations for applications.
- **[homelab-k8s-environments-apps](https://github.com/anvaplus/homelab-k8s-environments-apps)** - Application configuration values and ArgoCD Application definitions.

## How It Works

The workflow is based on a GitOps approach with ArgoCD:

### Initial Setup Steps

1.  **Create the cluster**: The cluster is installed without a CNI and kube-proxy. As an example, you can see how the cluster is created in the [sidero-omni-talos-proxmox](https://github.com/anvaplus/sidero-omni-talos-proxmox) repository.
    ```bash
    omnictl cluster template sync -v -f cluster-template/k8s-dev-dhcp.yaml
    ```

2.  **Install Cilium**: Install Cilium using Helm.
    ```bash
    helm repo add cilium https://helm.cilium.io/
    helm repo update
    helm install \
        cilium \
        cilium/cilium \
        --version 1.18.6 \
        --namespace kube-system \
        --set ipam.mode=kubernetes \
        --set kubeProxyReplacement=true \
        --set securityContext.capabilities.ciliumAgent="{CHOWN,KILL,NET_ADMIN,NET_RAW,IPC_LOCK,SYS_ADMIN,SYS_RESOURCE,DAC_OVERRIDE,FOWNER,SETGID,SETUID}" \
        --set securityContext.capabilities.cleanCiliumState="{NET_ADMIN,SYS_ADMIN,SYS_RESOURCE}" \
        --set cgroup.autoMount.enabled=false \
        --set cgroup.hostRoot=/sys/fs/cgroup \
        --set k8sServiceHost=localhost \
        --set k8sServicePort=7445
    ```

3.  **Install Argo CD**: Install Argo CD with Helm.
    ```bash
    helm repo add argo https://argoproj.github.io/argo-helm
    helm repo update
    kubectl create namespace argocd
    helm install argo-cd argo/argo-cd --version 9.3.4 -n argocd
    ```

4.  **Install the main project**: This project allows ArgoCD to manage the repositories.
    ```bash
    kubectl create -f /homelab-k8s-argo-config/_initial_setup/project-argo-config.yaml
    ```

### GitOps Workflow

1.  **Base Configuration**: The `base/` directory contains the base Helm charts and configurations for platform tools. These are meant to be generic and reusable across all environments.

2.  **Environment Overlays**: The `environments/` directory contains environment-specific configurations. Each environment (e.g., `dev`, `prod`) has its own directory where it can override the base configurations using Kustomize. This allows for tailoring applications to the specific needs of each environment.

3.  **App-of-Apps Pattern**: The `_root` directory within each environment is used to implement the app-of-apps pattern in ArgoCD. By running `kubectl apply -k .` in this directory, you install the root applications that, in turn, manage all other applications for that environment.

4.  **Adding New Tools**: To add a new tool, you first add its base configuration to the `base/` directory. Then, you customize it for each environment in the `environments/` directory.

Currently, `ArgoCD` and `Cilium` are configured using this approach.

## Repository Structure

The current structure of this repository is as follows, and it will evolve as more tools are installed in the homelab:
```
├── base/                      # Base configurations for all tools
│   ├── argocd/               # ArgoCD itself configuration
│   └── cilium/               # Cilium CNI configuration
│   └── projects/             # ArgoCD project definitions
└── environments/              # Environment-specific overlays
    ├── dev/                  # Development environment configs
    │   ├── _root/            # Root application for the dev environment
    │   ├── argocd/           # ArgoCD overrides for dev
    │   └── cilium/           # Cilium overrides for dev
    │   └── projects/         # ArgoCD project overrides for dev
    └── prod/                 # Production environment configs
```

## Secrets Management

Secrets should be managed using a tool like the External Secrets Operator.
- Store secrets in a secure vault (e.g., HashiCorp Vault, GCP Secret Manager, etc.).
- External Secrets will sync them to Kubernetes secrets.
- Never commit actual secret values to this repository.

