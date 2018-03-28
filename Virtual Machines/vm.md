# Virtual Machine

https://docs.microsoft.com/en-us/cli/azure/vm?view=azure-cli-latest

https://docs.microsoft.com/en-us/azure/virtual-machines/linux/

## Create a Ubuntu virtual machine from the marketpalce

### List images and sizes

```bash
    az vm image list --output table

    az vm list-sizes --location westus2 --output table
```

```bash
    az vm create \
        --name 'usw2-web-01' \
        --resource-group 'rg1' \
        --image UbuntuLTS \
        --generate-ssh-keys
```

```bash
    sudo apt-get -y install nginx
```

### cloud-config

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

### Availability Set

```bash
    az vm availability-set create \
        --resource-group 'rg1' \
        --name 'usw2-web-set-01' \
        --platform-fault-domain-count 2 \
        --platform-update-domain-count 2
```

```bash
    az vm create \
        --name 'usw2-web-01' \
        --resource-group 'rg1' \
        --image UbuntuLTS \
        --size Standard_F4s \
        --vnet-name 'usw2-dev-01' \
        --subnet 'web' \
        --availability-set 'usw2-web-set-01' \
        --nsg '' \
        --custom-data cloud-init.txt
```

### Use an existing network interface

```bash
    for i in `seq 1 5`; do
        az network nic create \
            --resource-group rg1 \
            --vnet-name 'usw2-dev-01' \
            --subnet 'web' \
            --name usw2-web-0$i-nic1
    done
```

```bash
    for i in `seq 1 2`; do
        az vm create \
            --name usw2-web-0$i \
            --resource-group 'rg1' \
            --nics usw2-web-0$i-nic1 \
            --image UbuntuLTS \
            --size Standard_F4s \
            --availability-set 'usw2-web-set-01' \
            --public-ip-address "" \
            --nsg '' \
            --custom-data cloud-init.txt
    done
```

```bash
    az vm open-port --port 80 --resource-group rg1 --name 'usw2-web-01'
```

### Resize the virtual machine

```bash
    az vm list-vm-resize-options --resource-group rg1 --name usw2-web-01 --query [].name
```

```bash
    az vm resize --resource-group myResourceGroupVM --name myVM --size Standard_DS4_v2
```

## Virtual machine Scale Set

https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/quick-create-cli

```bash
    az group create --name 'rg2' --location westus2

    az vmss create \
        --resource-group 'rg2' \
        --name 'usw2-web-set-01' \
        --image UbuntuLTS \
        --custom-data cloud-init.txt \
        --upgrade-policy-mode automatic

    az vmss extension set \
        --publisher Microsoft.Azure.Extensions \
        --version 2.0 \
        --name CustomScript \
        --resource-group 'rg2' \
        --vmss-name 'usw2-web-set-01' \
        --settings '{"fileUris":["https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/automate_nginx.sh"],"commandToExecute":"./automate_nginx.sh"}'

    az network lb probe create \
        --resource-group 'rg2' \
        --lb-name 'usw2-web-set-01LB' \
        --name 'usw2-web-set-01-probe' \
        --protocol tcp \
        --port 80

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

## Custom Virtual Machine Image

https://docs.microsoft.com/en-us/azure/virtual-machines/linux/create-upload-generic

https://docs.microsoft.com/en-us/azure/virtual-machines/linux/create-upload-centos

https://docs.microsoft.com/en-us/azure/virtual-machines/linux/create-upload-ubuntu

https://docs.microsoft.com/en-us/azure/virtual-machines/linux/upload-vhd

```bash
    sudo yum list WALinuxAgent

    sudo yum install WALinuxAgent
```

```bash
    az storage account create \
        --name 'imagesusw2dev01' \
        --resource-group 'rg1' \
        --location westus2 \
        --sku Standard_LRS \
        --kind StorageV2
```

```bash
    az disk create \
        --resource-group myResourceGroup \
        --name myManagedDisk \
        --source https://mystorageaccount.blob.core.windows.net/mydisks/myDisk.vhd
```

## Tags

```bash
    az resource tag -n myVM \
    -g myResourceGroup \
    --tags Dept=IT Environment=Test Project=Documentation \
    --resource-type "Microsoft.Compute/virtualMachines"

    az resource list --tag Environment=Test --query [].name
```