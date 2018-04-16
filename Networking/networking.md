# Networking

## Networks

Azure network topology reference and guidance: https://docs.microsoft.com/en-us/azure/networking/networking-virtual-datacenter

### Show Existing Networks

```bash
    az network vnet list --out table
```

### Create a Virtual Network (vnet) with a subnet called 'web'

```bash
    az network vnet create \
        --name 'usw2-dev-01' \
        --resource-group '<resource group name>' \
        --address-prefix 172.16.0.0/16 \
        --subnet-name web \
        --subnet-prefix 172.16.10.0/24
```

#### Show the virtual networks

```bash
    az network vnet list --out table
```

### Create a network security group (nsg) for each subnet

```bash
    az network nsg create --resource-group '<resource group name>' --name 'usw2-dev-01-web'
    az network nsg create --resource-group '<resource group name>' --name 'usw2-dev-01-app'
    az network nsg create --resource-group '<resource group name>' --name 'usw2-dev-01-data'
```

#### Show the network security groups

```bash
    az network vnet list --out table
```

### Associate the 'web' network security group (nsg) with the corresponding subnet

```bash
    az network vnet subnet update \
        --name 'web' \
        --resource-group '<resource group name>' \
        --vnet-name 'usw2-dev-01' \
        --network-security-group 'usw2-dev-01-web'
```

### Add 'app' subnet and associate corresponding network security group

```bash
    az network vnet subnet create \
        --name 'app' \
        --resource-group '<resource group name>' \
        --vnet-name 'usw2-dev-01' \
        --address-prefix 172.16.20.0/24 \
        --network-security-group 'usw2-dev-01-app'
```

### Add 'data' subnet and associate corresponding network security group

```bash
    az network vnet subnet create \
        --name 'data' \
        --resource-group '<resource group name>' \
        --vnet-name 'usw2-dev-01' \
        --address-prefix 172.16.30.0/24 \
        --network-security-group 'usw2-dev-01-data'
```

### Add 'gateway' subnet and associate corresponding network security group

```bash
    az network vnet subnet create \
        --name 'gatewaysubnet' \
        --resource-group '<resource group name>' \
        --vnet-name 'usw2-dev-01' \
        --address-prefix 172.16.0.0/24
```

### Public IP Address for the VPN gateway

**NOTE**: You will need to provide a globally unique -dns-name parameter.

```bash
    az network public-ip create \
        --name 'usw2-dev-01-vpn' \
        --resource-group '<resource group name>' \
        --dns-name 'usw2-dev-01-vpn' \
        --allocation-method Static \
        --sku standard
```

### Create the VPN gateway

```bash
    az network vnet-gateway create \
        --name 'usw2-dev-01-vpn' \
        --resource-group '<resource group name>' \
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
        --registration-vnets usw2-dev-02
```

```bash
    az network dns zone create \
        -g rg1 \
        -n dev.local \
        --zone-type Private \
        --registration-vnets usw2-dev-02
```

### External load balancer

https://docs.microsoft.com/en-us/azure/load-balancer/load-balancer-overview

https://docs.microsoft.com/en-us/azure/load-balancer/load-balancer-outbound-connections

#### External load balancer public IP address

```bash
    az network public-ip create \
        --resource-group rg1 \
        --name 'usw2-dev-01-elb3' \
        --sku standard
```

#### Create the external load balancer

```bash
    az network lb create \
        --resource-group '<resource group name>' \
        --name 'usw2-dev-01-elb' \
        --frontend-ip-name 'usw2-dev-01-elb-fe' \
        --backend-pool-name 'usw2-dev-01-elb-be' \
        --public-ip-address 'usw2-dev-01-elb3' \
        --sku standard
```

#### Create the external load balancer health probe

```bash
    az network lb probe create \
        --resource-group '<resource group name>' \
        --lb-name 'usw2-dev-01-elb' \
        --name 'usw2-dev-01-elb-probe' \
        --protocol tcp \
        --port 80
```

#### Create the external load balancing rule

```bash
    az network lb rule create \
        --resource-group '<resource group name>' \
        --lb-name 'usw2-dev-01-elb' \
        --name 'usw2-dev-01-elb-rule' \
        --protocol tcp \
        --frontend-port 80 \
        --backend-port 80 \
        --frontend-ip-name 'usw2-dev-01-elb-fe' \
        --backend-pool-name 'usw2-dev-01-elb-be' \
        --probe-name 'usw2-dev-01-elb-probe'
```

#### Add the network interface to the external load balancer back-end pool

