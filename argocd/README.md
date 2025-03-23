### 1. Create the  namespace argocd

```
microk8s.kubectl create namespace argocd
```

### 2. Apply install argocd on the Cluster

```
microk8s.kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
