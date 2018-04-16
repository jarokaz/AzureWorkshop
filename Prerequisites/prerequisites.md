# Prerequisites

## Install the AzureCLI

Overview: https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest

apt: https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-apt?view=azure-cli-latest

yum: https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-yum?view=azure-cli-latest

Add the AzureCLI source repo to the distribution (apt)

1. Modify your sources list

    ```bash
        AZ_REPO=$(lsb_release -cs)
        echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" | \
            sudo tee /etc/apt/sources.list.d/azure-cli.list
    ```

2. Get the Microsoft signing key:

    ```bash
        sudo apt-key adv --keyserver packages.microsoft.com --recv-keys 52E16F86FEE04B979B07E28DB02C46DF417A0893
    ```

3. Install AzureCLI

    ```bash
        sudo apt-get install apt-transport-https
        sudo apt-get update && sudo apt-get install azure-cli
    ```

4. Udpate

    ```bash
        sudo apt-get update && sudo apt-get upgrade
    ```

## Connect to Azure

1. Login

    ```bash
        az login
    ```

    To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code {nnnnnnnn} to authenticate.

2. List available subscriptions

    ```bash
        az account list \
            --query "[].{SubscriptionName:name,SubscriptionID:id}" \
            --out table
    ```

3. Set the active subscription

    ```bash
        az account set -s <SubscriptionID>
    ```

### Show Azure Regions

```bash
    az account list-locations -o table
```

This is an exmpale showing how to format the JSON output (http://jmespath.org/ - query language for JSON)

```bash
    az account list-locations \
        --query "[].{Region:name}" \
        --out table
```

### Show Resource Groups

```bash
    az group list --out table
```

### Create a Resource Group

```bash
    az group create --name '<resource group name>' --location <region>
```

## Diagnostic Storage Account

### Verify storage account name availability

**NOTE**: Storage account names have a global scope. Please choose a unique name.

Azure naming conventions and scopes: https://docs.microsoft.com/en-us/azure/architecture/best-practices/naming-conventions

```bash
    az storage account check-name --name '<storage account name>'
```

### Create a storage account to hold diagnostics logs

```bash
    az storage account create \
        --name '<storage account name>' \
        --resource-group '<resource group name>' \
        --location <region> \
        --sku Standard_LRS \
        --kind StorageV2
```

### Enable the Storage Account firewall

https://docs.microsoft.com/en-us/azure/storage/common/storage-network-security

https://docs.microsoft.com/en-us/cli/azure/storage/account/network-rule?view=azure-cli-latest

```bash
    az storage account update \
        --name '<storage account name>' \
        --resource-group '<resource group name>' \
        --default-action Deny \
        --bypass Logging Metrics AzureServices
```

### Disable the Storage Account firewall

```bash
    az storage account update \
        --name '<storage account name>' \
        --resource-group '<resource group name>' \
        --default-action Allow
```