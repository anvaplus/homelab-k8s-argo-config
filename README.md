# homelab-k8s-argo-config

## Overview

This repository contains all ArgoCD configurations for the Homelab platform. It manages the installation and configuration of platform tools and infrastructure components across different environments using a GitOps approach.

*Note: This document represents a plan. The contents of this repository will evolve as the homelab is built and refined.*

## 🔗 Related Repositories

This repository is part of a 4-repository GitOps architecture:

- **[homelab-k8s-argo-config](https://github.com/anvaplus/homelab-k8s-argo-config)** (This repository) - ArgoCD configurations for platform tools
- **[homelab-k8s-base-manifests](https://github.com/anvaplus/homelab-k8s-base-manifests)** - Helm chart templates for applications
- **[homelab-k8s-environments](https://github.com/anvaplus/homelab-k8s-environments)** - Version declarations for microservices
- **[homelab-k8s-environments-apps](https://github.com/anvaplus/homelab-k8s-environments-apps)** - Application configuration values

## Purpose

This repository serves as the central configuration hub for:
- Platform infrastructure tools (e.g., Istio, Kafka, Redis, etc.)
- ArgoCD application definitions
- Environment-specific configurations (development, production)
- RBAC and security policies
- Secrets management configurations

## Repository Structure

The planned structure for this repository is as follows:
```
├── base/                      # Base configurations for all tools
│   ├── argocd/               # ArgoCD itself configuration
│   ├── cert-manager/         # Certificate management
│   ├── external-dns/         # DNS management
│   ├── external-secrets/     # Secret synchronization
│   ├── ingress/              # Ingress controller
│   ├── istio/                # Service mesh
│   ├── namespaces/           # Namespace definitions
│   ├── projects/             # ArgoCD project definitions
│   └── secrets/              # Secret management
└── environments/              # Environment-specific overlays
    ├── dev/                  # Development environment configs
    └── prod/                 # Production environment configs
```

## Key Components

### Base Tools Configuration

The `base/` directory contains Kustomize configurations for essential platform tools. This can be extended as the homelab grows.

### Environment Configuration

Each environment (`dev`, `prod`) has its own directory in `environments/` that:
- Overrides base configurations
- Defines environment-specific values
- Manages environment secrets
- Configures ingress and DNS for that environment

### Projects

The `base/projects/` directory defines ArgoCD AppProjects, which group applications and define:
- Source repositories
- Allowed destinations
- RBAC policies

## How It Works

### GitOps Workflow

1. **Configuration Changes**: Create PRs with configuration changes.
2. **Review Process**: Review and approve changes.
3. **Auto-Sync**: ArgoCD automatically detects changes in the `main` branch and applies them to the cluster.
4. **Self-Healing**: ArgoCD continuously reconciles the desired state with the actual cluster state.

## Environment Details

### Development (DEV) Environment
- **Purpose**: For testing new configurations and applications.
- **Auto-sync**: Enabled.

### Production (PROD) Environment
- **Purpose**: For stable, long-running applications.
- **Auto-sync**: Enabled, but with more cautious review processes.

## Secrets Management

Secrets should be managed using a tool like the External Secrets Operator.
- Store secrets in a secure vault (e.g., HashiCorp Vault, GCP Secret Manager, etc.).
- External Secrets will sync them to Kubernetes secrets.
- Never commit actual secret values to this repository.
