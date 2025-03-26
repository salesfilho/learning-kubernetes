## 1. Kubernetes Load Balancer

Kubernetes generally exposes cluster applications in three ways:
- Node Port
- Load Balancer
- Ingress

In this lab cluster, we chose to use a baremetal configuration using [MetalLB Loadbalancer ](https://metallb.universe.tf/) and the Ingress service.

The topology of the lab cluster is shown below:

```mermaid
graph TD;
    Externsl_User-->LoadBalancer_IP:192.168.0.245;
    LoadBalancer_IP:192.168.0.245-->master01;
    LoadBalancer_IP:192.168.0.245-->master02;
    LoadBalancer_IP:192.168.0.245-->dcn01;

    master01-->Ingress_Controller;
    master02-->Ingress_Controller;
    dcn01-->Ingress_Controller;

    Ingress_Controller-->Service_A;
    Ingress_Controller-->Service_B;

    Service_A-->Pod_A;
    Service_B-->Pod_B;

```

## 2. Setting up the MetalLB Service

The first step is enable addon on microk8s cluster. When you enable this add on you will be asked for an IP address pool that MetalLB will hand out IPs.

## 2.1 Enabling MetalLB Service

```
microk8s enable metallb
```

Alternatively, you can provide the IP address pool in the enable command. In this lab case, we'll take 192.168.0.40-192.168.0.50:

```
microk8s enable metallb:192.168.0.240-192.168.0.250
```

## 2.2 Configure IPAddressPool resources

It is possible to configure IP address pools that MetalLB will use to allocate IP addresses using custom resources.

For our example lab, create the following custom address pool:

```
# addresspool.yaml
---
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: default-addresspool
  namespace: metallb-system
spec: 
  addresses:
  - 192.168.0.240-192.168.0.250
```

And apply it with:

```
microk8s kubectl apply -f addresspool.yaml
```

## 2.3 Setting up a MetalLB/Ingress service

For load balancing in a MicroK8s cluster, MetalLB can make use of Ingress. so, let's enable ingress:
```
microk8s enable ingress
```

Now, create a suitable ingress service, using the config file bellow:

```
# ingress-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: ingress
  namespace: ingress
  annotations:
    metallb.universe.tf/address-pool: default-addresspool
    cert-manager.io/cluster-issuer: local-ca-cluster-issuer
spec:
  selector:
    name: nginx-ingress-microk8s
  type: LoadBalancer
  # loadBalancerIP is optional. MetalLB will automatically allocate an IP 
  # from its pool if not specified. You can also specify one manually.
  # loadBalancerIP: x.y.z.a
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 80
    - name: https
      protocol: TCP
      port: 443
      targetPort: 443
```

To apply configuratio, run:

```
microk8s kubectl apply -f ingress-service.yaml
```


Now there is a load-balancer which listens on an arbitrary IP and directs traffic towards one of the listening ingress controllers.
Let's check it:

```
microk8s kubectl get svc -n ingress -o wide
```

...the result is something like this:

```
NAME      TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)                      AGE   SELECTOR
ingress   LoadBalancer   10.152.183.209   192.168.0.240   80:32068/TCP,443:30386/TCP   47h   name=nginx-ingress-microk8s
```
