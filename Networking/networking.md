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

http://jmespath.org/

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
    az group create --name 'rg1' --location westus2
```

## Storage

### Verify storage account name availability

```bash
    az storage account check-name --name 'logsusw2dev01'
```

### Create a storage account to hold log files and diagnostics data

```bash
    az storage account create \
        --name 'logsusw2dev01' \
        --resource-group 'rg1' \
        --location westus2 \
        --sku Standard_LRS \
        --kind StorageV2
```

### Enable firewall

```bash
    az storage account update \
        --name 'logsusw2appxdev01' \
        --resource-group 'rg1' \
        --default-action Deny \
        --bypass Logging Metrics AzureServices
```

### Disable firewall

```bash
    az storage account update \
        --name 'logsusw2appxdev01' \
        --resource-group 'rg1' \
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
        --name 'usw2-dev-01' \
        --resource-group 'rg1' \
        --address-prefix 172.16.0.0/16 \
        --subnet-name web \
        --subnet-prefix 172.16.10.0/24
```

### Create a network security group (nsg) for each subnet

```bash
    az network nsg create --resource-group 'rg1' --name 'usw2-dev-01-web'
```

### Associate the 'web' network security group (nsg) with the corresponding subnet

```bash
    az network vnet subnet update \
        --name 'web' \
        --resource-group 'rg1' \
        --vnet-name 'usw2-dev-01' \
        --network-security-group 'usw2-dev-01-web'
```

### Add 'app' subnet and associate corresponding network security group

```bash
    az network vnet subnet create \
        --name 'app' \
        --resource-group 'rg1' \
        --vnet-name 'usw2-dev-01' \
        --address-prefix 172.16.20.0/24 \
        --network-security-group 'usw2-dev-01-app'
```

### Add 'data' subnet and associate corresponding network security group

```bash
    az network vnet subnet create \
        --name 'data' \
        --resource-group 'rg1' \
        --vnet-name 'usw2-dev-01' \
        --address-prefix 172.16.30.0/24 \
        --network-security-group 'usw2-dev-01-data'
```

### Add 'gateway' subnet and associate corresponding network security group

```bash
    az network vnet subnet create \
        --name 'gatewaysubnet' \
        --resource-group 'rg1' \
        --vnet-name 'usw2-dev-01' \
        --address-prefix 172.16.0.0/24
```

### Public IP Address for the VPN gateway

```bash
    az network public-ip create \
        --name 'usw2-dev-01-vpn' \
        --resource-group 'rg1' \
        --dns-name 'usw2-dev-01-vpn' \
        --allocation-method Dynamic \
        --sku standard
```

### Create the VPN gateway

```bash
    az network vnet-gateway create \
        --name 'usw2-dev-01-vpn' \
        --resource-group 'rg1' \
        --address-prefixes 192.168.16.0/24 \
        --vnet 'usw2-dev-01' \
        --public-ip-address 'usw2-dev-01-vpn' \
        --client-protocol SSTP \
        --gateway-type VPN \
        --sku VpnGw1
```

### Private DNS zone

https://docs.microsoft.com/en-us/azure/dns/private-dns-getstarted-cli

```bash
    az extension add --name dns

    az network dns zone create \
        -g rg1 \
        -n dev.local \
        --zone-type Private \
        --resolution-vnets usw2-dev-02
        --registration-vnets usw2-dev-02


    az network dns zone create \
        -g rg1 \
        -n mtc.local \
        --zone-type Private \
        --registration-vnets usw2-dev-01
```

### Load Balancer

https://docs.microsoft.com/en-us/azure/load-balancer/load-balancer-overview

https://docs.microsoft.com/en-us/azure/load-balancer/load-balancer-outbound-connections

```bash
    az network public-ip create \
        --resource-group rg1 \
        --name 'usw2-web-01-ip' \
        --sku standard

    az network public-ip create \
        --resource-group rg1 \
        --name 'usw2-web-02-ip' \
        --sku standard

    az network public-ip create \
        --resource-group rg1 \
        --name 'usw2-dev-01-elb' \
        --sku standard

    az network lb create \
        --resource-group 'rg1' \
        --name 'usw2-dev-01-elb' \
        --frontend-ip-name 'usw2-dev-01-fe' \
        --backend-pool-name 'usw2-dev-01-be' \
        --public-ip-address 'usw2-dev-01-elb' \
        --sku standard

    az network lb probe create \
        --resource-group 'rg1' \
        --lb-name 'usw2-dev-01-elb' \
        --name 'usw2-dev-01-probe' \
        --protocol tcp \
        --port 80

    az network lb rule create \
        --resource-group 'rg1' \
        --lb-name 'usw2-dev-01-elb' \
        --name 'usw2-dev-01-rule' \
        --protocol tcp \
        --frontend-port 80 \
        --backend-port 80 \
        --frontend-ip-name 'usw2-dev-01-fe' \
        --backend-pool-name 'usw2-dev-01-be' \
        --probe-name 'usw2-dev-01-probe'

    for i in `seq 1 2`; do
        az network nic ip-config address-pool add \
            --lb-name 'usw2-dev-01-elb' \
            --address-pool 'usw2-dev-01-be' \
            --nic-name usw2-web-0$i-nic1 \
            --ip-config-name 'ipconfig1' \
            --resource-group 'rg1'
    done

    az network nic ip-config address-pool add \
        --lb-name 'usw2-dev-01-elb' \
        --address-pool 'usw2-dev-01-be' \
        --nic-name 'usw2-web-01-nic1' \
        --ip-config-name 'ipconfig1' \
        --resource-group 'rg1'

    az network nic ip-config address-pool add \
        --lb-name 'usw2-dev-01-elb' \
        --address-pool 'usw2-dev-01-be' \
        --nic-name 'usw2-web-02-nic1' \
        --ip-config-name 'ipconfig1' \
        --resource-group 'rg1'

    az network lb address-pool show \
        --resource-group 'rg1' \
        --lb-name 'usw2-dev-01-elb' \
        --name 'usw2-dev-01-be' \
        --query backendIpConfigurations \
        --output tsv | cut -f4
```