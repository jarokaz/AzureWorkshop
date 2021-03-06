# Virtual Machine

Overview: https://docs.microsoft.com/en-us/azure/virtual-machines/linux/

## Create a Ubuntu virtual machine from the marketpalce

### List available images and virtual machine sizes

```bash
    az vm image list --output table
```

### List available images and virtual machine sizes for a specific region

```bash
    az vm list-sizes --location westus2 --output table
```

### Create a basic virtual machine

The following command will create a virtual machine, with a public IP address

```bash
    az vm create \
        --name 'usw2-web-01' \
        --resource-group '<resource group name>' \
        --image UbuntuLTS \
        --generate-ssh-keys
```

## Create and configure a Ubuntu virtual machine from the marketpalce

Overview: https://docs.microsoft.com/en-us/azure/virtual-machines/linux/tutorial-automate-vm-deployment

### Create a cloud-config file

How to customize a Linux virtual machine on first boot with cloud init

Create a VM that installs NGINX and runs a simple 'Hello World' Node.js app. The following cloud-init configuration installs the required packages, creates a Node.js app, then initialize and starts the app.

In your current shell, create a file named cloud-init.txt and paste the following configuration. For example, create the file in the Cloud Shell not on your local machine. You can use any editor you wish. Enter sensible-editor cloud-init.txt to create the file and see a list of available editors. Make sure that the whole cloud-init file is copied correctly, especially the first line:

```yaml
    #cloud-config
    package_upgrade: true
    packages:
    - nginx
    - nodejs
    - npm
    write_files:
    - owner: www-data:www-data
    - path: /etc/nginx/sites-available/default
        content: |
        server {
            listen 80;
            location / {
            proxy_pass http://localhost:3000;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection keep-alive;
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
            }
        }
    - owner: azureuser:azureuser
    - path: /home/azureuser/myapp/index.js
        content: |
        var express = require('express')
        var app = express()
        var os = require('os');
        app.get('/', function (req, res) {
            res.send('Hello World from host ' + os.hostname() + '!')
        })
        app.listen(3000, function () {
            console.log('Hello world app listening on port 3000!')
        })
    runcmd:
    - service nginx restart
    - cd "/home/azureuser/myapp"
    - npm init
    - npm install express -y
    - nodejs index.js
```

### Create application server availability set

```bash
    az vm availability-set create \
        --resource-group '<resource group name>' \
        --name 'usw2-app-set-01' \
        --platform-fault-domain-count 2 \
        --platform-update-domain-count 5
```

```bash
    for i in `seq -w 01 02`; do
        az network nic create \
            --resource-group rg1 \
            --vnet-name 'usw2-dev-01' \
            --subnet 'app' \
            --name usw2-app-$i-nic1
    done
```

```bash
    for i in `seq -w 01 02`; do
        az vm create \
            --name usw2-app-$i \
            --resource-group '<resource group name>' \
            --nics usw2-app-$i-nic1 \
            --image UbuntuLTS \
            --size Standard_F2s \
            --availability-set 'usw2-app-set-01' \
            --public-ip-address "" \
            --nsg '' \
            --custom-data cloud-init.txt
    done
```

### Create web server availability Set

```bash
    az vm availability-set create \
        --resource-group '<resource group name>' \
        --name 'usw2-web-set-01' \
        --platform-fault-domain-count 2 \
        --platform-update-domain-count 5
```

```bash
    for i in `seq -w 01 02`; do
        az network nic create \
            --resource-group rg1 \
            --vnet-name 'usw2-dev-01' \
            --subnet 'web' \
            --name usw2-web-$i-nic1
    done
```

```bash
    for i in `seq -w 01 02`; do
        az vm create \
            --name usw2-web-$i \
            --resource-group '<resource group name>' \
            --nics usw2-web-$i-nic1 \
            --image UbuntuLTS \
            --size Standard_F2s \
            --availability-set 'usw2-web-set-01' \
            --public-ip-address "" \
            --nsg '' \
            --custom-data cloud-init.txt
    done
```

### create database server Availability Set

```bash
    az vm availability-set create \
        --resource-group '<resource group name>' \
        --name 'usw2-data-set-01' \
        --platform-fault-domain-count 2 \
        --platform-update-domain-count 2
```

