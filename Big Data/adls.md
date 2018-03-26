# Big data storage walkthrough
## Azure storage
### Getting help.
```
az storage --help
az storage account --help
az storage container --help
az storage blob --help
```
### Create a storage account.
Note that only Standard general-purpose storage accounts are supported with HDInsight
```

az group create \
    --location <location> \
    --name <resource group name>

az storage account create \
    --location <location> \
    --name <account name> \
    --resource-group <resource group name> \
    --sku Standard_LRS
```
    
### Display storage account details.
```
az storage account show \
    --name <account name> \
    --resource-group <resource group name> \
    -o table
```
    
### List storage account keys.
```
az storage account keys list \
    -n <account name> \
    -g <resource group name> \
    -o table
```

### Set default storage account environment variables
```
export AZURE_STORAGE_ACCOUNT=<account name>
export AZURE_STORAGE_ACCESS_KEY=<access key>
```
