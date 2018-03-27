# Virtual Machine

https://docs.microsoft.com/en-us/cli/azure/vm?view=azure-cli-latest

## Create a Ubuntu virtual machine from the marketpalce

```bash
    az vm image list --all

    az vm create -n 'usw2-web-01' -g 'usw2-appx-dev-01' --image UbuntuLTS --generate-ssh-keys

    az vm create -n 'usw2-web-02' -g 'usw2-appx-dev-01' --image UbuntuLTS

    az vm create -n MyVm -g MyResourceGroup --public-ip-address "" --image Win2012R2Datacenter
```

## Scale Set