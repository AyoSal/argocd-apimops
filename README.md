# argocd-apimops


# Deploy Apigee Apim Operator with ArgoCD Automation

This sample demonstrates how to deploy the Apigee APIM Operator ina GitOps fashion using ArgoCD automation with source code in this github repo, synced with ArgoCD to the GKE cluster and to the GKE cluster and to Apigee.

The Apigee APIM Operator is designed to support cloud-native developers by providing a command-line interface that integrates with familiar Kubernetes tools like kubectl. The operator works by using various APIM resources to keep your Google Kubernetes Engine cluster synchronized with the Apigee runtime.

The Operator allows you to define and manage API management aspects directly within your Kubernetes cluster using the same YAML files and kubectl commands you already use for your other Kubernetes resources (like deployments, services, etc). The APIM Operator brings Apigee's powerful API management capabilities right into your Kubernetes workflow.



Prerequisites

Provision Apigee X, ensure Apigee X is created in the same region as the GKE cluster created in Create GKE Kubernetes Cluster step
Have access and permissions in Google Cloud IAM to create required IAM roles and Google service accounts required
Have access to deploy API Proxies in Apigee,
Have access to create Environments and Environment Groups in Apigee
Have access to create API Products, Developers, and Developer Apps in Apigee
Have access to provision Load Balancer Resources (ip address, forwarding rule, url map, backend service, NEGs, etc)
Have access to create Load Balancer Service Extensions
Make sure the following tools are available in your terminal's $PATH (Cloud Shell has these preconfigured)
gcloud SDK
curl
jq


Step 1

Start by setting required variables,  installing your GKE cluster and creating the required IAM roles and service accounts
Run scripts 1,2 and 3 from this [repo](https://github.com/AyoSal/apim-operator)



# Setup ArgoCD in your GKE Cluster

Create the ArgoCD namespace 
```
  kubectl create ns argocd
```

Install ArgoCD into your cluster
```
 kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```


Patch Argocd service from ClusterIP to loadbalancer 
```
kubectl -n argocd patch svc argocd-server --type='json' -p '[{"op":"replace","path":"/spec/type","value":"LoadBalancer"}]'
```

Download the ArgoCD biinary by followng steps at this [page[(https://github.com/argoproj/argo-cd/releases/tag/v2.9.2) based on your operating system. 
Once ArgoCD is installed proceed to the next step.

Retrieve the initial password for Argo cd with -

```
 argocd admin initial-password -n argocd
```

Retrieve the external IP of the ArgoCD service and use that to log into the ArgoCD UI with the initial password received above, change this password to one of your choice. 

Re-login with your new password.

Click on settings tile and connect to git repo. Create repositories required for ArgoCD to pull the helm charts of the components as shown below, 


![Image of screenshot](/media/repo-setup.png)


For connecting the gitHub repo to ArgoCD, you may need to generate ssh keys and share between github and ArgoCD.
Once the repo is created successfully, you will see a green tick as shown below.

![Image of screenshot](/media/argocd-repos.png)

