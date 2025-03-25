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
  name: argocd-server-ingress
  namespace: argocd
  annotations:
    nginx.ingress.kubernetes.io/backend-protocol: "https"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    nginx.ingress.kubernetes.io/ssl-passthrough: "true"
spec:
  ingressClassName: nginx
  rules:
  - host: argocd.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: argocd-server
            port:
              name: https

```

Don't forget to configure your local DNS or hosts file with the hostname ``argocd.local``.

### 4. Testing configuration

##### Check the status of argocd service like this:
```
microk8s.kubectl get svc -n argocd
```
... the result is something like this:

```
argocd-applicationset-controller          ClusterIP      10.152.183.86    <none>           7000/TCP,8080/TCP            138m
argocd-dex-server                         ClusterIP      10.152.183.246   <none>           5556/TCP,5557/TCP,5558/TCP   138m
argocd-metrics                            ClusterIP      10.152.183.210   <none>           8082/TCP                     138m
argocd-notifications-controller-metrics   ClusterIP      10.152.183.109   <none>           9001/TCP                     138m
argocd-redis                              ClusterIP      10.152.183.152   <none>           6379/TCP                     138m
argocd-repo-server                        ClusterIP      10.152.183.79    <none>           8081/TCP,8084/TCP            138m
argocd-server                             LoadBalancer   10.152.183.58    10.100.100.100   80:30867/TCP,443:30679/TCP   138m
argocd-server-metrics                     ClusterIP      10.152.183.180   <none>           8083/TCP                     138m
sa
```

##### Check Ingress rules:

Run this command:
```
microk8s.kubectl describe ingress -n argocd

```
... the result is something like this:

```
Name:             argocd-server-ingress
Labels:           <none>
Namespace:        argocd
Address:          127.0.0.1
Ingress Class:    nginx
Default backend:  <default>
Rules:
  Host          Path  Backends
  ----          ----  --------
  argocd.local  
                /   argocd-server:https (10.1.59.209:8080)
Annotations:    nginx.ingress.kubernetes.io/backend-protocol: https
                nginx.ingress.kubernetes.io/force-ssl-redirect: true
                nginx.ingress.kubernetes.io/ssl-passthrough: true
Events:         <none>
```

##### Access ArgoCD UI:

Point your browser to ``http://argocd.local``

That is all!!!



