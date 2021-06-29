# User guide for Prometheus and F5 examples
User guide to deploying Prometheus and collecting metrics from Telemetry Streaming (TS), Container Ingress Services (CIS), and NGINX Kubernetes Ingress Controller (KIC).

In the case of KIC, we will deploy both open source KIC (i.e., free community version), and KIC using NGINX Plus (i.e., requires paid license or demo key).

This guide is heavily similar to the work done by Mark Dittmer here. This guide mostly exists to append NGINX KIC to Mark's user guide and demonstrate the added value of KIC with NGINX Plus.

## Prerequisites
All instructions for prerequisites are provided or linked to below.
- Running K8s cluster version >1.18.
- Running BIG-IP in standalone or HA cluster.
- NGINX Plus license or demo cert/key pair.
- A private registry hosting an image of your KIC image based on NGINX Plus.

## CIS (Container Ingress Services)
Once your BIG-IP is up and running, edit the file /ingress/cis/cis1.yaml (and, optionally, cis2.yaml if you have 2 BIG-IP's in a HA pair). Edit the IP address around line 49 so that CIS points to the BIG-IP's mgmt IP address.
Also edit the file /ingress/cis/secret_sa_rbac.yaml so that the password in the secret that is created is the base64-encoded value of the password of your BIG-IP.
Install CIS with the following commands:
```bash
kubectl apply -f /ingress/cis/secret_sa_rbac.yaml
kubectl apply -f /ingress/cis/crd-definition/customresourcedefinitions.yaml
kubectl apply -f /ingress/cis/cis1.yaml
kubectl apply -f /ingress/cis/cis2.yaml
```

## KIC (NGINX Kubernetes Ingress Controller)

## KIC with NGINX Plus

## Prometheus
Prometheus is a free software application used for event monitoring and alerting. We will deploy Prometheus and again, these instructions are simply copied from another user guide. Our goal here is to run Prometheus in a pod inside Kubernenetes so that it can pull metrics from other pods, using the K8s api to discover other pods with appropriate annotations and labels.


