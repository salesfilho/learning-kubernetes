# 1. Creating a CA-Cert

### 1.1 Enable cert-manager
Enable addon:
```
microk8s enable cert-manager 
```

Check status:

```
microk8s status
```

It's running:



```
microk8s is running
high-availability: no
  datastore master nodes: 192.168.0.253:19001
  datastore standby nodes: none
addons:
  enabled:
    cert-manager         # (core) Cloud native certificate management
    dns                  # (core) CoreDNS
    ha-cluster           # (core) Configure high availability on the current node
    helm                 # (core) Helm - the package manager for Kubernetes
    helm3                # (core) Helm 3 - the package manager for Kubernetes
    ingress              # (core) Ingress controller for external access
    metallb              # (core) Loadbalancer for your Kubernetes cluster
  disabled:
    cis-hardening        # (core) Apply CIS K8s hardening
    community            # (core) The community addons repository
    dashboard            # (core) The Kubernetes dashboard
    host-access          # (core) Allow Pods connecting to Host services smoothly
    hostpath-storage     # (core) Storage class; allocates storage from host directory
    kube-ovn             # (core) An advanced network fabric for Kubernetes
    mayastor             # (core) OpenEBS MayaStor
    metrics-server       # (core) K8s Metrics Server for API access to service metrics
    minio                # (core) MinIO object storage
    observability        # (core) A lightweight observability stack for logs, traces and metrics
    prometheus           # (core) Prometheus operator for monitoring and logging
    rbac                 # (core) Role-Based Access Control for authorisation
    registry             # (core) Private image registry exposed on localhost:32000
    rook-ceph            # (core) Distributed Ceph storage using Rook
    storage              # (core) Alias to hostpath-storage add-on, deprecated
```

### 1.2 Create self signed cluster issuer:

```
# cluster-issuer.yaml

apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: selfsigned-cluster-issuer
spec:
  selfSigned: {}

```

Run this command to create the Cluster Issue:

```
microk8s.kubectl apply  -f cluster-issuer.yaml 
```

Create CA certificate in the cert-manager namespace to use it as a ClusterIssuer:

```
# ca-certificate.yaml

apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: local-ca
  namespace: cert-manager
spec:
  isCA: true
  commonName: local-ca
  secretName: local-ca
  issuerRef:
    name: selfsigned-cluster-issuer
    kind: ClusterIssuer
    group: cert-manager.io
```

Run this commant to create the CA Certificate:

```
microk8s.kubectl apply  -f ca-certificate.yaml
```

Create clusterissuer

```
# local-ca-cluster-issuer.yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: local-ca-cluster-issuer
spec:
  ca:
    secretName: local-ca
```

Run this commant to create the clusterissuer:

```
microk8s.kubectl apply  -f local-ca-cluster-issuer.yaml
```

Now, let's test with ingress:

```
# test-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    cert-manager.io/cluster-issuer: local-ca-cluster-issuer
  name: test-ingress
  namespace: test
spec:
  rules:
  - host: site.local
    http:
      paths:
      - pathType: Prefix
        path: /
        backend:
          service:
            name: demo
            port:
              number: 80
  tls:
  - hosts:
    - site.local
    secretName: site-cert
```

Run this to create ingress:
```
microk8s.kubectl apply  -f test-ingress.yaml
```

To check and Extract CA, cert, key:

```
microk8s.kubectl get secret -n ingress-test myingress-cert -o json | jq -r  '.data["ca.crt"]' | base64 -d > ca.crt
microk8s.kubectl get secret -n ingress-test myingress-cert -o json | jq -r  '.data["tls.crt"]' | base64 -d > tls.crt
microk8s.kubectl get secret -n ingress-test myingress-cert -o json | jq -r  '.data["tls.key"]' | base64 -d > tls.key

```

To check and Extract cluster issuer CA

```
microk8s.kubectl get secrets -n cert-manager local-ca -o json | jq -r '.data["tls.crt"]' | base64 -d > issuer.crt

diff issuer.crt ca.crt && echo "Issuing CA matches Ingress CA" || echo "Issuing CA doesn't match Ingress CA"

openssl verify -CAfile issuer.crt tls.crt

```





