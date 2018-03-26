# Storage

## Verify storage account name availability

```bash
    az storage account check-name --name 'logsusw2appxdev01'
```

## Create a storage account to hold log files and diagnostics data

```bash
    az storage account create \
        --name 'logsusw2appxdev01' \
        --resource-group 'usw-appx-data-01' \
        --location westus2 \
        --sku Standard_LRS \
        --kind StorageV2
```

## Enable firewall

```bash
    az storage account update \
        --name 'logsusw2appxdev01' \
        --resource-group 'usw-appx-data-01' \
        --default-action Deny
```

## Disable firewall

```bash
    az storage account update \
        --name 'logsusw2appxdev01' \
        --resource-group 'usw-appx-data-01' \
        --default-action Allow
```

## Add Network Rules

https://docs.microsoft.com/en-us/azure/storage/common/storage-network-security

https://docs.microsoft.com/en-us/cli/azure/storage/account/network-rule?view=azure-cli-latest
