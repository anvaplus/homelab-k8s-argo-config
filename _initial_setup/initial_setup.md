1. Create the cluster. The cluster is inatlled withoud CNI and kube-proxy

```bash
omnictl cluster template sync -v -f cluster-template/k8s-dev-dhcp.yaml
```

2. Install Cilium using Helm

```bash
helm repo add cilium https://helm.cilium.io/
helm repo update
helm install \
    cilium \
    cilium/cilium \
    --version 1.18.0 \
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

3. Install Argo CD with Helm

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
kubectl create namespace argocd
helm install argo-cd argo/argo-cd --version 9.3.4 -n argocd
```

4. Install the main project

```
kubectl create -f /homelab-k8s-argo-config/_initial_setup/project-argo-config.yaml
```