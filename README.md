az aks get-credentials --name my-kube-cluster --resource-group mycluster-rg
docker build -t kube-nginx:v1 .
docker login myclusteracr.azurecr.io
docker tag kube-nginx:v1 myclusteracr.azurecr.io/kube-nginx:v1
docker push myclusteracr.azurecr.io/kube-nginx:v1

docker run --name kub-nginx --rm -p 80:80 -d myclusteracr.azurecr.io/kube-nginx:v1

az aks update -n my-kube-cluster -g mycluster-rg --attach-acr myclusteracr 
kubectl apply -f deployment.yaml

kubectl delete -f deployment.yaml
az aks update -n my-kube-cluster -g mycluster-rg --detach-acr myclusteracr 

<!-- kubectl logs -f $(kubectl get po -n kube-system | egrep -o 'aci-connector-linux-[A-Za-z0-9-]+') -n kube-system -->

#!/bin/bash
# This script requires Azure CLI version 2.25.0 or later. Check version with `az --version`.

# Modify for your environment.
# ACR_NAME: The name of your Azure Container Registry
# SERVICE_PRINCIPAL_NAME: Must be unique within your AD tenant
ACR_NAME=acrdemo2ss
SERVICE_PRINCIPAL_NAME=acr-sp-demo

# Obtain the full registry ID for subsequent command args
ACR_REGISTRY_ID=$(az acr show --name $ACR_NAME --query id --output tsv)

# Create the service principal with rights scoped to the registry.
# Default permissions are for docker pull access. Modify the '--role'
# argument value as desired:
# acrpull:     pull only
# acrpush:     push and pull
# owner:       push, pull, and assign roles
SP_PASSWD=$(az ad sp create-for-rbac --name $SERVICE_PRINCIPAL_NAME --scopes $ACR_REGISTRY_ID --role acrpull --query password --output tsv)
SP_APP_ID=$(az ad sp list --display-name $SERVICE_PRINCIPAL_NAME --query [].appId --output tsv)

# Output the service principal's credentials; use these in your services and
# applications to authenticate to the container registry.
echo "Service principal ID: $SP_APP_ID"
echo "Service principal password: $SP_PASSWD"

kubectl create secret docker-registry  my-secret \
-- namespace default \
-- docker-server=myclusteracr.azurecr.io \
--docker-username=6a97aebc-148f-422a-86b5-0e79623dcca2 \
--docker-assword=PWi8Q~YDRzLenhZgNv-KKJe5E24bYWbuI2Tgocfj

echo -n '{"auths": {"myclusteracr.azurecr.io": {"username": "6a97aebc-148f-422a-86b5-0e79623dcca2", "password": "PWi8Q~YDRzLenhZgNv-KKJe5E24bYWbuI2Tgocfj"}}}' | base64
