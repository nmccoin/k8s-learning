# Randomized Helm Chart
In this example, we start by creating a helm chart, removing the contents of the templates directory, and copy in our
.yaml files again. Instead of stopping there, we will randomize the .yaml files, build a values file, and then deploy
our app using our custom made values file.

## Benefits over basic chart deployment
Setting up the .yaml to use values files allows you to resuse the same chart for multiple deployments

## Configuring your .yaml files
All the files are built the same way. We will just evaluate a portion of the deployment.yaml file.

Helm has some built-in variables, like .Release.Name. Other variables you put into the values.yaml file. In the code
block below, you can see for the name, we are using the `Release.Name` variable that will be passed in when we begin
the deployment of the helm chart. Variables like `Values.labels.owner` are passed in from the values.yaml file. Values
passed from the values file will start with `Values.`.

`metadata:`

`name: {{ .Release.Name }}-demo # Name of the deployed pods`

`namespace: {{ .Values.Namespace }} # Namespace targeted for deployment`

`labels: # All deployments must have an owner, app, environment and costCenter labels`

`owner: "{{ .Values.labels.owner }}" # Contact email for app owner`

`app: {{ .Values.appName }} # the name of app`

`environment: "{{ .Values.labels.environment }}" # dev, test, prod`

`costCenter: "{{ .Values.labels.costCenter }}" # cost center is required for chargeback/showback`

## Configuring your values.yaml file
The code block below is from the values.yaml file. The formatting is very important, as it almost builds a 'path' that
needs to be followed to get the correct value in the correct place in your deployment. We see that the `appName` isn't
indented over, meaning that variable can be called using a field `{{ .Values.appName }}`. The owner, environment, and 
costCenter variables would need to include the `.labels` so that helm grabs the correct item when parsing the values
file. For example, owner would be formatted like `{{ Values.labels.owner }}` in the deployment file.

`appName: sleddogs #name of the deployed app`

`labels:`

  `owner: dan.slater.providence.org`
  
  `environment: test #What app environemnt is this? dev, test, prod?`
  
  `costCenter: 00000000 #What is your valid cost center?`

## Building alternate values files
To build your own values file, create a new yaml file, copy the contents of the existing values file, paste them into
the new file, modify the values you wish to pass in, and then save the file.

## Install using a custom values file
If you created a custom values file called dogsncats.yaml and wanted to deploy the helm chart using that file, the 
command would be `helm install -f dogsncats.yaml <deploymentname> <chartname>`
