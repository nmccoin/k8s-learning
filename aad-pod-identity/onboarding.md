# Step by Step guide to implement AAD Pod Identity in AKS Cluster 

 In this document, we are going to detail the steps to configure Azure AD Pod Identity for connecting pods in AKS Cluster with Azure Key Vault. Even though this demonstration is done using KeyVault, it should not be any different to connect to other Azure services. All the steps from 1 to 6 have to be done by Application Teams and reach out to Cluster Admins for any help during the process. 
 
 
## 1. Create an Azure Managed Identity
Application teams must create Managed User Identity to access their KeyVault. Once we create the Identity, it looks like below.
![image](https://git.providence.org/storage/user/339/files/157cd480-5cd3-11ec-9bf7-593096bb1acb?raw=true)

## 2. Create an Azure KeyVault and add Secret
Go to Azure Portal and create an Azure KeyVault
Add Secret to KeyVault
![image](https://git.providence.org/storage/user/339/files/4a1b8980-5dc2-11ec-9185-b5dd22b24cdf)


## 3. Assign Managed Identity Operator Role for Cluster NodePool Identity on Azure Managed Identity
Assign `Managed Identity Operator Role` on Azure Managed Identity like below. Please contact your cluster administrator to get the Cluster NodePool Identity.
For current deployments, use following nodepools<br/>

| Cluster          | Agent Pool Name            |
| ---------------- | -------------------------- |
| Dev/Test Cluster | aks-xcEwmVSi-agentpool     |
| Prod Cluster     | aks-zVZRMHRfszDD-agentpool |

![image](https://git.providence.org/storage/user/339/files/12621100-5dc5-11ec-88b7-aab621fb18d2)


Failing to complete this step will result in below error


```

E1216 00:29:01.522258       1 mic.go:1111] failed to update user-assigned identities on node aks-system-12583646-vmss (add [1], del [0], update[0]), error: failed to update identities for aks-system-12583646-vmss in MC_rg-aks-poc-container-westus2, error: compute.VirtualMachineScaleSetsClient#Update: Failure sending request: StatusCode=0 -- Original Error: Code="LinkedAuthorizationFailed" Message="The client 'b9276f33-e80c-4f09-9cdf-24c28c0feea4' with object id 'b9276f33-e80c-4f09-9cdf-24c28c0feea4' has permission to perform action 'Microsoft.Compute/virtualMachineScaleSets/write' on scope '/subscriptions/7e242177-ef0b-431f-8c76-e40907843aa7/resourceGroups/MC_rg-aks-poc-container-westus2/providers/Microsoft.Compute/virtualMachineScaleSets/aks-system-12583646-vmss'; however, it does not have permission to perform action 'Microsoft.ManagedIdentity/userAssignedIdentities/assign/action' on the linked scope(s) '/subscriptions/99a3af26-911d-4bc5-8fcc-8f9de81e46c5/resourcegroups/rg-westus2-shared/providers/microsoft.managedidentity/userassignedidentities/aks-aad-user-identity' or the linked scope(s) are invalid."
E1216 00:29:02.006234       1 mic.go:1129] failed to apply binding aad-pod-identity-poc/aad-user-identity-binding node aks-system-12583646-vmss000001 for pod aad-pod-identity-poc/az-keyvault-reader-test-77b46877d6-6kj24, error: failed to update identities for aks-system-12583646-vmss in MC_rg-aks-poc-container-westus2, error: compute.VirtualMachineScaleSetsClient#Update: Failure sending request: StatusCode=0 -- Original Error: Code="LinkedAuthorizationFailed" Message="The client 'b9276f33-e80c-4f09-9cdf-24c28c0feea4' with object id 'b9276f33-e80c-4f09-9cdf-24c28c0feea4' has permission to perform action 'Microsoft.Compute/virtualMachineScaleSets/write' on scope '/subscriptions/7e242177-ef0b-431f-8c76-e40907843aa7/resourceGroups/MC_rg-aks-poc-container-westus2/providers/Microsoft.Compute/virtualMachineScaleSets/aks-system-12583646-vmss'; however, it does not have permission to perform action 'Microsoft.ManagedIdentity/userAssignedIdentities/assign/action' on the linked scope(s) '/subscriptions/99a3af26-911d-4bc5-8fcc-8f9de81e46c5/resourcegroups/rg-westus2-shared/providers/microsoft.managedidentity/userassignedidentities/aks-aad-user-identity' or the linked scope(s) are invalid."
```
## 4. Set access policies for Azure Identity in KeyVault

Please add access policies to provide necessary permissions to Azure Managed Identity on KeyVault as shown in following picture.

![image](https://git.providence.org/storage/user/339/files/e6935b00-5dc5-11ec-9c61-987377fd00ee)


Failing to complete this step will result in below errors


```
2021/12/16 00:38:19 failed to retrieve Keyvault secret versions: keyvault.BaseClient#GetSecretVersions: Failure responding to request: StatusCode=403 -- Original Error: autorest/azure: Service returned an error. Status=403 Code="Forbidden" Message="The user, group or application 'appid=248b3285-aa4c-436f-b0da-8ab35ba2d14a;oid=bfcf34d4-cde4-48dd-b7d7-6c3db4ff884c;iss=https://sts.windows.net/2e319086-9a26-46a3-865f-615bed576786/' does not have secrets list permission on key vault 'kv-westus2-aks;location=westus2'. For help resolving this issue, please see https://go.microsoft.com/fwlink/?linkid=2125287" InnerError={"code":"AccessDenied"}
```
## 5. Deploy AzureIdentity and AzureIdentityBinding in AKS Cluster

Create deployment for AzureIdentity and AzureIdentityBinding

```
  apiVersion: "aadpodidentity.k8s.io/v1"
  kind: AzureIdentity
  metadata:
    name: aad-user-identity # Name of Azure Identity
    namespace: aad-pod-identity-poc # Namespace where this AzureIdentity will be created
  spec:
    type: 0 # User Assigned Identity : 0
     resourceID: /subscriptions/99a3af26-911d-4bc5-8fcc-8f9de81e46c5/resourceGroups/rg-westus2-shared/providers/Microsoft.ManagedIdentity/userAssignedIdentities/aks-aad-user-identity
     clientID: 248b3285-aa4c-436f-b0da-8ab35ba2d14a
   ---
   apiVersion: "aadpodidentity.k8s.io/v1"
   kind: AzureIdentityBinding
   metadata:
     name: aad-user-identity-binding
     namespace: aad-pod-identity-poc
   spec:
     azureIdentity: aad-user-identity
     selector: "bind-user-identity-keyvault"
 ```
## 6. Modify Application pod deployment to refer to IdentityBinding

An example of KeyVault Reader test yaml with aadpodidbinding is below

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: az-keyvault-reader-test
  namespace: aad-pod-identity-poc
spec:
  replicas: 1
  selector:
    matchLabels:
      app: az-keyvault-reader-test
      aadpodidbinding: bind-user-identity-keyvault
  template:
    metadata:
      labels:
        app: az-keyvault-reader-test
        aadpodidbinding: bind-user-identity-keyvault
    spec:
      containers:
        - name: busybox
          image: busybox
          command:
            [
              "sh",
              "-c",
              "wget --tries=70 -qO- http://127.0.0.1:8333/secrets/grafana-admin-username-poc/",
            ]
          imagePullPolicy: Always
          resources:
            requests:
              memory: "4Mi"
              cpu: "100m"
            limits:
              memory: "8Mi"
              cpu: "200m"
        - name: az-keyvault-reader-sidecar
          image: cmendibl3/az-keyvault-reader:0.2
          imagePullPolicy: Always
          env:
            - name: AZURE_KEYVAULT_NAME
              value: kv-westus2-aks
          resources:
            requests:
              memory: "8Mi"
              cpu: "100m"
            limits:
              memory: "16Mi"
              cpu: "200m"
      restartPolicy: Always
```
    
## 7. Assign Roles for Agent Pool Identity of the cluster on MC_ClusterName Resource Group (Done by Cluster Admins )
Once we add agentpool Identity of the cluster to `Reader` and `Virtual Machine Contributor` roles on MC_ClusterName resource group, it looks like below.

![image](https://git.providence.org/storage/user/339/files/c95d8d00-5dc3-11ec-8187-167745f27f54)


If we try to access a secret in KeyVault without performing above step, we'll see below errors

```
E1215 23:46:00.914853       1 mic.go:1111] failed to update user-assigned identities on node aks-system-12583646-vmss (add [1], del [0], update[0]), error: failed to get identity resource, error: failed to get vmss aks-system-12583646-vmss in resource group MC_rg-aks-poc-container-westus2, error: failed to get vmss aks-system-12583646-vmss in resource group MC_rg-aks-poc-container-westus2, error: compute.VirtualMachineScaleSetsClient#Get: Failure responding to request: StatusCode=403 -- Original Error: autorest/azure: Service returned an error. Status=403 Code="AuthorizationFailed" Message="The client 'b9276f33-e80c-4f09-9cdf-24c28c0feea4' with object id 'b9276f33-e80c-4f09-9cdf-24c28c0feea4' does not have authorization to perform action 'Microsoft.Compute/virtualMachineScaleSets/read' over scope '/subscriptions/7e242177-ef0b-431f-8c76-e40907843aa7/resourceGroups/MC_rg-aks-poc-container-westus2/providers/Microsoft.Compute/virtualMachineScaleSets/aks-system-12583646-vmss' or the scope is invalid. If access was recently granted, please refresh your credentials."
E1215 23:46:01.260587       1 mic.go:1114] failed to get a list of user-assigned identites from node aks-system-12583646-vmss, error: failed to get identity resource, error: failed to get vmss aks-system-12583646-vmss in resource group MC_rg-aks-poc-container-westus2, error: failed to get vmss aks-system-12583646-vmss in resource group MC_rg-aks-poc-container-westus2, error: compute.VirtualMachineScaleSetsClient#Get: Failure responding to request: StatusCode=403 -- Original Error: autorest/azure: Service returned an error. Status=403 Code="AuthorizationFailed" Message="The client 'b9276f33-e80c-4f09-9cdf-24c28c0feea4' with object id 'b9276f33-e80c-4f09-9cdf-24c28c0feea4' does not have authorization to perform action 'Microsoft.Compute/virtualMachineScaleSets/read' over scope '/subscriptions/7e242177-ef0b-431f-8c76-e40907843aa7/resourceGroups/MC_rg-aks-poc-container-westus2/providers/Microsoft.Compute/virtualMachineScaleSets/aks-system-12583646-vmss' or the scope is invalid. If access was recently granted, please refresh your credentials."
```

## 8. Verify AzureAssignedIdentity got created in your namespace (Done by Cluster Admins)

Once all the configuration is completed successfully, we'll see an AzureAssignedIdentity created in default namespace

