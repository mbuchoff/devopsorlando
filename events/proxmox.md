# Proxmox

[VM serial access](https://chatgpt.com/share/6865228d-3c68-8005-b2ca-d8d74f4d0580)

# TODO: WHAT IS PROXMOX?

## [YouTube Tutorial](https://www.youtube.com/watch?v=5j0Zb6x_hOk&list=PLT98CRl2KxKHnlbYhtABg6cF50bYa8Ulo)
## [Documentation](https://pve.proxmox.com/pve-docs/pve-admin-guide.html)
- Login
  - Username: `root`
  - Password: `password`

## [How to get an Azure VM with ProxMox](https://chatgpt.com/share/68627b5b-7078-8005-9a54-516293f966d1)
  ```
  az storage account create -n saproxmox -g rg-proxmox -l eastus2 --sku Standard_LRS
  az storage container create --account-name saproxmox --name vhds
  ```

  ```
  $osDiskName = "disk-proxmox2"  # Name of your existing disk

  az storage blob upload --account-name saproxmox --container-name vhds --file "C:\Users\mbuch\OneDrive\Desktop\proxmox\proxmox-azure.vhd" --name proxmox.vhd --overwrite
  az disk create --resource-group rg-proxmox --name $osDiskName --source https://saproxmox.blob.core.windows.net/vhds/proxmox.vhd --os-type Linux

  $rg = "rg-proxmox"
  $location = "eastus"
  $vnetName = "proxmox-vnet"
  $subnetName = "proxmox-subnet"
  $nicName = "proxmox-nic2"
  $publicIpName = "proxmox-ip2"

  az network public-ip create --resource-group $rg --name $publicIpName --allocation-method Static

  az network vnet create --resource-group $rg --name $vnetName --address-prefix 10.0.0.0/16 `
  --subnet-name $subnetName --subnet-prefix 10.0.0.0/24

  $nsgName = "proxmox-wideopen-nsg"
  az network nsg create --resource-group $rg --name $nsgName

  # TODO: MORE SECURE
  az network nsg rule create --resource-group $rg --nsg-name $nsgName --name "AllowAllIn" `
  --priority 100 --direction Inbound --access Allow --protocol '*' --source-port-range '*' `
  --destination-port-range '*' --source-address-prefix '*' --destination-address-prefix '*'

  az network vnet subnet update --resource-group $rg --vnet-name $vnetName `
  --name $subnetName --network-security-group $nsgName

  az network nic create `
  --resource-group $rg `
  --name $nicName `
  --vnet-name $vnetName `
  --subnet $subnetName `
  --network-security-group $nsgName `
  --public-ip-address $publicIpName

  $vmName = "vm-proxmox2"
  $size = "Standard_DS1_v2"     # Or whatever you want

  az vm create `
  --resource-group $rg `
  --name $vmName `
  --nics $nicName `
  --attach-os-disk $osDiskName `
  --os-type Linux `
  --size $size

  $ip = az network public-ip show --resource-group $rg --name $publicIpName --query "ipAddress" -o tsv
  Write-Output "Public IP is: $ip"

  az vm boot-diagnostics enable `
  --resource-group $rg `
  --name $vmName `
  --storage saproxmox

  # Wait a bit after starting the VM, then fetch the log
  az vm boot-diagnostics get-boot-log --resource-group $rg --name $vmName

  # Changed NSG of proxmox-nic
  ```

- Chrome Reader Mode extension somehow interferes with web proxy
- You can ssh into your proxmox
- First thing, Updates > Refresh, _Upgrade
  - Ignore errors
  - Turn off Proxmox repositories by going into Updates > Repositories and disabling anything with `enterprise.proxmox`
  - Which package manager is this using? Interface suggestes apt
  - Adding a Test repository means being an early adopter
  - You will see your action in the Task History section
- Disks
  - Although the LVM section might suggest there's no free space, the used space is assigned to LVM, which you can see in the LVM Thin section
- VMs migrate between Proxmox servers on a cluster quickly and stay live. Containers are the opposite.
- Predownload:
  - Proxmox ISO
  - Ubuntu server ISO
- Consider mimicing [Launching a VM](https://www.youtube.com/watch?v=xBUnV2rQ7do&t=1177s)
  - mbuchoff / password
  - Your VM will need virtualization enabled. Could be a hurdle. In HyperV: `Set-VMProcessor -VMName "YourVMName" -ExposeVirtualizationExtensions $true`
- networking for Ubuntu Server
  - Proxmox ip: 172.24.201.156
  - Subnet: 172.24.192.0/20
  - Address: 172.24.201.200
  - Gateway: 172.24.192.1
  - Name servers: 8.8.8.8, 1.1.1.1
