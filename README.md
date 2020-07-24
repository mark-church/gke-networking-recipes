# GKE Networking Recipes

This repository contains various use cases and examples of GKE Networking. For each of the use-cases there are full YAML examples that show how and when these GKE capabilities should be used.

If you're not familiar with the basics of Kubernetes networking then check out [cluster networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/) and [service networking](https://kubernetes.io/docs/concepts/services-networking/). These resources should give you some of the foundations behind Kubernetes networking.

GKE is a managed Kubernetes platform that provides a more opinionated and seamless experience. For more information on GKE networking, check out some of the following resources:
- [GKE Network Overview](https://cloud.google.com/kubernetes-engine/docs/concepts/network-overview)
- [GKE Ingress Overview](https://cloud.google.com/kubernetes-engine/docs/concepts/ingress)
- [Ingress features](https://cloud.google.com/kubernetes-engine/docs/how-to/ingress-features)

### Ingress

- [Basic External Ingress](external-ingress-basic) - Deploy host-based routing through an internet-facing HTTP load balancer

