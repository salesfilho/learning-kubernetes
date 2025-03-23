## Kubernetes Load Balancer

Kubernetes generally exposes cluster applications in three ways:
- Node Port
- Load Balancer
- Ingress

In this lab cluster, we chose to use a baremetal configuration using [MetalLB Loadbalancer ](https://metallb.universe.tf/) and the Ingress service.

The topology of the lab cluster is shown below.




### Setting up a MetalLB/Ingress service
