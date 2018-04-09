# Containers with K8

https://docs.microsoft.com/en-us/azure/aks/

## ACR

https://docs.microsoft.com/en-us/azure/container-registry/

https://github.com/Microsoft/MTC_ContainerCamp/blob/master/modules/kubernetes/kubernetes.md

## ACS Engine

https://github.com/Azure/acs-engine

### Creat a K8 Cluster

az provider show -n Microsoft.Network --out table

az provider show -n Microsoft.Storage --out table

az provider show -n Microsoft.Compute --out table

az provider show -n Microsoft.ContainerService --out table

az provider register -n Microsoft.Network

az provider register -n Microsoft.Storage

az provider register -n Microsoft.Compute

az provider register -n Microsoft.ContainerService

az group create --name 'rg2' --location eastus

az aks create --resource-group 'rg2' --name 'eus2-dev-k8-01' --node-count 2

az aks install-cli

az aks get-credentials --resource-group 'rg2' --name 'eus2-dev-k8-01'

az aks browse --resource-group 'rg2' --name 'eus2-dev-k8-01'