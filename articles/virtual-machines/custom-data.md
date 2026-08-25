---
 title: Custom data on Azure virtual machines
 description: Learn how to use custom data to inject scripts or other metadata into Linux and Windows Azure virtual machines at provisioning time.
 services: virtual-machines
 author: mimckitt
 ms.service: azure-virtual-machines
 ms.topic: how-to
 ms.date: 05/28/2026
 ms.author: mimckitt
 ms.reviewer: mattmcinnes
# Customer intent: As a cloud administrator, I want to inject custom data into Azure virtual machines during provisioning, so that I can configure the VM with specific scripts and metadata for seamless setup and management.
---

# Custom data on Azure virtual machines

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets

You might need to inject a script or other metadata into a Microsoft Azure virtual machine (VM) at provisioning time. Azure supports this scenario through a feature called *custom data*.

Custom data is made available to the VM during first startup or setup, which is called *provisioning*. Provisioning is the process where VM creation parameters (for example, host name, username, password, certificates, custom data, and keys) are made available to the VM. A provisioning agent, such as the [cloud-init](./linux/using-cloud-init.md#troubleshooting-cloud-init), processes those parameters.

> [!NOTE]
> Azure also offers a separate [user data](./user-data.md) feature. Unlike custom data, user data is retrievable from the [Azure Instance Metadata Service (IMDS)](./linux/instance-metadata-service.md?tabs=linux#get-user-data), is persistent for the lifetime of the VM, and can be updated without stopping or reimaging the VM. Use user data when you need post-provision access or updates; use custom data when you need a payload processed by the provisioning agent at first boot.

## Pass custom data to the VM
To use custom data, you must Base64-encode the contents before passing the data to the API--unless you're using a CLI tool that does the conversion for you, such as the Azure CLI. The size can't exceed 64 KB.

In the CLI, you can pass your custom data as a file, as the following example shows. The file is converted to Base64.

```azurecli
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image Ubuntu2204 \
  --custom-data cloud-init.txt \
  --generate-ssh-keys
```

In Azure Resource Manager, there's a [base64 function](/azure/azure-resource-manager/templates/template-functions-string#base64):

```json
"name": "[parameters('virtualMachineName')]",
"type": "Microsoft.Compute/virtualMachines",
"apiVersion": "2019-07-01",
"location": "[parameters('location')]",
"dependsOn": [
..],
"variables": {
        "customDataBase64": "[base64(parameters('stringData'))]"
    },
"properties": {
..
    "osProfile": {
        "computerName": "[parameters('virtualMachineName')]",
        "adminUsername": "[parameters('adminUsername')]",
        "adminPassword": "[parameters('adminPassword')]",
        "customData": "[variables('customDataBase64')]"
        },
```

## Process custom data
The provisioning agents installed on the VMs handle communication with the platform and placing data on the file system. 

### Windows
Custom data is placed in *%SystemDrive%\AzureData\CustomData.bin* as a binary file, but it isn't processed. To process this file, build a custom image and write code that reads *CustomData.bin*.

The *%SystemDrive%\AzureData* folder is secured during provisioning so that only **NT AUTHORITY\SYSTEM** and **BUILTIN\Administrators** have full control, and *CustomData.bin* inherits these permissions. If you copy *CustomData.bin* elsewhere for processing, set permissions on the destination explicitly, as its permissions don’t transfer with the file..

### Linux  
On Linux operating systems, custom data is passed to the VM via the *ovf-env.xml* file. That file is copied to the */var/lib/waagent* directory during provisioning. Newer versions of the Linux Agent copy the Base64-encoded data to */var/lib/waagent/CustomData* for convenience.

Azure currently supports two provisioning agents:

* **Linux Agent**. By default, the agent doesn't process custom data. You need to build a custom image with the data enabled. The [relevant settings](https://github.com/Azure/WALinuxAgent#configuration) are:
  
  * `Provisioning.DecodeCustomData`
  * `Provisioning.ExecuteCustomData`

  When you enable custom data and run a script, the virtual machine will not report a successful VM provision until the script has finished executing. If the script exceeds the total VM provisioning time limit of 40 minutes, VM creation fails. 
  
  If the script fails to run, or errors happen during execution, that's not a fatal provisioning failure. You need to create a notification path to alert you for the completion state of the script.

  To troubleshoot custom data execution, review */var/log/waagent.log*.

* **cloud-init**. By default, this agent processes custom data. It accepts [multiple formats](https://cloudinit.readthedocs.io/en/latest/topics/format.html) of custom data, such as cloud-init configuration and scripts. 

  Similar to the Linux Agent, if errors happen during execution of the configuration processing or scripts when cloud-init is processing the custom data, that's not a fatal provisioning failure. You need to create a notification path to alert you for the completion state of the script. 
  
  However, unlike the Linux Agent, cloud-init doesn't wait for custom data configurations from the user to finish before reporting to the platform that the VM is ready. For more information on cloud-init on Azure, including troubleshooting, see [cloud-init support for virtual machines in Azure](./linux/using-cloud-init.md).


## FAQ
### Can I update custom data after the VM has been created?
For single VMs, you can't update custom data in the VM model. But for Virtual Machine Scale Sets, you can update custom data. For more information, see [Modify a Scale Set](../virtual-machine-scale-sets/virtual-machine-scale-sets-upgrade-scale-set.md#how-to-update-global-scale-set-properties). When you update custom data in the model for a Virtual Machine Scale Set:

* Existing instances in the scale set don't get the updated custom data until they're updated to the latest model and reimaged.
* New instances receive the new custom data.

### Can I place sensitive values in custom data?
We advise *not* to store sensitive data in custom data. For more information, see [Azure data security and encryption best practices](/azure/security/fundamentals/data-encryption-best-practices).

### Is custom data made available in IMDS?
No. Custom data isn't surfaced through the Azure Instance Metadata Service (IMDS). If you need to retrieve a payload from inside the VM after provisioning, or update it without stopping or reimaging the VM, use the [user data](./user-data.md) feature instead. For details on reading user data from IMDS, see [Get user data](./linux/instance-metadata-service.md?tabs=linux#get-user-data).
