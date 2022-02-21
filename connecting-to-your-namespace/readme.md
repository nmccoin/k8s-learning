# Connecting to your PSJH Provisioned Kubernetes Namespace

## Prerequesites

### Software
Prior to connecting to the namespace you will need to have Azure Commandline Tools and Kubectl installed on your local
workstation. This will not work inside of Azure cloud shell. Links to these tools are below.

#### Azure CLI
* [Azcli installation process](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)

#### Kubernetes client - kubectl
* [kubectl installation process](https://kubernetes.io/docs/tasks/tools/)

### Provisioned Namespace
After your namespace has been provisioned, you should have received an email with some connection information looking
something like this (the information below may not reflect the information in the email you receive).
* az account set --subscription "52282b21-4eb3-49a7-babe-743ef9-DO-NOT-USE"
* az aks get-credentials --resource-group rg-aks-shared-container-xcEwmVSi --name aks-xcEwmVSi

### Required Permissions(performed by your AKS admin)
Add the AAD Security Group to following roles for the AKS Cluster
- Azure Kubernetes Service Cluster User Role
- Azure Kubernetes Service RBAC Reader
- Reader

## Connecting to the Cluster
Open a command prompt and issue the following command to log into Azure:
* c:\az login

Run the two commands that were provided in the email you received (reminder, yours may differ):
* c:\az account set --subscription "52282b21-4eb3-49a7-babe-743ef9-DO-NOT-USE"
* c:\az aks get-credentials --resource-group rg-aks-shared-container-xcEwmVSi --name aks-xcEwmVSi

When the second command completes successfully, you will see some returned verbage like this (your verbage will differ
based on the path to your .kube/config file):
* Merged "aks-xcEwmVSi" as current context in /Users/dslater/.kube/config

## Issuing commands
You will not be able to issue commands against resouces outside your namespace. It is important to specify your
namespace when you are issuing commands. This is done using the `-n <namespace>` switch. Examples are listed below:
* c:\kubectl -n `<namespace>` get pods
* c:\kubectl -n `<namespace>` get deployments
* c:\kubectl -n `<namespace>` get services
* c:\kubectl -n `<namespace>` get ingress

These are just a few of the commands that can be used. Not all commands will be available to you. For more information
please reference the [kubectl reference docs](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands)
