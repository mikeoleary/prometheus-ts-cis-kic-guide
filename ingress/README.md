# Setting up ingress
Follow these instructions to configure ingress to your cluster with F5 CIS and NGINX Ingress Controller

## F5 CIS (<b>C</b>ontroller <b>I</b>ngress <b>S</b>ervices)
These instructions will configure 2x CIS instances.

1. (Optional. Only required if you changed default variables.) Edit the file ````cis/secret_sa_rbac.yaml```` so that the password is the <b>base64 encoded</b> password of the admin account on F5 BIG-IP.
2. (Optional. Only required if you changed default variables.) Edit the file ````cis/cis1.yaml```` so that the fields below are updated
````bash
    "--bigip-url=`<mgmt ip of bigip>`",
    "--bigip-partition=`<partition on bigip to manage>`",
````
3. (Optional. Only required if you changed default variables.) Edit the file ````cis/cis2.yaml```` so that the fields below are updated
````bash
    "--bigip-url=`<mgmt ip of bigip>`",
    "--bigip-partition=`<partition on bigip to manage>`",
````
4. Set your kube_config file by copying the file that was created from the infra build, or set an environment variable.
````bash
    #update your kube config file
    mv ~/azure-aks-ingress-link/infra/kube_config ~/.kube/config
    #or, set an environment variable
    export KUBECONFIG=~/azure-aks-ingress-link/infra/kube_config
````
5. Then deploy CIS in your cluster with the commands:
````bash
    #create the Kubernetes secret, service account, and rbac resources in your cluster
    kubectl apply -f cis/secret_sa_rbac.yaml
    #deploy CIS
    kubectl apply -f cis/cis1.yaml
    kubectl apply -f cis/cis2.yaml
````

## NGINX Ingress Controller (referred to as KIC, for <b>K</b>ubernetes <b>I</b>ngress <b>C</b>ontroller)
These instructions will configure NGINX Ingress Controller

1.  Run the following commands to create the resources in Kubernetes that make up NGINX ingress controller.
````bash
    #create namespace, rbac, tls, configmap, and ingress class to support KIC
    kubectl apply -f nginx/common/ns-and-sa.yaml
    kubectl apply -f nginx/rbac/rbac.yaml
    kubectl apply -f nginx/common/default-server-secret.yaml
    kubectl apply -f nginx/common/nginx-config.yaml
    kubectl apply -f nginx/common/ingress-class.yaml
    
    #Create CRD's
    kubectl apply -f nginx/crd/k8s.nginx.org_policies.yaml
    kubectl apply -f nginx/crd/k8s.nginx.org_transportservers.yaml
    kubectl apply -f nginx/crd/k8s.nginx.org_virtualserverroutes.yaml
    kubectl apply -f nginx/crd/k8s.nginx.org_virtualservers.yaml
    
    #Run the Ingress Controller
    kubectl apply -f nginx/deployment/nginx-ingress.yaml
````
2. Run the commands to create a service to expose the ingress controller pods
````bash
    #Create NGINX ingress service
    kubectl apply -f nginx/service/service.yaml
````

## IngressLink
These instructions will configure IngressLink, which is a CRD that will be picked up by CIS and configure a VS on F5.
````bash
    #deploy IngressLink
    kubectl apply -f ingresslink/crd/ingresslink.yaml
    kubectl apply -f ingresslink/vs-ingresslink.yaml
    kubectl apply -f ingresslink/vs-ingresslink2.yaml
````

## Deleting your resources
This script will delete the resources you have created. 

````bash
    #delete IngressLink
    kubectl delete -f ingresslink/vs-ingresslink.yaml
    kubectl delete -f ingresslink/vs-ingresslink2.yaml
    kubectl delete -f ingresslink/crd/ingresslink.yaml

    #delete KIC
    kubectl delete -f nginx/service/service.yaml
    kubectl delete -f nginx/deployment/nginx-ingress.yaml
    kubectl delete -f nginx/common/ingress-class.yaml
    kubectl delete -f nginx/common/nginx-config.yaml
    kubectl delete -f nginx/common/default-server-secret.yaml
    kubectl delete -f nginx/rbac/rbac.yaml
    kubectl delete -f nginx/common/ns-and-sa.yaml
    #delete CRDs
    kubectl delete -f nginx/crd/k8s.nginx.org_policies.yaml
    kubectl delete -f nginx/crd/k8s.nginx.org_transportservers.yaml
    kubectl delete -f nginx/crd/k8s.nginx.org_virtualserverroutes.yaml
    kubectl delete -f nginx/crd/k8s.nginx.org_virtualservers.yaml

    #delete CIS
    kubectl delete -f cis/cis1.yaml
    kubectl delete -f cis/cis2.yaml
    kubectl delete -f cis/secret_sa_rbac.yaml
````