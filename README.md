

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


Next start with the apim operator crds

create an application in ArgoCD as shown in the image

![Image of screenshot](/media/argocd-application-crds-yaml.png)

Once saved, go to the ArgoCD applicatoins view and create the crds by clicking on the sync button to trigger the creation of the crds, the application should show sync ok as below

![Image of screenshot](/media/argocd-crd-app-sync.png)

Next will be to create the gateway and httproute and httpbin backend applications.
 
![Image of screenshot](/media/argocd-gw-httproute-app.png)


 Next create the apim operator application which will create some  kubernetes objects for the operator such as operator cluster role, kubernetes service account, kubernetes pod all for the operator, 
It is worth mentioning that the operator does require some values to be configured as override values such as project, apigee org and service account to be used.

![Image of screenshot](/media/operatorwithoverrides.png)

![Image of screenshot](/media/argocd-operator-app-structure-complete.png)

The operator and other components my take a while to create, to confirm its all created use the below command to confirm its in running status as below.

Next step is to create a group of objects we consider as phase1 objects which will create the api product, api operation set and apim extension policy objects.

![Image of screenshot](/media/phase1-app.png)

you can run following command to confirm when the apim extension policy gets into running state.

```
kubectl logs  apigee-apim-operator-7b98fdd44d-9blsk -n apim | grep "\"status\": \"RUNNING\""
```

![Image of screenshot](/media/operatorwithoverrides.png)


it may take a while for these changes to reflect in argocd application so may be worth doing a re-sync after a while.


After the phaes 1 is done, create an Apigee application and apigee developer in the apigee ui and link them to the api product created earlier


Last step is to creat the phase 2 application in argocd which contains the template-rule, admin template and some apigee policy templates
![Image of screenshot](/media/all-apps-phase1-2.png)



after this is done you can now begin some testing as shown in the links below.


Test API Key Enforcement Now we have created an APP in Apigee and have an API Key, we can use the following command to send a request to the Gateway using the API key to test API key enforcement.

     curl http://GATEWAY_IP_ADDRESS/get -H "Host: HOST_NAME" -H "x-api-key: API_KEY"
Where GATEWAY_IP_ADDRESS is the ip address of the Gateway. HOST_NAME is the hostname defined in the Gateways HTTPRoute/ API_KEY is the API Key value obtained from running script 8 above. You can retrieve the Gateway IP address using te follwoing command

    kubectl get gateways.gateway.networking.k8s.io GATEWAY_NAME -o=jsonpath="{.status.addresses[0].value}"
The request should succeed and return a response similar to the following:

![Image of screenshot](/media/apikeyenforce.png)


 Test Quota Enforcement To test the quota enforcement defined in your APIM extension policy, send the request from the previous step to the Gateway ten times within the span of one minute.

You can run the following script to generate the requests:

    #!/bin/sh
for i in $(seq 1 11); do
    curl http://GATEWAY_IP_ADDRESS/get -H "Host: HOST_NAME" -H "x-api-key: API_KEY"
    sleep 1
done
This action should trigger a quota violation and raise a fault similar to the following:

    {"fault":{"faultstring":"Rate limit quota violation. Quota limit  exceeded. Identifier : _default","detail":{"errorcode":"policies.ratelimit.QuotaViolation"}}}


Test REST Operations Enforcement To test the rest operations enforcement, use the following command to send a request to the Gateway using a URL that is not in the API operation set:

    curl http://GATEWAY_IP_ADDRESS/post -H "Host: HOST_NAME" -H "x-api-key: API_KEY"
The request should fail with a response similar to the following:

    {"fault":{"faultstring":"Invalid ApiKey for given resource","detail":{"errorcode":"oauth.v2.InvalidApiKeyForGivenResource"}}}
Congratulations!!! You have Successfully setup the Apigee APIM Operatpr!!!

You can now clean up the resources created by deleting the argocd in a sequence. As of the time of creating this repo, the Apigee App and Apigee developer need to be creted manually, 

1. Delete the phase 2 app in Argocd
2. Delete the Apigee App and Developer in the Apigee UI
3. Detach the Apigee environment from the Apigee instance
4. Delete the other Argocd Apps for complete clean up

