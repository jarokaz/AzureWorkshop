# Networks

https://docs.microsoft.com/en-us/azure/networking/networking-virtual-datacenter


## Show Existing Networks

```bash
    az network vnet list
```

```bash
    az network vnet list --out table
```

## Create a Virtual Network (vnet) with a subnet called 'web'

```bash
    az network vnet create \
        -n 'usw-appx-dev-01' \
        -g 'usw-appx-net-01' \
        --address-prefix 172.17.0.0/16 \
        --subnet-name web \
        --subnet-prefix 172.17.10.0/24
```

```bash
    az network vnet create \
        -n 'usw-appx-dev-02' \
        -g 'usw-appx-net-01' \
        --address-prefix 172.18.0.0/16 \
        --subnet-name web \
        --subnet-prefix 172.18.10.0/24
```

## Create a network security group (nsg) for each subnet

```bash
    az network nsg create -g usw-appx-net-01 -n 'usw-appx-dev-web'
    az network nsg create -g usw-appx-net-01 -n 'usw-appx-dev-app'
    az network nsg create -g usw-appx-net-01 -n 'usw-appx-dev-data'
```

## Associate the 'web' network security group (nsg) with the corresponding subnet

```bash
    az network vnet subnet update \
        -n web \
        -g 'usw-appx-net-01' \
        --vnet-name 'usw-appx-dev-01' \
        --network-security-group 'usw-appx-dev-web'
```

```bash
    az network vnet subnet update \
        -n web \
        -g 'usw-appx-net-01' \
        --vnet-name 'usw-appx-dev-02' \
        --network-security-group 'usw-appx-dev-web'
```

## Add 'app' subnet and associate corresponding network security group

```bash
    az network vnet subnet create \
        -n app \
        -g 'usw-appx-net-01' \
        --vnet-name 'usw-appx-dev-01' \
        --address-prefix 172.17.20.0/24 \
        --network-security-group 'usw-appx-dev-app'
```

```bash
    az network vnet subnet create \
        -n app \
        -g 'usw-appx-net-01' \
        --vnet-name 'usw-appx-dev-02' \
        --address-prefix 172.18.20.0/24 \
        --network-security-group 'usw-appx-dev-app'
```

## Add 'data' subnet and associate corresponding network security group

```bash
    az network vnet subnet create \
        -n data \
        -g 'usw-appx-net-01' \
        --vnet-name 'usw-appx-dev-01' \
        --address-prefix 172.17.30.0/24 \
        --network-security-group 'usw-appx-dev-data'
```

```bash
    az network vnet subnet create \
        -n data \
        -g 'usw-appx-net-01' \
        --vnet-name 'usw-appx-dev-02' \
        --address-prefix 172.18.30.0/24 \
        --network-security-group 'usw-appx-dev-data'
```

## Add 'gateway' subnet and associate corresponding network security group

```bash
    az network vnet subnet create \
        -n gatewaysubnet \
        -g 'usw-appx-net-01' \
        --vnet-name 'usw-appx-dev-01' \
        --address-prefix 172.17.0.0/24
```

```bash
    az network vnet subnet create \
        -n gatewaysubnet \
        -g 'usw-appx-net-01' \
        --vnet-name 'usw-appx-dev-02' \
        --address-prefix 172.18.0.0/24
```

## Public IP Address for the VPN gateway

```bash
    az network public-ip create \
        -g 'usw-appx-net-01' \
        -n 'usw-appx-dev-vpn-ip-01' \
        --dns-name 'usw-appx-vpn-01' \
        --allocation-method Dynamic
```

```bash
    az network public-ip create \
        -g 'usw-appx-net-01' \
        -n 'usw-appx-dev-vpn-ip-02' \
        --dns-name 'usw-appx-vpn-02' \
        --allocation-method Dynamic
```

## Create the VPN Gateway

```bash
    az network vnet-gateway create \
        -n 'usw-appx-dev-vpn-gw-01' \
        -g 'usw-appx-net-01' \
        --address-prefixes 192.168.17.0/24 \
        --vnet 'usw-appx-dev-01' \
        --public-ip-address 'usw-appx-dev-vpn-ip-01' \
        --client-protocol SSTP \
        --gateway-type VPN \
        --sku VpnGw1
```

```bash
    az network vnet-gateway create \
        -n 'usw-appx-dev-vpn-gw-02' \
        -g 'usw-appx-net-01' \
        --address-prefixes 192.168.17.0/24 \
        --vnet 'usw-appx-dev-02' \
        --public-ip-address 'usw-appx-dev-vpn-ip-02' \
        --client-protocol SSTP \
        --gateway-type VPN \
        --sku VpnGw1
```

## Load Balancer

https://docs.microsoft.com/en-us/azure/load-balancer/load-balancer-overview

https://docs.microsoft.com/en-us/azure/load-balancer/load-balancer-outbound-connections