```bash
    for i in `seq -w 01 02`; do
        az network nic create \
            --resource-group rg1 \
            --vnet-name 'usw2-dev-01' \
            --subnet 'data' \
            --name usw2-data-$i-nic1
    done
```

```bash
    for i in `seq -w 01 02`; do
        az vm create \
            --name usw2-data-$i \
            --resource-group '<resource group name>' \
            --nics usw2-data-$i-nic1 \
            --image UbuntuLTS \
            --size Standard_F2s \
            --availability-set 'usw2-data-set-01' \
            --public-ip-address "" \
            --nsg '' \
            --custom-data cloud-init.txt
    done
```

### Resize the virtual machine

```bash
    az vm list-vm-resize-options \
        --resource-group '<resource group name>' \
        --name 'usw2-web-01' \
        --query [].name
```

```bash
    az vm resize \
        --resource-group '<resource group name>' \
        --name 'usw2-web-01' \
        --size Standard_DS4_v2
```

## Virtual machine Scale Set

https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/quick-create-cli

### Create a new resource group

```bash
    az group create --name 'rg2' --location westus2
```

### Create the virtual machine scale set

```bash
    az vmss create \
        --resource-group 'rg2' \
        --name 'usw2-web-set-01' \
        --image UbuntuLTS \
        --custom-data cloud-init.txt \
        --upgrade-policy-mode automatic
```

### Create a virtual machine scale set extension

```bash
    az vmss extension set \
        --publisher Microsoft.Azure.Extensions \
        --version 2.0 \
        --name CustomScript \
        --resource-group 'rg2' \
        --vmss-name 'usw2-web-set-01' \
        --settings '{"fileUris":["https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/automate_nginx.sh"],"commandToExecute":"./automate_nginx.sh"}'
```

### Create a virtual machine scale load balancer probe

```bash
    az network lb probe create \
        --resource-group 'rg2' \
        --lb-name 'usw2-web-set-01LB' \
        --name 'usw2-web-set-01-probe' \
        --protocol tcp \
        --port 80
```

### Create a virtual machine scale load balancer rule

```bash
    az network lb rule create \
        --resource-group 'rg2' \
        --name 'usw2-web-set-01-rule' \
        --lb-name 'usw2-web-set-01LB' \
        --backend-pool-name 'usw2-web-set-01LBBEPool' \
        --backend-port 80 \
        --frontend-ip-name 'loadBalancerFrontEnd' \
        --frontend-port 80 \
        --protocol tcp \
        --probe-name 'usw2-web-set-01-probe'
```

## Custom Virtual Machine Images

https://docs.microsoft.com/en-us/azure/virtual-machines/linux/create-upload-generic

https://docs.microsoft.com/en-us/azure/virtual-machines/linux/create-upload-centos

https://docs.microsoft.com/en-us/azure/virtual-machines/linux/create-upload-ubuntu

https://docs.microsoft.com/en-us/azure/virtual-machines/linux/upload-vhd

### Install the Azure Linux Agent

```bash
    sudo yum list WALinuxAgent

    sudo yum install WALinuxAgent
```

### Create a storage account for the VHD image file

```bash
    az storage account create \
        --name 'imagesusw2dev01' \
        --resource-group '<resource group name>' \
        --location westus2 \
        --sku Standard_LRS \
        --kind StorageV2
```

### Upload the custom VHD image file

```bash
    az disk create \
        --resource-group '<resource group name>' \
        --name centos \
        --source https://imagesusw2dev01.blob.core.windows.net/disk2/centos.vhd
```

### Create a virtual machine from the custom image

```bash
    az vm create \
        --name usw2-app-08 \
        --resource-group '<resource group name>' \
        --nics 'usw2-app-08-nic1' \
        --os-type linux \
        --attach-os-disk centos \
        --size Standard_F2s \
        --public-ip-address "" \
        --nsg '' \
        --custom-data cloud-init.txt
```

### Metadata Tags

```bash
    az resource tag -n '<resource group name>' \
        -g myResourceGroup \
        --tags Dept=IT Environment=Test Project=Documentation \
        --resource-type "Microsoft.Compute/virtualMachines"

    az resource list --tag Environment=Test --query [].name
```