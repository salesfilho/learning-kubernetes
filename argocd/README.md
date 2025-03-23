1. Create the  namespace argocd

```kubectl create namespace argocd```

2. Apply install argocd on the Cluster

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml```
