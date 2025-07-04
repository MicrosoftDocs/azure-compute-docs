---
title: Overview of cloud-init support for Linux VMs in Azure
description: Overview of cloud-init capabilities to configure a VM at provisioning time in Azure.
author: vamckMS
ms.service: azure-virtual-machines
ms.subservice: extensions
ms.custom: linux-related-content
ms.collection: linux
ms.topic: how-to
ms.date: 03/20/2025
ms.author: vakavuru
# Customer intent: "As a cloud engineer, I want to utilize cloud-init to automate the configuration of Linux VMs during provisioning, so that I can streamline deployments and eliminate the need for manual setup processes."
---
# cloud-init support for virtual machines in Azure

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Flexible scale sets 

This article explains the support that exists for [cloud-init](https://cloudinit.readthedocs.io) to configure a virtual machine (VM) or Virtual Machine Scale Sets at provisioning time in Azure. These cloud-init configurations are run on first boot once the resources have been provisioned by Azure.  

VM Provisioning is the process where Azure passes down your VM Create parameter values, such as hostname, username, and password, and make them available to the VM as it boots up. A 'provisioning agent' will consume those values, configure the VM, and report back when completed. 

Azure supports two provisioning agents [cloud-init](https://cloudinit.readthedocs.io), and the [Azure Linux Agent (WALA)](../extensions/agent-linux.md).

## cloud-init overview
[cloud-init](https://cloudinit.readthedocs.io) is a widely used approach to customize a Linux VM as it boots for the first time. You can use cloud-init to install packages and write files, or to configure users and security. Because cloud-init is called during the initial boot process, there are no additional steps or required agents to apply your configuration.  For more information on how to properly format your `#cloud-config` files or other inputs, see the [cloud-init documentation site](https://cloudinit.readthedocs.io/en/latest/topics/format.html#cloud-config-data).  `#cloud-config` files are text files encoded in base64.

cloud-init also works across distributions. For example, you don't use **apt-get install** or **yum install** to install a package. Instead you can define a list of packages to install. cloud-init automatically uses the native package management tool for the distro you select.

We're actively working with our endorsed Linux distro partners in order to have cloud-init enabled images available in the Azure Marketplace. 
These images will make your cloud-init deployments and configurations work seamlessly with VMs and virtual machine scale sets. Initially we collaborate with the endorsed Linux distro partners and upstream to ensure cloud-init functions with the OS on Azure, then the packages are updated and made publicly available in the distro package repositories. 

There are two stages to making cloud-init available to the supported Linux distributions on Azure, package support, and then image support:
* 'cloud-init package support on Azure' documents, which cloud-init packages onwards are supported or in preview, so you can use these packages with the OS in a custom image.
* 'image cloud-init ready' documents if the image is already configured to use cloud-init.

### Canonical
| Publisher / Version| Offer | SKU | Version | image cloud-init ready | cloud-init package support on Azure|
|:--- |:--- |:--- |:--- |:--- |:--- |
|Canonical 24.04 |UbuntuServer |24.04-LTS |latest |yes | yes |
|Canonical 22.04 |UbuntuServer |22.04-LTS |latest |yes | yes |
|Canonical 20.04 |UbuntuServer |20.04-LTS |latest |yes | yes |


### RHEL
| Publisher / Version| Offer | SKU | Version | image cloud-init ready | cloud-init package support on Azure|
|:--- |:--- |:--- |:--- |:--- |:--- |
|RedHat 7 |RHEL |7.7, 7.8, 7_9 |latest |yes | yes |
|RedHat 8 |RHEL |8.1, 8.2, 8_3, 8_4 |latest |yes | yes |
|RedHat 9 |RHEL |9_0, 9_1 |latest |yes | yes |

* All other RedHat SKUs starting from RHEL 7 (version 7.7) and RHEL 8 (version 8.1) including both Gen1 and Gen2 images are provisioned using cloud-init. Cloud-init isn't supported on RHEL 6. 

### Oracle

 Publisher / Version| Offer | SKU | Version | image cloud-init ready | cloud-init package support on Azure|
|:--- |:--- |:--- |:--- |:--- |:--- |
|Oracle 7 |Oracle Linux |77, 78, ol79 |latest |yes | yes |
|Oracle 8 |Oracle Linux |81, ol82, ol83-lvm, ol84-lvm |latest |yes | yes |

* All other Oracle SKUs starting from Oracle 7 (version 7.7) and Oracle 8 (version 8.1) including both Gen1 and Gen2 images are provisioned using cloud-init.

### SUSE SLES

 Publisher / Version| Offer | SKU | Version | image cloud-init ready | cloud-init package support on Azure|
|:--- |:--- |:--- |:--- |:--- |:--- |
|SUSE 15 |SLES (SUSE Linux Enterprise Server) |sp1, sp2, sp3 |latest |yes | yes |
|SUSE 12 |SLES (SUSE Linux Enterprise Server) |sp5 |latest |yes | yes |

### Debian
| Publisher / Version | Offer | SKU | Version | image cloud-init ready | cloud-init package support on Azure|
|:--- |:--- |:--- |:--- |:--- |:--- |
| debian-10 |Debian |10-cloudinit-gen2 |Debian:debian-10:10-cloudinit-gen2:0.0.1015 |yes |yes
| debian-10 |Debian |10-cloudinit-gen2 |Debian:debian-10:10-cloudinit-gen2:0.0.991 |yes |yes
| debian-10 |Debian |10-cloudinit-gen2 |Debian:debian-10:10-cloudinit-gen2:0.0.999 |yes |yes

Currently, Azure Stack supports the provisioning of cloud-init enabled images.

## What is the difference between cloud-init and the Linux Agent (WALA)?
WALA is an Azure platform-specific agent used to provision and configure VMs, and handle [Azure extensions](../extensions/features-linux.md). 

We're enhancing the task of configuring VMs to use cloud-init instead of the Linux Agent in order to allow existing cloud-init customers to use their current cloud-init scripts, or new customers to take advantage of the rich cloud-init configuration functionality. If you have existing investments in cloud-init scripts for configuring Linux systems, there are **no additional settings required** to enable cloud-init process them. 

cloud-init can't process Azure extensions, so WALA is still required in the image to process extensions but needs to have its provisioning code disabled. For endorsed Linux distros images that are being converted to provision by cloud-init, they have WALA installed, and setup correctly.

When creating a VM, if you don't include the Azure CLI `--custom-data` switch at provisioning time, cloud-init or WALA takes the minimal VM provisioning parameters required to provision the VM and complete the deployment with the defaults.  If you reference the cloud-init configuration with the `--custom-data` switch, whatever is contained in your custom data will be available to cloud-init when the VM boots.

cloud-init configurations applied to VMs don't have time constraints and won't cause a deployment to fail by timing out. This isn't true for WALA, if you change the WALA defaults to process custom-data, it can't exceed the total VM provisioning time allowance of 40 minutes, if so, the VM Create will fail.

## cloud-init VM provisioning without a UDF driver  
Beginning with cloud-init 21.2, you can use cloud-init to provision a VM in Azure without a UDF driver. If a UDF driver isn't available in the image, cloud-init uses the metadata that's available in the Azure Instance Metadata Service to provision the VM. This option works only for SSH key and [user data](../user-data.md). To pass in a password or custom data to a VM during provisioning, you must use a UDF driver.

## Deploying a cloud-init enabled Virtual Machine
Deploying a cloud-init enabled virtual machine is as simple as referencing a cloud-init enabled distribution during deployment. Linux distribution maintainers have to choose to enable and integrate cloud-init into their base Azure published images. Once you confirm the image you want to deploy is cloud-init enabled, you can use the Azure CLI to deploy the image. 

The first step in deploying this image is to create a resource group with the [az group create](/cli/azure/group) command. An Azure resource group is a logical container into which Azure resources are deployed and managed. 

The following example creates a resource group named *myResourceGroup* in the *eastus* location.

```azurecli-interactive 
az group create --name myResourceGroup --location eastus
```

The next step is to create a file in your current shell, named *cloud-init.txt* and paste the following configuration. For this example, create the file in the Cloud Shell not on your local machine. You can use any editor of your choice. Enter sensible-editor cloud-init.txt to create the file and see a list of available editors. Use the editor of your choice. Some typical choices are [nano](https://linux.die.net/man/1/nano), [vim](https://linux.die.net/man/1/vi), or [ed](https://linux.die.net/man/1/ed). Make sure that the whole cloud-init file is copied correctly, especially the first line:

 SLES| Ubuntu | RHEL
|:--- |:--- |:---
| ` # cloud-config `<br>` package_upgrade: true `<br>` packages: `<br>`  - apache2 ` | ` # cloud-config `<br>` package_upgrade: true `<br>` packages: `<br>`  - httpd ` | ` # cloud-config `<br>` package_upgrade: true `<br>` packages: `<br>`  - httpd ` |
 
 
> [!NOTE]
> cloud-init has multiple [input types](https://cloudinit.readthedocs.io/en/latest/topics/format.html), cloud-init will use first line of the customData/userData to indicate how it should process the input, for example `#cloud-config` indicates that the content should be processed as a cloud-init config.

Exit the file and save the file according to the editor. Verify file name on exit.

The final step is to create a VM with the [az vm create](/cli/azure/vm) command. 

The following example creates a VM named `ubuntu2204` and creates SSH keys if they don't already exist in a default key location. To use a specific set of keys, use the `--ssh-key-value` option.  Use the `--custom-data` parameter to pass in your cloud-init config file. Provide the full path to the *cloud-init.txt* config if you saved the file outside of your present working directory. 

```azurecli-interactive 
az vm create \
  --resource-group myResourceGroup \
  --name ubuntu2204 \
  --image Canonical:UbuntuServer:22_04-lts:latest \
  --custom-data cloud-init.txt \
  --generate-ssh-keys 
```

When the VM is created, the Azure CLI shows information specific to your deployment. Take note of the `publicIpAddress`. This address is used to access the VM.  It takes some time for the VM to be created, the packages to install, and the app to start. There are background tasks that continue to run after the Azure CLI returns you to the prompt. You can SSH into the VM and use the steps outlined in the Troubleshooting section to view the cloud-init logs. 

You can also deploy a cloud-init enabled VM by passing the [parameters in ARM template](/azure/azure-resource-manager/templates/deploy-cli#inline-parameters).

## Troubleshooting cloud-init
Once the VM has been provisioned, cloud-init runs through all the modules and script defined in `--custom-data` in order to configure the VM.  If you need to troubleshoot any errors or omissions from the configuration, you need to search for the module name (`disk_setup` or `runcmd` for example) in the cloud-init log - located in **/var/log/cloud-init.log**.

> [!NOTE]
> Not every module failure results in a fatal cloud-init overall configuration failure. For example, using the `runcmd` module, if the script fails, cloud-init will still report provisioning succeeded because the runcmd module executed.

For more information of cloud-init logging, see the [cloud-init documentation](https://cloudinit.readthedocs.io/en/latest/development/logging.html)

## Telemetry
cloud-init collects usage data and sends it to Microsoft to help improve our products and services. Telemetry is only collected during the provisioning process (first boot of the VM). The data collected helps us investigate provisioning failures and monitor performance and reliability. Data collected doesn't include any identifiers (personal identifiers). Read our [privacy statement](https://go.microsoft.com/fwlink/?LinkId=521839) to learn more. Some examples of telemetry being collected are (this isn't an exhaustive list): OS-related information (cloud-init version, distro version, kernel version), performance metrics of essential VM provisioning actions (time to obtain DHCP lease, time to retrieve metadata necessary to configure the VM, etc.), cloud-init log, and dmesg log.

Telemetry collection is currently enabled for most of our marketplace images that use cloud-init. It's enabled by specifying KVP telemetry reporter for cloud-init. In most Azure Marketplace images, this configuration can be found in the file /etc/cloud/cloud.cfg.d/10-azure-kvp.cfg. Removing this file during image preparation disables telemetry collection for any VM created from this image. 

Sample content of 10-azure-kvp.cfg
```
reporting:
  logging:
    type: log
  telemetry:
    type: hyperv
```
## Next steps

[Troubleshoot issues with cloud-init](cloud-init-troubleshooting.md).


For cloud-init examples of configuration changes, see the following documents:
 
- [Add an additional Linux user to a VM](cloudinit-add-user.md)
- [Run a package manager to update existing packages on first boot](cloudinit-update-vm.md)
- [Change VM local hostname](cloudinit-update-vm-hostname.md) 
- [Install an application package, update configuration files and inject keys](tutorial-automate-vm-deployment.md)
