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
  annotations:
    cert-manager.io/cluster-issuer: local-ca-cluster-issuer
    nginx.ingress.kubernetes.io/ssl-passthrough: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
  name: argocd-ingress
  namespace: argocd
spec:
  ingressClassName: nginx
  rules:
  - host: argocd.local
    http:
      paths:
      - pathType: Prefix
        path: /
        backend:
          service:
            name: argocd-server
            port:
              number: 443
  tls:
  - hosts:
    - argocd.local
    secretName: argocd-server-tls

```

Don't forget to configure your local DNS or hosts file with the hostname ``argocd.local``.

### 4. Testing configuration

##### Check the status of argocd service like this:
```
microk8s.kubectl get svc -n argocd
```
... the result is something like this:

```
NAME                                      TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)                      AGE
argocd-applicationset-controller          ClusterIP      10.152.183.50    <none>          7000/TCP,8080/TCP            16h
argocd-dex-server                         ClusterIP      10.152.183.157   <none>          5556/TCP,5557/TCP,5558/TCP   16h
argocd-metrics                            ClusterIP      10.152.183.214   <none>          8082/TCP                     16h
argocd-notifications-controller-metrics   ClusterIP      10.152.183.53    <none>          9001/TCP                     16h
argocd-redis                              ClusterIP      10.152.183.73    <none>          6379/TCP                     16h
argocd-repo-server                        ClusterIP      10.152.183.105   <none>          8081/TCP,8084/TCP            16h
argocd-server                             LoadBalancer   10.152.183.198   192.168.0.241   80:32455/TCP,443:32712/TCP   16h
argocd-server-metrics                     ClusterIP      10.152.183.241   <none>          8083/TCP                     16h
```

Pay attention to LoadBalancer EXTERNAL-IP. To access the ArgoCD UI you need set argocd.local in your DNS client or /etc/hosts 

##### Check Ingress rules:

Run this command:
```
microk8s.kubectl describe ingress -n argocd

```
... the result is something like this:

```
Name:             argocd-ingress
Labels:           <none>
Namespace:        argocd
Address:          127.0.0.1
Ingress Class:    nginx
Default backend:  <default>
TLS:
  argocd-server-tls terminates argocd.local
Rules:
  Host          Path  Backends
  ----          ----  --------
  argocd.local  
                /   argocd-server:443 (10.1.66.91:8080)
Annotations:    cert-manager.io/cluster-issuer: local-ca-cluster-issuer
                nginx.ingress.kubernetes.io/backend-protocol: HTTPS
                nginx.ingress.kubernetes.io/ssl-passthrough: true
Events:         <none>
```

##### First Login as Admin:

To log in to the ArgoCD UI for the first time, you need to use the argocd admin password that was automatically generated during installation. To do this, you can run this command:

```
microk8s.kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```
If everything is ok, go to ``http://argocd.local`` in your browser to log in using ``admin`` as the username and the password you got in the previous step.

##### Reset Admin password:

If something went wrong with the login, you need to force reset the administrator password. To do this, you need three steps:

1. Reset current password in argocd secret
2. Restart all argocd-server pods
3. Recover new password

Let's to do this. Run this command to reset current password in argocd secret (Invalidating Admin Credentials):

```
kubectl patch secret argocd-secret -n argocd -p '{"data": {"admin.password": null, "admin.passwordMtime": null}}'
```

Restart all argocd-server pods:

```
kubectl delete pods -n argocd -l app.kubernetes.io/name=argocd-server

```
... or force delete

```
kubectl delete pods -n argocd -l app.kubernetes.io/name=argocd-server --grace-period=0 --force

```

Recover new password (Decrypting the New Password):

```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

```

This command fetches the base64-encoded password, decodes it, and displays the new admin password.
