### Como aplicar os recursos

1. Crie os arquivos YAML acima
2. Aplique-os na ordem:

```
microk8s.kubectl apply -f secrets.yaml
microk8s.kubectl apply -f postgres-statefulset.yaml
microk8s.kubectl apply -f postgres-service.yaml
microk8s.kubectl apply -f backend-deployment.yaml
microk8s.kubectl apply -f backend-service.yaml
microk8s.kubectl apply -f frontend-deployment.yaml
microk8s.kubectl apply -f frontend-service.yaml
```

