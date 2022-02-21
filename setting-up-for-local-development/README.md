# Setting Up a Local Development Environment
Instructions below should assist you with preparing your laptop or workstation to allow for local development of Kubernetes applications.

## Required Software
Docker Desktop will provide a localized single node Kubernetes cluster for you to practice and test deployments. In order to have Docker Desktop available on your local device, get the name of your laptop or workstation and open a ticket with the support desk to have it added to the device collection listed below:

PUC - SWD - Docker Desktop 3.3.1 - Install

Once your device refreshes the software policy, Docker Desktop should automatically install to your device. A reboot may be required. Docker Desktop can also be found in the Company Portal once the software policy updates.

While Docker Desktop does provide it's own version of kubectl, the Kubernetes Client (kubectl) should be installed to ensure compatibility with our clusters. It can be found [here](https://kubernetes.io/docs/tasks/tools/). This is the commandline tool used to interact with the cluster.

To practice building and deploying Helm charts, the Helm 3 client binary can be downloaded from [here](https://github.com/helm/helm/releases/tag/v3.7.2)

## Docker Desktop
Docker Desktop will have to be configured to start the local Kubernetes Environment. To do this open the Docker Desktop application, go into settings, and then Kubernetes. Check the box next to 'Enable Kubernetes'. Once started, you will have a local Kubernetes environment you can interact with.

## Interacting with your Cluster(s)

### All About Contexts
Kubectl (the commandline client for interacting with clusters) stores your cluster connection details in what is called a context. As you add a new cluster to the client, it is added to your local configuration file. To view contexts currently available to you on your local workstation, open a command prompt and use the command `kubectl config get-contexts`. Below is a sample of what I see when I run the command on my laptop.
```
 dslater@UA-MAC-FVFCH175L413  ~  kubectl config get-contexts
CURRENT   NAME                     CLUSTER            AUTHINFO                                                             NAMESPACE
          aks-IzXjikftnltB-admin   aks-IzXjikftnltB   clusterAdmin_rg-aks-sec-dev-IzXjikftnltB_aks-IzXjikftnltB            twistlock
          aks-MwtGSjtX-admin       aks-MwtGSjtX       clusterAdmin_rg-aks-cloud-engineering-MwtGSjtX_aks-MwtGSjtX          cae-ami-dev
          aks-poc-westus2-admin    aks-poc-westus2    clusterAdmin_rg-aks-poc-container-westus2_aks-poc-westus2            demo-dan-slater
          aks-rOnTvaid-admin       aks-rOnTvaid       clusterAdmin_rg-aks-cloud-engineering-devops-rOnTvaid_aks-rOnTvaid   ado-agent-pool
          aks-ujjQwbNL-admin       aks-ujjQwbNL       clusterAdmin_rg-aks-vault-ujjQwbNL_aks-ujjQwbNL                      test
          aks-xcEwmVSi-admin       aks-xcEwmVSi       clusterAdmin_rg-aks-shared-container-xcEwmVSi_aks-xcEwmVSi           omics-automation-test
          aks-zVZRMHRfszDD-admin   aks-zVZRMHRfszDD   clusterAdmin_rg-aks-ce-prod-zVZRMHRfszDD_aks-zVZRMHRfszDD            san-fabric-automation
          docker-desktop           docker-desktop     docker-desktop
```

To change context (effectively changing what cluster you are interacting with) to the docker-desktop cluster, you would use the command `kubectl config use-context docker-desktop`. After running that command you should see an asterisk next under the 'Current' column next to docker-desktop if you were to issue the `kubectl config get-contexts` command again.

### Kubectl Commands
A cheatsheet for kubectl commands can be found [here](https://kubernetes.io/docs/reference/kubectl/cheatsheet/).

Some commands and examples for interacting with your cluster and their output are below.
```
 dslater@UA-MAC-FVFCH175L413  ~  kubectl get namespaces
NAME              STATUS   AGE
default           Active   20m
kube-node-lease   Active   20m
kube-public       Active   20m
kube-system       Active   20m
```

```
 dslater@UA-MAC-FVFCH175L413  ~  kubectl create namespace demo
namespace/demo created
 dslater@UA-MAC-FVFCH175L413  ~  kubectl get namespaces
NAME              STATUS   AGE
default           Active   22m
demo              Active   11s
kube-node-lease   Active   22m
kube-public       Active   22m
kube-system       Active   22m
```

I want to deploy a pod to my namespace using a yaml deployment file I have written. To do that I would use the -f switch followed by the filename of my yaml code. The .yaml code used was `kind: Deployment` so we see below we have a deployment named demo-ds.

Example
```
 dslater@UA-MAC-FVFCH175L413  ~/Desktop/demo  kubectl apply -f deployment.yaml -n demo
deployment.apps/demo-ds created
```

To issue commands against a namespace, you can either use the `-n name-of-namespace` switch or use the command `kubectl config set-context --current --namespace=name-of-namespace`. We can use either of those commands to see our pod that was deployed when we applyed our yaml in the step above.

Example of using the -n switch
```
 dslater@UA-MAC-FVFCH175L413  ~/Desktop/demo  kubectl -n demo get pods
NAME                       READY   STATUS    RESTARTS   AGE
demo-ds-6d7d9ff6b4-h8jb2   1/1     Running   0          53s
```

Example of changing default context to desired namespace
```
 dslater@UA-MAC-FVFCH175L413  ~/Desktop/demo  kubectl config set-context --current --namespace=demo
Context "docker-desktop" modified.
 dslater@UA-MAC-FVFCH175L413  ~/Desktop/demo  kubectl get pods
NAME                       READY   STATUS    RESTARTS   AGE
demo-ds-6d7d9ff6b4-h8jb2   1/1     Running   0          2m1s
```

To view deployments to a namespace, the `kubectl get deployments` command can be used. Please note, if you do not set your context to the namespace, you will have to add the `-n namespace-name` to the command.

Example
```
 dslater@UA-MAC-FVFCH175L413  ~/Desktop/demo  kubectl get deployments
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
demo-ds   1/1     1            1           16m
```

Using the command `get pods` or the command `get deployments` we see fairly similar information. When using `get pods` we see that the name of the pod, the number of ready pods, the current status, the number of restarts, and the age of the pods.

Example
```
 dslater@UA-MAC-FVFCH175L413  ~/Desktop/demo  kubectl get pods
NAME                       READY   STATUS    RESTARTS   AGE
demo-ds-6d7d9ff6b4-h8jb2   1/1     Running   0          2m1s
```
Using the command `get deployments` we see the name of the deployment, the number of pods reporting as ready, the number up-to-date, the number available, and the age of the deployment.

Example
```
 dslater@UA-MAC-FVFCH175L413  ~/Desktop/demo  kubectl get deployments
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
demo-ds   1/1     1            1           16m
```

To test if container is working correctly, without setting up a service or ingress (ingress is not recommended to be configured in local environments due to complexity), the `port-forward` command can be used to forward a port on your local device to the port the container is listening on in the kubernetes cluster. Examples below are for the deployed container, which is known to be listening on port 9000.

To port-forward directly into a pod:
```
 dslater@UA-MAC-FVFCH175L413  ~/Desktop/demo  kubectl port-forward pod/demo-ds-6d7d9ff6b4-h8jb2 9000
Forwarding from 127.0.0.1:9000 -> 9000
Forwarding from [::1]:9000 -> 9000
```
To port-forward directly into a deployment:

```
 dslater@UA-MAC-FVFCH175L413  ~/Desktop/demo  kubectl port-forward deployment/demo-ds 9000
Forwarding from 127.0.0.1:9000 -> 9000
Forwarding from [::1]:9000 -> 9000
```
To port-forward using a deployed service (note: you would need to deploy a service first, for the example below, I deployed a service expecting traffic on port 443). Port-forward cannot bind to port 443 on the local host, because of this we designate a different localhost port that is then directed to the service as port 443 traffic (which is then seamlessly forwarded to our container on port 9000)
```
 dslater@UA-MAC-FVFCH175L413  ~/Desktop/demo  kubectl port-forward service/hello-ds-service 8443:443
Forwarding from 127.0.0.1:8443 -> 9000
Forwarding from [::1]:8443 -> 9000
```
To test the connection to the container in the first two examples, I would open a web browser and connect to http://localhost:9000. The last example, traffic is not being directed to the container, but instead to the service. To validate that connection I would open a web browser and connect to http://localhost:8443. Instead of a web browser, the curl command can also be leveraged if available to validate connectivity.
```
 dslater@UA-MAC-FVFCH175L413  ~  curl localhost:9000
<html>
 <head>
 </head>
 <body>
   <h1>Hello World</h1>
    <img src="background.jpg" alt="Fur Rendezvous World Championship Sled Dog Races" width="1024" height="683">

   <p> <a href="https://git.providence.org/cloud-engineering/PSJH-AKS-Intro/releases">Click here</a> and download the latest release of the source code so you can complete the instructional module(s) on your own or with the video series</p>
   <p> The video series can be found <a href="https://providence4.sharepoint.com/sites/CloudandAutomationEngineering/Shared%20Documents/Forms/AllItems.aspx?viewid=e2e19ebb%2Da31b%2D4330%2Daa57%2D8dac5a3bb27c&id=%2Fsites%2FCloudandAutomationEngineering%2FShared%20Documents%2FAKS%20Kubernetes%2FDeployment%20and%20Support%20Process%2FDeployment%20Instructions">here</a> </p>
 </body>
</html>
```
## Helm
Helm is a package manager for Kubernetes. A main benefit of using a helm chart is that the values (configuration) of your application deployment are passed into the application via a values.yaml file. This means it's possible to have a file for qa, prod, and dev environments all while using the same chart.

### Helm Commands and Basic Usage
A list of helm commands can be found [here](https://helm.sh/docs/helm/). Below are some examples of how some of the common helm commands are used. This section will cover creating a helm chart, building a values file, and pulling a helm chart from a repository.

#### Creating a helm chart
To begin it is important to create the basic structure of the helm chart. This is accomplished using `helm create` followed by what the chart will be named. Below is the command and output, as well as the diretory structure to create a helm chart named `horace`.

     dslater@UA-MAC-FVFCH175L413  ~/Desktop/demo/example-horace  helm create horace
    Creating horace
     dslater@UA-MAC-FVFCH175L413  ~/Desktop/demo/example-horace  ls
    horace
     dslater@UA-MAC-FVFCH175L413  ~/Desktop/demo/example-horace  cd horace
     dslater@UA-MAC-FVFCH175L413  ~/Desktop/demo/example-horace/horace  ls
    Chart.yaml  charts      templates   values.yaml

The command `helm create horace` creates a directory, and then populates it with required files. A listing of default populated files are below.

     dslater@UA-MAC-FVFCH175L413  ~/Desktop/demo/example-horace/horace  ls -l
    total 16
    -rw-r--r--   1 dslater  staff  1142 Dec  2 08:42 Chart.yaml
    drwxr-xr-x   2 dslater  staff    64 Dec  2 08:42 charts
    drwxr-xr-x  10 dslater  staff   320 Dec  2 08:42 templates
    -rw-r--r--   1 dslater  staff  1873 Dec  2 08:42 values.yaml

The values.yaml and the files found in the templates directory are only examples.

The most basic usage of a helm chart will be to copy in yaml files to the templates directory after deleting all files in that directory. After removing all contents of the templates directory and copying in the yanl files, the `horace` helm chart can be deployed to the local cluster. 

#### Helm Install/Upgrade
To install a helm chart, the `helm install` or `helm upgrade --install` command is used. Format of the `helm install` command is as follows: `helm install <NameOfRelease> <NameOfChart>`. In the example below, the deployment release name is `example` and the chart used to deploy it was `horace`, created previously. If the deployment `example` had already been deployed and that deployment needed to be modified, then the helm command would be `helm upgrade --install example horace`. This command will upgrade the deployment if a previous deployment exists, otherwise, it will simply install the deployment.

     dslater@UA-MAC-FVFCH175L413  ~/Desktop/demo/example-horace  helm install example horace
    NAME: example
    LAST DEPLOYED: Thu Dec  2 09:42:31 2021
    NAMESPACE: demo
    STATUS: deployed
    REVISION: 1
    TEST SUITE: None

In the returned data above, the `NAME:` field is the release name.
#### Helm list
Helm list will list out deployments from the current session.

     dslater@UA-MAC-FVFCH175L413  ~/Desktop/demo/example-horace  helm list
    NAME   	NAMESPACE	REVISION	UPDATED                              	STATUS  	CHART       	APP VERSION
    example	demo     	1       	2021-12-02 09:42:31.788998 -0900 AKST	deployed	horace-0.1.0	1.16.0

#### Values files and how to use them
To leverage values files, the .yaml files in the templates directory must be modified. Items to modify could include the names of the ingress, container to be pulled, dns name, environment, etc. Below is an example of how to start modifying a deployment.yaml file.

Original file
```
# Always have to specify the api version used for the resource being deployed
apiVersion: apps/v1
# kind refers to the type of resource you want to deploy (Pod, Service, Deployment)
kind: Deployment
# Metadata examples are: namespace to deploy to, app name, etc
metadata:
  name: demo-ds # Name of the deployed pods
  namespace: demo # Namespace targeted for deployment
  labels: # All deployments must have an `owner, app, environment and costCenter` labels
    owner: "dan.slater.providence.org" # Contact email for app owner
    app: app-ds # the name of app
    environment: "dev" # dev, test, prod
    costCenter: "00000000" # cost center is required for chargeback/showback
  annotations:
    container.apparmor.security.beta.kubernetes.io/demo-ds: runtime/default # apparmor required for admission control
# spec - this is the desired state of the deployment, for example, what container image
# you wish to deploy, how many copies, etc
spec:
  selector:
    matchLabels:
      app: app-ds
  replicas: 1 # The number of pods you wish to have available
  template:
    metadata:
      labels:
        app: app-ds # Name to be leveraged in the Service configuration and container spec
    spec:
      securityContext: # Required for admission control
        fsGroup: 150 # directory will have group ID 101 - Valid range 100-1100
        runAsUser: 150 # Container cannot run as root user on the node - Valid range 100-1100
        runAsGroup: 150 # Valid range 100-1100
        supplementalGroups: [200] # Valid range 100-1100
      containers:
        - name: hello-world
          image: docker.io/dslaterak/hello-world:latest # This is where you specify your image location, name, and version to deploy
          securityContext:
            allowPrivilegeEscalation: false # Ensures no child process in container can gain more privileges than it's parent
            privileged: false # enforce container cannot run as root
          resources:
            # Must set min limits for your requested resources
            requests:
              memory: "64Mi"
              cpu: "250m"
            # Must set max limits for your requested resources
            limits:
              memory: "128Mi"
              cpu: "500m"
          ports:
            - containerPort: 9000
```

Section of Modified File
```
# Always have to specify the api version used for the resource being deployed
apiVersion: apps/v1
# kind refers to the type of resource you want to deploy (Pod, Service, Deployment)
kind: Deployment
# Metadata examples are: namespace to deploy to, app name, etc
metadata:
  name: {{ .Release.Name }}-demo # Name of the deployed pods
  namespace: {{ .Values.nameSpace }} # Namespace targeted for deployment
  labels: # All deployments must have an `owner, app, environment and costCenter` labels
    owner: "{{ .Values.labels.owner }}" # Contact email for app owner
```

Focusing under the `metadata` stanza, there are three sections where a variable has now replaced a staticly assigned value. 

The value .Release.Name does not have to be specified in a values.yaml file. That value is passed in when the `helm install` command is issued. For instance, if the command was `helm install example horace`, then the value for .Release.Name would be `example`.

Any value starting with .Values is assigned in the values file. An example of how that would be formatted for the other two available values are below.

Sample values.yaml file
```
nameSpace: <target-namespace>

labels:
  owner: <owner name>
```

Assume in there was also a value of {{ .Values.resources.requests.memory }}
```
nameSpace: <target-namespace>

labels:
  owner: <owner name>

resources:
  requests:
    memory: <amount of memory>
```

Example of a complete values file for a deployment containing deployment.yaml, service.yaml, and ingress.yaml files.
```
# Default values for hello-world.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.


nameSpace: demo-dan-slater

appName: sleddogs #name of the deployed app

labels:
  owner: dan.slater.providence.org
  environment: test #What app environemnt is this? dev, test, prod?
  costCenter: 00000000 #What is your valid cost center?

securityContext:
  fsGroup: 150 #directory group id number
  runAsUser: 150 #uid of user running process - cannot run as root
  runAsGroup: 150 #guid of group user belongs to

replicaCount: 2 #number of deployed containers in the replicaset - specifying more than 1 provides redundancy

containers:
  name: hello-world #name of deployed containers
  image:
    repository: docker.io/dslaterak/hello-world:latest #location to find containers
    pullPolicy: Always #define how often deployment should refresh the container when deploying
  securityContext:
    privilegeEsc: false #container cannot have privilege access to host
    priviledged: false #container cannot run as root
  nodePool: user

resources:
  requests:
    cpu: 150m
    memory: 64Mi
  limits:
    cpu: 350m
    memory: 128Mi

ports:
  protocol: TCP #ip protocol of traffic
  containerPort: 9000 #port container is communicating on
  ingressPort: 443 #port traffic is hitting ingress on

tls:
  host: dev #dns subdomain used by k8s cluster
  tlsName: sleddogs-tls
```

#### Helm Delete
Deletion of helm deployments are done via the `helm delete` command. The release name used above was `example`. To delete this release, the command would be `helm delete example`.

Example of delete command
```
 dslater@UA-MAC-FVFCH175L413  ~/Desktop/demo/example-horace  helm delete example
release "example" uninstalled
```

#### Adding Helm Repos
Vendors host and offer their own helm repos. From there charts can be downloaded or directly deployed. To add a repo to the helm client the command is `helm repo add <LocalName> <Chart-URL>`. The user provides the name for the local repo. The vendor would supply you with the url to their helm charts. In the example below `helm` was used as the local name.

Example
```
 dslater@UA-MAC-FVFCH175L413  ~/Desktop/demo/example-horace  helm repo add helm https://charts.helm.sh/stable
"helm" has been added to your repositories
```

To view repos added to the local client, the command `helm repo list` is used.

Example
```
 dslater@UA-MAC-FVFCH175L413  ~/Desktop/demo/example-horace  helm repo list
NAME 	URL
helm 	https://charts.helm.sh/stable
```


When a repo is added, it is necessary to update the repo using the `helm repo update` command. Issuing the command `helm repo update` will update all repositories that have been added to the local client. To specify a specific repo, the command `helm repo update <RepoName>` can be used. 

Example
```
 dslater@UA-MAC-FVFCH175L413  ~/Desktop/demo/example-horace  helm repo update helm
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "helm" chart repository
Update Complete. ⎈Happy Helming!⎈
```

#### Helm Search
Searching repos can be useful to see what charts are available, deprecated etc. The command used is `helm search repo <RepoName>`. The repo name is going to be the local name in the client. The example below is searching the repo named `helm`. This is not a complete listing of all charts available in this repo, just a small sample.

Example
```
 dslater@UA-MAC-FVFCH175L413  ~/Desktop/demo/example-horace  helm search repo helm
NAME                               	CHART VERSION	APP VERSION            	DESCRIPTION
helm/helm-exporter                 	0.3.3        	0.4.0                  	DEPRECATED Exports helm release stats to promet...
helm/acs-engine-autoscaler         	2.2.2        	2.1.1                  	DEPRECATED Scales worker nodes within agent pools
helm/aerospike                     	0.3.5        	v4.5.0.5               	DEPRECATED A Helm chart for Aerospike in Kubern...
helm/airflow                       	7.13.3       	1.10.12                	DEPRECATED - please use: https://github.com/air...
helm/ambassador                    	5.3.2        	0.86.1                 	DEPRECATED A Helm chart for Datawire Ambassador
helm/anchore-engine                	1.7.0        	0.7.3                  	Anchore container analysis and policy evaluatio...
helm/apm-server                    	2.1.7        	7.0.0                  	DEPRECATED The server receives data from the El...
helm/ark                           	4.2.2        	0.10.2                 	DEPRECATED A Helm chart for ark
```

#### Helm Pull
It is useful to download a copy of a chart to either add it to source control or to see exactly what the chart is doing, the different values that can be passed in, etc. To download a copy of the chart use `helm pull <ChartName>`. Below is an example of the command as well as the downloaded chart. This command is being run assuming it is desired to download the `helm-exporter` chart listed first in the `helm search` section above. A compressed version of the chart is downloaded. 

Example
```
 dslater@UA-MAC-FVFCH175L413  ~/Desktop/demo/example-horace/helm  helm pull helm/helm-exporter
 dslater@UA-MAC-FVFCH175L413  ~/Desktop/demo/example-horace/helm  ls
helm-exporter-0.3.3.tgz
```
## I am ready to move to a Development AKS Cluster, what is the onboarding process?

PSJH offers namespaces to clustomers in shared Kubernetes clusters hosted in Azure. Non-shared clusters are not supported today.

In order to have a namespace provisioned, the customer must provide a dataflow diagram and present it in the SecArch call. This process is kicked off by emailing Brian Cady for more information and documentation.

To move into PSJH environments, it's important the application team is prepared to use gitflow to control the deployment of the different environments. Users will not have access to log into kubernetes directly, instead relying on ArgoCD to complete your deployment(s). This is what makes it so important to have a working helm chart for your application, using values files for different environments.

To have your repo added to ArgoCD, you will need to provide CAE with a git link to your repo. You will also need to generate a SSH key, attaching the public key to your repo and the private key to CAE to add to ArgoCD. CAE will add your repo to ArgoCD, build out the team project, and provide rbac to ArgoCD. Once this has been completed, you will be able to log into ArgoCD and build your application deployment.