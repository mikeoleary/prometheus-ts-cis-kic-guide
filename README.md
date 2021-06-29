# User guide for Prometheus and F5 examples
User guide to deploying Prometheus and collecting metrics from Telemetry Streaming (TS), Container Ingress Services (CIS), and NGINX Kubernetes Ingress Controller (KIC).

In the case of KIC, we will deploy both open source KIC (i.e., free community version), and KIC using NGINX Plus (i.e., requires paid license or demo key).

This guide is heavily similar to the work done by Mark Dittmer here. This guide mostly exists to append NGINX KIC to Mark's user guide and demonstrate the added value of KIC with NGINX Plus.

## Prerequisites
All instructions for prerequisites are provided or linked to below.
- Running K8s cluster version >1.18.
- Running BIG-IP in standalone or HA cluster, where BIG-IP can route to the pod overlay network (hosted K8s services or VXLAN/BGP has been configured).
- NGINX Plus license or demo cert/key pair.
- A private container image of NGINX Plus KIC built using Docker commands outlined [here](https://docs.nginx.com/nginx-ingress-controller/installation/building-ingress-controller-image/)
- A private container registry hosting the image of your KIC image based on NGINX Plus.

## CIS (Container Ingress Services)
1. Edit the file /ingress/cis/cis1.yaml and change the IP address around line 49 so that CIS points to the BIG-IP's mgmt IP address.
2. Optionally, /ingress/cis/cis2.yaml if you have 2 BIG-IP's in an HA pair, with the same IP address change.
3. Edit the file /ingress/cis/secret_sa_rbac.yaml. The password in the secret should be the base64-encoded value of your BIG-IP admin password.
4. Install CIS with the following commands:
```bash
kubectl apply -f ingress/cis/secret_sa_rbac.yaml
kubectl apply -f ingress/cis/crd-definition/customresourcedefinitions.yaml
kubectl apply -f ingress/cis/cis1.yaml
kubectl apply -f ingress/cis/cis2.yaml
```

## NGINX KIC (Kubernetes Ingress Controller)
Install KIC using the open source, freely-available image from Docker Hub. Official instructions from NGINX are [here](https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-manifests/). For this demo I recommend you run the commands below:
````bash
    #create namespace, rbac, tls, configmap, and ingress class to support KIC
    kubectl apply -f ingress/nginx/common/ns-and-sa.yaml
    kubectl apply -f ingress/nginx/rbac/rbac.yaml
    kubectl apply -f ingress/nginx/common/default-server-secret.yaml
    kubectl apply -f ingress/nginx/common/nginx-config.yaml
    kubectl apply -f ingress/nginx/common/ingress-class.yaml
    
    #Create CRD's
    kubectl apply -f ingress/nginx/crd/k8s.nginx.org_policies.yaml
    kubectl apply -f ingress/nginx/crd/k8s.nginx.org_transportservers.yaml
    kubectl apply -f ingress/nginx/crd/k8s.nginx.org_virtualserverroutes.yaml
    kubectl apply -f ingress/nginx/crd/k8s.nginx.org_virtualservers.yaml
    
    #Run the Ingress Controller
    kubectl apply -f ingress/nginx/deployment/nginx-ingress.yaml

    #Expose NGINX ingress via cluster IP service
    kubectl apply -f ingress/nginx/service/service.yaml
````
## NGINX Plus KIC
For the purpose of demonstration, we will also deploy KIC based on NGINX Plus. This will demonstrate the [benefits of NGINX Plus](https://www.nginx.com/products/nginx/#compare-versions).
1. Save your login details for your private container registry as a K8s secret. Edit the file /ingress/nginx-plus/common/docker-login-secret.yaml and follow the [instructions from Kubernetes docs](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/) if needed. The line with ```.dockerconfigjson``` should have the base64-encoded file from your local file ```~/.docker/config.json```
2. Edit the file /ingress/nginx-plus/deployment/nginx-plus-deployment.yaml and around line #23 you will edit the location from which to pull your NGINX Plus KIC image.
3. Edit the file /ingress/nginx-plus/vs-ts.yaml around line #9 and provide the desired IP of the VIP on F5. Optionally do this with vs-ts2.yaml if you are running BIG-IP in HA.
4. Then run the following commands to install NGINX Plus Ingress Controller.
````bash
    #create namespace, rbac, tls, configmap, and ingress class to support KIC
    kubectl apply -f ingress/nginx-plus/common/ns-and-sa.yaml
    kubectl apply -f ingress/nginx-plus/rbac/rbac.yaml
    kubectl apply -f ingress/nginx-plus/common/default-server-secret.yaml
    kubectl apply -f ingress/nginx-plus/common/nginx-plus-config.yaml
    kubectl apply -f ingress/nginx-plus/common/ingress-class.yaml
    
    #Run the Ingress Controller
    kubectl apply -f ingress/nginx-plus/deployment/nginx-ingress.yaml

    #Expose NGINX ingress via cluster IP service
    kubectl apply -f ingress/nginx-plus/service/service.yaml
    
    #Create F5 TransportServer resource to expose NGINX Plus ingress controller via F5 BIG-IP
    kubectl apply -f ingress/nginx-plus/vs-ts.yaml
    kubectl apply -f ingress/nginx-plus/vs-ts2.yaml
````
## Prometheus
Prometheus is a free software application used for event monitoring and alerting. We will deploy Prometheus and again, these instructions are simply copied from another user guide. Our goal here is to run Prometheus in a pod inside Kubernenetes so that it can pull metrics from other pods, using the K8s api to discover other pods with appropriate annotations and labels.


