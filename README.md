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


Service account needs to have service account user permission as well to be able to deploy policies to proxy in Apigee
