# homelab-k8s-argo-config

## Overview

This repository is the GitOps control plane for platform services in the homelab cluster.

It contains Argo CD projects, platform Applications, shared namespaces/secrets, and environment overlays for infrastructure components such as networking, certificates, storage, and database operators.

## GitOps Architecture (4 Repositories)

The homelab deployment model is split by responsibility:

- [homelab-k8s-argo-config](https://github.com/anvaplus/homelab-k8s-argo-config): Argo CD projects and platform stack definitions  (this repository).
- [homelab-k8s-base-manifests](https://github.com/anvaplus/homelab-k8s-base-manifests): Helm charts and library templates used by workloads.
- [homelab-k8s-environments](https://github.com/anvaplus/homelab-k8s-environments): Version registry per environment (for example image tag/version fields).
- [homelab-k8s-environments-apps](https://github.com/anvaplus/homelab-k8s-environments-apps): Application catalog and runtime values per environment.

## What This Repository Owns

- Argo CD AppProject and source repository allow-list.
- Platform components under base (argocd, cilium, ingress, cert-manager, longhorn, cnpg, external-secrets, keycloak).
- Environment overlays under environments (dev, prod).
- Kustomize roots for ordered platform installation.

This repository does not own workload chart templates or workload runtime values.

## Bootstrap Flow

1. Create the cluster (for example from the Talos/Omni repository).
2. Install Argo CD.
3. Apply initial project definition:

```bash
kubectl apply -f _initial_setup/project-argo-config.yaml
```

4. Apply environment root kustomization (for example dev):

```bash
kubectl apply -k environments/dev/_root
```

After bootstrap, Argo CD continuously reconciles platform resources from this repository.

## How It Connects To App Deployments

Platform setup here enables workload deployments that are defined in the other 3 repositories:

1. Charts come from homelab-k8s-base-manifests.
2. Version values come from homelab-k8s-environments.
3. Runtime values and Argo CD app definitions come from homelab-k8s-environments-apps.

The AppProject in base/projects includes all required source repositories so multi-source Applications can resolve successfully.

## Repository Layout

Run from repository root:

```bash
tree -d -L 3
```

```text
.
├── _initial_setup
├── base
│   ├── argocd
│   ├── cert-manager
│   ├── cilium
│   ├── database
│   │   ├── cnpg-cluster
│   │   └── cnpg-operator
│   ├── external-secrets
│   │   ├── 1password-connect
│   │   └── external-secrets
│   ├── ingress
│   │   ├── metallb
│   │   └── traefik
│   ├── keycloak
│   ├── longhorn
│   ├── namespaces
│   ├── projects
│   └── secrets
│       ├── argocd
│       ├── cert-manager
│       └── external-secrets-config
└── environments
    ├── dev
    │   ├── _root
    │   ├── argocd
    │   ├── cert-manager
    │   ├── cilium
    │   ├── database
    │   ├── external-secrets
    │   ├── ingress
    │   ├── keycloak
    │   ├── longhorn
    │   ├── namespaces
    │   ├── projects
    │   └── secrets
    └── prod
```

## Change Guidelines

- Add new platform capability in base first, then wire overlay/customization in each environment.
- Keep environment-specific values in environments overlays, not in base.
- When adding new workload source repositories, update the AppProject sourceRepos allow-list.