```bash
    az network nic ip-config address-pool add \
        --lb-name 'usw2-dev-01-elb' \
        --address-pool 'usw2-dev-01-elb-be' \
        --nic-name 'usw2-web-01-nic1' \
        --ip-config-name 'ipconfig1' \
        --resource-group '<resource group name>'

    az network nic ip-config address-pool add \
        --lb-name 'usw2-dev-01-elb' \
        --address-pool 'usw2-dev-01-elb-be' \
        --nic-name 'usw2-web-02-nic1' \
        --ip-config-name 'ipconfig1' \
        --resource-group '<resource group name>'
```

#### Show the load balancer address pool

```bash
    az network lb address-pool show \
        --resource-group '<resource group name>' \
        --lb-name 'usw2-dev-01-elb' \
        --name 'usw2-dev-01-elb-be' \
        --query backendIpConfigurations \
        --output tsv | cut -f4
```

#### Remove the network interface to the external load balancer back-end pool

```bash
    az network nic ip-config address-pool remove \
        --lb-name 'usw2-dev-01-elb' \
        --address-pool 'usw2-dev-01-elb-be' \
        --nic-name 'usw2-web-01-nic1' \
        --ip-config-name 'ipconfig1' \
        --resource-group '<resource group name>'

    az network nic ip-config address-pool remove \
        --lb-name 'usw2-dev-01-elb' \
        --address-pool 'usw2-dev-01-elb-be' \
        --nic-name 'usw2-web-02-nic1' \
        --ip-config-name 'ipconfig1' \
        --resource-group '<resource group name>'
```

### Internal Load Balancer

https://docs.microsoft.com/en-us/azure/load-balancer/load-balancer-overview

https://docs.microsoft.com/en-us/azure/load-balancer/load-balancer-outbound-connections

#### Create the external load balancer

```bash
    az network lb create \
        --resource-group '<resource group name>' \
        --name 'usw2-dev-01-ilb' \
        --frontend-ip-name 'usw2-dev-01-ilb-fe' \
        --private-ip-address 172.16.20.100 \
        --backend-pool-name 'usw2-dev-01-ilb-be' \
        --vnet-name 'usw2-dev-01' \
        --subnet 'app' \
        --sku standard
```

``` bash
    az network lb probe create \
        --resource-group '<resource group name>' \
        --lb-name 'usw2-dev-01-ilb' \
        --name 'usw2-dev-01-ilb-probe' \
        --protocol tcp \
        --port 80
```

``` bash
    az network lb rule create \
        --resource-group '<resource group name>' \
        --lb-name 'usw2-dev-01-ilb' \
        --name 'usw2-dev-01-ilb-rule' \
        --protocol tcp \
        --frontend-port 80 \
        --backend-port 80 \
        --frontend-ip-name 'usw2-dev-01-ilb-fe' \
        --backend-pool-name 'usw2-dev-01-ilb-be' \
        --probe-name 'usw2-dev-01-ilb-probe'
```

### Add network interface to the ilb back-end pool

```bash
    az network nic ip-config address-pool add \
        --lb-name 'usw2-dev-01-ilb' \
        --address-pool 'usw2-dev-01-ilb-be' \
        --nic-name 'usw2-app-01-nic1' \
        --ip-config-name 'ipconfig1' \
        --resource-group '<resource group name>'

    az network nic ip-config address-pool add \
        --lb-name 'usw2-dev-01-ilb' \
        --address-pool 'usw2-dev-01-ilb-be' \
        --nic-name 'usw2-app-02-nic1' \
        --ip-config-name 'ipconfig1' \
        --resource-group '<resource group name>'
```

## Application Gateway

### Add 'data' subnet and associate corresponding network security group

```bash
    az network vnet subnet create \
        --name 'appgw' \
        --resource-group '<resource group name>' \
        --vnet-name 'usw2-dev-01' \
        --address-prefix 172.16.1.0/24 \
        --network-security-group ''

    az network public-ip create \
        --resource-group rg1 \
        --name 'usw2-dev-01-gw2' \
        --sku basic

    az network application-gateway create \
        --name 'usw2-dev-01-gw' \
        --location westus2 \
        --resource-group '<resource group name>' \
        --capacity 2 \
        --sku WAF_Medium \
        --http-settings-cookie-based-affinity Enabled \
        --public-ip-address 'usw2-dev-01-gw2' \
        --vnet-name 'usw2-dev-01' \
        --subnet 'appgw' \
        --servers "172.16.30.4" "172.16.30.5"
```