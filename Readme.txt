Create Service Account and binding it to a cluster role
Now we have the Cluster Role created lets create the service account and bind it to a role

$ kubectl apply -f -<<EOF
 apiVersion: v1
 kind: ServiceAccount
 metadata:
   name: azure-devops-svc
EOF
serviceaccount/azure-devops-svc created

$ kubectl create clusterrolebinding azure-devops-role-binding-svc --clusterrole=create-deployments --serviceaccount=default:azure-devops-svc clusterrolebinding.rbac.authorization.k8s.io/azure-devops-role-binding-svc created
Testing the RBAC Access of the service account
Kubernetes has a feature to allow you to impersonate an account and see what level of permissions it may have, to do this we will run

kubectl auth can-i <verb> <resource>-n <namespace> --as=system:serviceaccount:<ServiceAccountName>:<Namespace>

This will return a yes or no for the action you are trying to complete.

An example of this:

$ kubectl auth can-i create pods --as=system:serviceaccount:default:azure-devops-svc
yes
$ kubectl auth can-i create pods -n montytest--as=system:serviceaccount default:azure-devops-svc
yes
$ kubectl auth can-i create namspaces --as=system:serviceaccount default:azure-devops-svc
Warning: the server doesn't have a resource type 'namspaces'
no
Creating Service Connection in Azure DevOps
Now we have the service account created and tested the permissions are as per our standard we can go ahead and create the service connection, for this logging into our Azure DevOps instance and selecting “Create Service Connection > Kubernetes Service Connection”


Selecting Service Account, we require 2 details from our cluster

1) The API Server URL

$ kubectl config view --minify -o jsonpath={.clusters[0].cluster.server}
https://montytesta-p-montytest-dev--a5c4ea-***.hcp.australiaeast.azmk8s.io:443
2) The Service Account Secret

kubectl get serviceAccounts <service-account-name> -n <namespace> -o=jsonpath={.secrets[*].name}

kubectl get secret <service-account-secret-name (Output from previous line> -n <namespace> -o json

This will create a JSON Output you will need to copy and paste it into your Azure DevOps service connection.


Save this and you are now ready to deploy your application from Azure DevOps into your K8s cluster

To reference this in your pipeline you there is a task you can create for kubectl


OR as a YAML Deployment

- task: Kubernetes@1
  inputs:
    connectionType: 'Kubernetes Service Connection'
    kubernetesServiceEndpoint: 'montytest-k8s-cluster'
    namespace: 'default'
    command: 'apply'
    arguments: '-f nginx-testbuild-aks.yaml'
    secretType: 'dockerRegistry'
    containerRegistryType: 'Azure Container Registry'
Running this in the pipeline:


Tags: DevOps, Kubernetes, rbac, 
