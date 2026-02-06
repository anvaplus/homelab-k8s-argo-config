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

## Repository Structure

The current structure of this repository is as follows, and it will evolve as more tools are installed in the homelab:
```
├── base/                     # Base configurations for all tools
│   ├── argocd/               # ArgoCD itself configuration
│   ├── cert-manager/         # cert-manager configuration
│   ├── cilium/               # Cilium CNI configuration
│   ├── external-secrets/     # External Secrets Operator configuration
│   ├── ingress/              # Ingress controllers configuration
│   │   ├── metallb/          # MetalLB load balancer
│   │   └── traefik/          # Traefik ingress controller
│   ├── namespaces/           # Namespace definitions
│   ├── projects/             # ArgoCD project definitions
│   └── secrets/              # Secret configurations
└── environments/             # Environment-specific overlays
    ├── dev/                  # Development environment configs
    │   ├── _root/            # Root application for the dev environment
    │   ├── argocd/           # ArgoCD overrides for dev
    │   ├── cert-manager/     # cert-manager overrides for dev
    │   ├── cilium/           # Cilium overrides for dev
    │   ├── external-secrets/ # External Secrets overrides for dev
    │   ├── ingress/          # Ingress overrides for dev
    │   │   ├── metallb/      # MetalLB configuration for dev
    │   │   └── traefik/      # Traefik configuration for dev
    │   ├── namespaces/       # Namespaces overrides for dev
    │   ├── projects/         # ArgoCD project overrides for dev
    │   └── secrets/          # Secrets overrides for dev
    └── prod/                 # Production environment configs
```

## Recent Updates

### February 2026

- **Gateway API Migration**: Migrated the ingress controller to utilize the Gateway API, enabling the `kubernetesGateway` provider and disabling legacy providers for more robust and standardized ingress management.
- **HTTPRoute Management**: Deployed a dedicated ArgoCD application to manage `HTTPRoute` resources, improving the organization and management of ingress configurations.
- **Global Domain Configuration**: Introduced a global configuration file to share a default domain across ArgoCD components, streamlining the setup for ingresses, certificates, and other services.
- **Wildcard Certificate Automation**: Added `cert-manager` resources to automate the issuance of wildcard certificates using Let's Encrypt and a DNS01 solver, simplifying TLS management.
- **Public DNS Resolvers for Cert-Manager**: Configured `cert-manager` to use public recursive nameservers for DNS-01 challenges, ensuring reliable domain validation in local DNS environments.
- **External Secrets for DNS Service Account**: Integrated `External Secrets` to securely fetch `cert-manager` DNS service account credentials from 1Password.

### January 2026

- **Cert-Manager Integration**: Added `cert-manager` base manifests and configurations to manage TLS certificates within the cluster.
- **Traefik Ingress Controller**: Deployed `Traefik` as the ingress controller, managed by ArgoCD, and enabled the Gateway API for modern ingress routing.
- **Gateway API in Cilium**: Enabled Gateway API support in the Cilium configuration to align with modern Kubernetes networking standards.
- **MetalLB for Load Balancing**: Integrated `MetalLB` to provide load balancer services, including IP address pool configuration.
- **1Password Integration**: Set up the `External Secrets Operator` to fetch secrets from `1Password`, enhancing security and secret management.
- **Namespace Management**: Added centralized namespace management through ArgoCD for better resource organization.
- **Cilium Migration to ArgoCD**: Migrated the Cilium installation to be managed by ArgoCD for declarative configuration and automated deployments.
- **ArgoCD Project and Application Setup**: Configured the initial ArgoCD project and a root application to manage resources in a GitOps-centric workflow.
