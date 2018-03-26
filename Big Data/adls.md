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
    --location westus2 \
    --name jkstoragetest

az storage account create \
    --location westus2 \
    --name jkstoragetest \
    --resource-group jkstoragetest \
    --sku Standard_LRS
```
    
### Display storage account details.
```
az storage account show \
    --name jkstoragetest \
    --resource-group jkstoragetest \
    -o table
```
    
### List storage account keys.
```
az storage account keys list \
    -n jkstoragetest \
    -g jkstoragetest \
```

