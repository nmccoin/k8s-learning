# Basic hello-world helm chart

## Install Helm
Helm must be installed on your local workstation. Downloads can be found [here](https://helm.sh/docs/intro/install/)
## How to create
To build an extremely basic helm chart follow the directions found [here](https://helm.sh/docs/chart_template_guide/getting_started/)

The quick intro is:
* Open command prompt, type in `helm create <chartname>` and hit enter
* Delete all files from the `<chartname>/templates/` directory
* Copy all your deployment .yaml files into the `<chartname>/templates/` directory
* Install your chart to your namespace using `helm install <deploymentname> <chartname> --namespace <yournamespace>` 
  and hit enter

To delete your helm deployed app
* From command prompt, type in `helm uninstall <deploymentname>` and hit enter
