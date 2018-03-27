# Authentication

## Connecting to Azure

### AzureCLI

https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest

```bash
    AZ_REPO=$(lsb_release -cs)
    echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" | \
        sudo tee /etc/apt/sources.list.d/azure-cli.list
```

```bash
    sudo apt-key adv --keyserver packages.microsoft.com --recv-keys 52E16F86FEE04B979B07E28DB02C46DF417A0893
    sudo apt-get install apt-transport-https
    sudo apt-get update && sudo apt-get install azure-cli
```

```bash
    sudo apt-get update && sudo apt-get upgrade
```

### Login

```bash
    az login
```

To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code {nnnnnnnn} to authenticate.

```bash
    az account list \
        --query "[].{SubscriptionName:name,SubscriptionID:id}" \
        --out table
```

```bash
    az account set -s <SubscriptionID>
```

```bash
    az account list-locations | grep name
```

#### Table output

```bash
    az account list-locations \
        --query "[].{Region:name}" \
        --out table
```

### Resource Groups

```bash
    az group create --name usw2-appx-net-01 --location westus2
    az group create --name usw2-appx-dev-01 --location westus2
    az group create --name usw2-appx-data-01 --location westus2
    az group create --name usc-appx-spark-01 --location centralus
```

### PowerShell

```posh
    Login-AzureRmAccount
    Get-AzureRmSubscription | Select Name,Id | Sort Name
    Select-AzureRmSubscription -SubscriptionId 'nnnnnnnn-nnnn-nnnn-nnnn-nnnnnnnnnnnn'
    Get-AzureRmLocation
```