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

### Create a container
```
az storage container create -n <container name>
```

### List containers in the account
```
az storage container list
```

### Upload a blob to a containers
```
az storage blob upload \
    --file data/train.txt \
    --container-name <container name> \
    --name train.txt
```

### List blobs in a container
```
az storage blob list \
    -c testcontainer
```

### Copy blobs
```
az storage blob copy start \
    --account-name <dest accountname> \
    --account-key <dest accountkey> \
    --destination-blob <dest file name> \
    --destination-container <dest container> \
    --source-uri https://sourceaccountname.blob.core.windows.net/<source container>/<source file name>
```
