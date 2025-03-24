### 1. Create the  namespace argocd

```
microk8s.kubectl create namespace argocd
```

### 2. Apply install argocd on the Cluster

```
microk8s.kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
### 3. Service Type Load Balancer

Change the argocd-server service type to ``LoadBalancer:``

```
microk8s kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

### 4. Ingress rule to Load Balancer

Create a ingress rule as seen below:

```
# argocd-ingress-rule.yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-ingress
  namespace: argocd
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  ingressClassName: public
  rules:
    - http:
        paths:
        - path: /argocd
          pathType: Exact
          backend:
            service:
              name: argocd-server
              port:
                number: 80
```
