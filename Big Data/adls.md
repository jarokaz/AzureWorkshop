# Big data storage walkthrough
## Azure storage
### Getting help
```
az storage --help
az storage account --help
az storage container --help
az storage blob --help
```
### Create a storage account
Note that only Standard general-purpose storage accounts are supported with HDInsight
```

az group create \
    --location westus2 \
    --name jkstoragetest

az storage account create \
    --location westus2 \
    --name jkstoragetest \
    --resource-group jkstoragetest \
    --sku Standard_LRS
```
    
    
