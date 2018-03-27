# Networking

## Authentication

### Connecting to Azure

#### AzureCLI

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

#### Login

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

##### Table output

```bash
    az account list-locations \
        --query "[].{Region:name}" \
        --out table
```

#### Resource Groups

```bash
    az group create --name usw2-appx-net-01 --location westus2
    az group create --name usw2-appx-dev-01 --location westus2
    az group create --name usw2-appx-data-01 --location westus2
    az group create --name usc-appx-spark-01 --location centralus
```

## Storage

### Verify storage account name availability

```bash
    az storage account check-name --name 'logsusw2appxdev01'
```

### Create a storage account to hold log files and diagnostics data

```bash
    az storage account create \
        --name 'logsusw2appxdev01' \
        --resource-group 'usw2-appx-data-01' \
        --location westus2 \
        --sku Standard_LRS \
        --kind StorageV2
```

```bash
    az storage account create \
        --name 'sparkusw2appxdev01' \
        --resource-group 'usw2-appx-data-01' \
        --location westus2 \
        --sku Standard_LRS \
        --kind StorageV2
```

```bash
    az storage account create \
        --name 'sparkuscappxdev01' \
        --resource-group 'usw2-appx-data-01' \
        --location centralus \
        --sku Standard_LRS \
        --kind StorageV2
```

### Enable firewall

```bash
    az storage account update \
        --name 'logsusw2appxdev01' \
        --resource-group 'usw2-appx-data-01' \
        --default-action Deny \
        --bypass Logging Metrics AzureServices
```

### Disable firewall

```bash
    az storage account update \
        --name 'logsusw2appxdev01' \
        --resource-group 'usw2-appx-data-01' \
        --default-action Allow
```

### Add Network Rules

https://docs.microsoft.com/en-us/azure/storage/common/storage-network-security

https://docs.microsoft.com/en-us/cli/azure/storage/account/network-rule?view=azure-cli-latest

## Networks

https://docs.microsoft.com/en-us/azure/networking/networking-virtual-datacenter

### Show Existing Networks

```bash
    az network vnet list
```

```bash
    az network vnet list --out table
```

### Create a Virtual Network (vnet) with a subnet called 'web'

```bash
    az network vnet create \
        -n 'usw2-appx-dev-01' \
        -g 'usw2-appx-net-01' \
        --address-prefix 172.17.0.0/16 \
        --subnet-name web \
        --subnet-prefix 172.17.10.0/24
```

```bash
    az network vnet create \
        -n 'usw2-appx-dev-02' \
        -g 'usw2-appx-net-01' \
        --address-prefix 172.18.0.0/16 \
        --subnet-name web \
        --subnet-prefix 172.18.10.0/24
```

```bash
    az network vnet create \
        -n 'usc-appx-spark-01' \
        -g 'usc-appx-spark-01' \
        --address-prefix 172.19.0.0/16 \
        --subnet-name hdi \
        --subnet-prefix 172.19.10.0/24
```

### Create a network security group (nsg) for each subnet

```bash
    az network nsg create -g usw2-appx-net-01 -n 'usw2-appx-dev-web'
    az network nsg create -g usw2-appx-net-01 -n 'usw2-appx-dev-app'
    az network nsg create -g usw2-appx-net-01 -n 'usw2-appx-dev-data'
```

### Associate the 'web' network security group (nsg) with the corresponding subnet

```bash
    az network vnet subnet update \
        -n web \
        -g 'usw2-appx-net-01' \
        --vnet-name 'usw2-appx-dev-01' \
        --network-security-group 'usw2-appx-dev-web'
```

```bash
    az network vnet subnet update \
        -n web \
        -g 'usw2-appx-net-01' \
        --vnet-name 'usw2-appx-dev-02' \
        --network-security-group 'usw2-appx-dev-web'
```

### Add 'app' subnet and associate corresponding network security group

```bash
    az network vnet subnet create \
        -n app \
        -g 'usw2-appx-net-01' \
        --vnet-name 'usw2-appx-dev-01' \
        --address-prefix 172.17.20.0/24 \
        --network-security-group 'usw2-appx-dev-app'
```

```bash
    az network vnet subnet create \
        -n app \
        -g 'usw2-appx-net-01' \
        --vnet-name 'usw2-appx-dev-02' \
        --address-prefix 172.18.20.0/24 \
        --network-security-group 'usw2-appx-dev-app'
```

### Add 'data' subnet and associate corresponding network security group

```bash
    az network vnet subnet create \
        -n data \
        -g 'usw2-appx-net-01' \
        --vnet-name 'usw2-appx-dev-01' \
        --address-prefix 172.17.30.0/24 \
        --network-security-group 'usw2-appx-dev-data'
```

```bash
    az network vnet subnet create \
        -n data \
        -g 'usw2-appx-net-01' \
        --vnet-name 'usw2-appx-dev-02' \
        --address-prefix 172.18.30.0/24 \
        --network-security-group 'usw2-appx-dev-data'
```

### Add 'gateway' subnet and associate corresponding network security group

```bash
    az network vnet subnet create \
        -n gatewaysubnet \
        -g 'usw2-appx-net-01' \
        --vnet-name 'usw2-appx-dev-01' \
        --address-prefix 172.17.0.0/24
```

```bash
    az network vnet subnet create \
        -n gatewaysubnet \
        -g 'usw2-appx-net-01' \
        --vnet-name 'usw2-appx-dev-02' \
        --address-prefix 172.18.0.0/24
```

### Public IP Address for the VPN gateway

```bash
    az network public-ip create \
        -g 'usw2-appx-net-01' \
        -n 'usw2-appx-dev-vpn-01' \
        --dns-name 'usw2-appx-vpn-01' \
        --allocation-method Dynamic
```

```bash
    az network public-ip create \
        -g 'usw2-appx-net-01' \
        -n 'usw2-appx-dev-vpn-02' \
        --dns-name 'usw2-appx-vpn-02' \
        --allocation-method Dynamic
```

### Create the VPN Gateway

```bash
    az network vnet-gateway create \
        -n 'usw2-appx-dev-vpn-01' \
        -g 'usw2-appx-net-01' \
        --address-prefixes 192.168.17.0/24 \
        --vnet 'usw2-appx-dev-01' \
        --public-ip-address 'usw2-appx-dev-vpn-01' \
        --client-protocol SSTP \
        --gateway-type VPN \
        --sku VpnGw1
```

```bash
    az network vnet-gateway create \
        -n 'usw2-appx-dev-vpn-gw-02' \
        -g 'usw2-appx-net-01' \
        --address-prefixes 192.168.18.0/24 \
        --vnet 'usw2-appx-dev-02' \
        --public-ip-address 'usw2-appx-dev-vpn-02' \
        --client-protocol SSTP \
        --gateway-type VPN \
        --sku VpnGw1
```

### Load Balancer

https://docs.microsoft.com/en-us/azure/load-balancer/load-balancer-overview

https://docs.microsoft.com/en-us/azure/load-balancer/load-balancer-outbound-connections
