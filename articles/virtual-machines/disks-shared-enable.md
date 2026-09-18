---
title: Enable shared disks for Azure managed disks
description: Configure an Azure managed disk as a shared disk so that you can attach it to multiple virtual machines.
author: roygara
ms.service: azure-disk-storage
ms.topic: how-to
ms.date: 09/17/2026
ms.author: rogarana
ms.custom: devx-track-azurecli, devx-track-azurepowershell
# Customer intent: As a cloud engineer, I want to configure shared disks for Azure managed disks, so that I can enable simultaneous access from multiple virtual machines to support clustered applications.
---

# Enable shared disks for Azure managed disks

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

This article explains how to enable the shared disks feature for Azure managed disks. With Azure shared disks, you can attach a managed disk to multiple virtual machines (VMs) simultaneously, enabling the deployment or migration of clustered applications to Azure.
 
If you're looking for conceptual information on managed disks that have shared disks enabled, see [Azure shared disks](disks-shared.md).

## Prerequisites

The scripts and commands in this article require either:

- Version 6.0.0 or newer of the Azure PowerShell module.

Or
- The latest version of the Azure CLI.

## Limitations

[!INCLUDE [virtual-machines-disks-shared-limitations](./includes/virtual-machines-disks-shared-limitations.md)]

## Supported operating systems

Shared disks support several operating systems. For the supported operating systems, see the [Windows](./disks-shared.md#sample-windows-shared-disk-workloads) and [Linux](./disks-shared.md#sample-linux-shared-disk-workloads) sections of the conceptual article.

## Disk sizes

[!INCLUDE [virtual-machines-disks-shared-sizes](./includes/virtual-machines-disks-shared-sizes.md)]

## Deploy a Premium SSD as a shared disk

To deploy a managed disk with the shared disk feature enabled, set the `maxShares` property to a value greater than 1. This setting makes the disk shareable across multiple VMs.

> [!IMPORTANT]
> Host caching isn't supported for shared disks.
> 
> The value of `maxShares` can only be set or changed when a disk is unmounted from all VMs. See the [Disk sizes](#disk-sizes) for the allowed values for `maxShares`.

# [Azure portal](#tab/azure-portal)

1. Sign in to the Azure portal. 
1. Search for and select **Disks**.
1. Select **+ Create** to create a new managed disk.
1. On the **Basics** pane, select a **Region**, and then select **Change size**.

    :::image type="content" source="media/disks-shared-enable/create-shared-disk-basics-pane.png" alt-text="Screenshot of the Create a managed disk Basics pane with Region set to West US, Availability zone set to None, and Change size highlighted." lightbox="media/disks-shared-enable/create-shared-disk-basics-pane.png":::

1. Select the Premium SSD size and SKU that you want and select **OK**.

    :::image type="content" source="media/disks-shared-enable/select-premium-shared-disk.png" alt-text="Screenshot of the Disk SKU list with Premium SSD under locally redundant storage and zone-redundant storage highlighted." lightbox="media/disks-shared-enable/select-premium-shared-disk.png":::

1. Proceed through the deployment until you get to the **Advanced** pane.
1. For **Enable shared disk**, select **Yes**, and then select a value for **Max shares**.

    :::image type="content" source="media/disks-shared-enable/enable-premium-shared-disk.png" alt-text="Screenshot of the shared disk settings with Enable shared disk set to Yes and Max shares set to 2." lightbox="media/disks-shared-enable/enable-premium-shared-disk.png":::

1. Select **Review + create**.


# [Azure CLI](#tab/azure-cli)

Use the following Azure CLI command to create a Premium SSD as a shared disk.

```azurecli
az disk create -g myResourceGroup -n mySharedDisk --size-gb 1024 -l westcentralus --sku Premium_LRS --max-shares 2
```

# [Azure PowerShell](#tab/azure-powershell)

Use the following Azure PowerShell commands to create a Premium SSD as a shared disk.

```azurepowershell-interactive
$dataDiskConfig = New-AzDiskConfig -Location 'WestCentralUS' -DiskSizeGB 1024 -AccountType Premium_LRS -CreateOption Empty -MaxSharesCount 2

New-AzDisk -ResourceGroupName 'myResourceGroup' -DiskName 'mySharedDisk' -Disk $dataDiskConfig
```

# [Resource Manager template](#tab/azure-resource-manager)

Before using the following template, replace `[parameters('dataDiskName')]`, `[resourceGroup().location]`, `[parameters('dataDiskSizeGB')]`, and `[parameters('maxShares')]` with your own values.

```rest
{ 
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "dataDiskName": {
      "type": "string",
      "defaultValue": "mySharedDisk"
    },
    "dataDiskSizeGB": {
      "type": "int",
      "defaultValue": 1024
    },
    "maxShares": {
      "type": "int",
      "defaultValue": 2
    }
  },
  "resources": [
    {
      "type": "Microsoft.Compute/disks",
      "name": "[parameters('dataDiskName')]",
      "location": "[resourceGroup().location]",
      "apiVersion": "2019-07-01",
      "sku": {
        "name": "Premium_LRS"
      },
      "properties": {
        "creationData": {
          "createOption": "Empty"
        },
        "diskSizeGB": "[parameters('dataDiskSizeGB')]",
        "maxShares": "[parameters('maxShares')]"
      }
    }
  ] 
}
```


---

## Deploy a Standard SSD as a shared disk

To deploy a managed disk with the shared disk feature enabled, set the `maxShares` property to a value greater than 1. This setting makes the disk shareable across multiple VMs.

> [!IMPORTANT]
> Host caching isn't supported for shared disks.
> 
> The value of `maxShares` can only be set or changed when a disk is unmounted from all VMs. See the [Disk sizes](#disk-sizes) for the allowed values for `maxShares`.

# [Azure portal](#tab/azure-portal)

1. Sign in to the Azure portal. 
1. Search for and select **Disks**.
1. Select **+ Create** to create a new managed disk.
1. On the **Basics** pane, select a **Region**, and then select **Change size**.

    :::image type="content" source="media/disks-shared-enable/create-shared-disk-basics-pane.png" alt-text="Screenshot of the Create a managed disk Basics pane with Region set to West US, Availability zone set to None, and Change size highlighted." lightbox="media/disks-shared-enable/create-shared-disk-basics-pane.png":::

1. Select the Standard SSD size and SKU that you want and select **OK**.

    :::image type="content" source="media/disks-shared-enable/select-standard-ssd-shared-disk.png" alt-text="Screenshot of the Disk SKU list with Standard SSD under locally redundant storage and zone-redundant storage highlighted." lightbox="media/disks-shared-enable/select-standard-ssd-shared-disk.png":::

1. Proceed through the deployment until you get to the **Advanced** pane.
1. For **Enable shared disk**, select **Yes**, and then select a value for **Max shares**.

    :::image type="content" source="media/disks-shared-enable/enable-premium-shared-disk.png" alt-text="Screenshot of the shared disk settings with Enable shared disk set to Yes and Max shares set to 2." lightbox="media/disks-shared-enable/enable-premium-shared-disk.png":::

1. Select **Review + create**.

# [Azure CLI](#tab/azure-cli)

Use the following Azure CLI command to create a Standard SSD as a shared disk.

```azurecli
az disk create -g myResourceGroup -n mySharedDisk --size-gb 1024 -l westcentralus --sku StandardSSD_LRS --max-shares 2
```

# [Azure PowerShell](#tab/azure-powershell)

Use the following Azure PowerShell commands to create a Standard SSD as a shared disk.

```azurepowershell-interactive
$dataDiskConfig = New-AzDiskConfig -Location 'WestCentralUS' -DiskSizeGB 1024 -AccountType StandardSSD_LRS -CreateOption Empty -MaxSharesCount 2

New-AzDisk -ResourceGroupName 'myResourceGroup' -DiskName 'mySharedDisk' -Disk $dataDiskConfig
```

# [Resource Manager template](#tab/azure-resource-manager)

Before using the following template, replace the default values for the `dataDiskName`, `dataDiskSizeGB`, and `maxShares` parameters with your own values.

```rest
{ 
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "dataDiskName": {
      "type": "string",
      "defaultValue": "mySharedDisk"
    },
    "dataDiskSizeGB": {
      "type": "int",
      "defaultValue": 1024
    },
    "maxShares": {
      "type": "int",
      "defaultValue": 2
    }
  },
  "resources": [
    {
      "type": "Microsoft.Compute/disks",
      "name": "[parameters('dataDiskName')]",
      "location": "[resourceGroup().location]",
      "apiVersion": "2019-07-01",
      "sku": {
        "name": "StandardSSD_LRS"
      },
      "properties": {
        "creationData": {
          "createOption": "Empty"
        },
        "diskSizeGB": "[parameters('dataDiskSizeGB')]",
        "maxShares": "[parameters('maxShares')]"
      }
    }
  ] 
}
```

---

## Deploy an Ultra Disk as a shared disk

To deploy a managed disk with the shared disk feature enabled, change the `maxShares` parameter to a value greater than 1. This makes the disk shareable across multiple VMs.

> [!IMPORTANT]
> The value of `maxShares` can only be set or changed when a disk is unmounted from all VMs. See the [Disk sizes](#disk-sizes) for the allowed values for `maxShares`.

# [Azure portal](#tab/azure-portal)

1. Sign in to the Azure portal. 
1. Search for and select **Disks**.
1. Select **+ Create** to create a new managed disk.
1. On the **Basics** pane, select **Change size**.
1. Select Ultra Disk for the **Disk SKU**.

    :::image type="content" source="media/disks-shared-enable/select-ultra-shared-disk.png" alt-text="Screenshot of the Disk SKU list with Ultra Disk under locally redundant storage selected." lightbox="media/disks-shared-enable/select-ultra-shared-disk.png":::

1. Select the disk size that you want and select **OK**.
1. Proceed through the deployment until you get to the **Advanced** pane.
1. For **Enable shared disk**, select **Yes**, and then select a value for **Max shares**.
1. Select **Review + create**.

    :::image type="content" source="media/disks-shared-enable/enable-ultra-shared-disk.png" alt-text="Screenshot of the Advanced pane with Enable shared disk set to Yes, Max shares set to 2, Ultra Disk performance values, and logical sector size set to 4096 bytes." lightbox="media/disks-shared-enable/enable-ultra-shared-disk.png":::

# [Azure CLI](#tab/azure-cli)

##### Regional disk example

The following Azure CLI commands create a regional Ultra Disk as a shared disk, update its performance settings, and show its properties.

```azurecli
#Creating an Ultra shared Disk 
az disk create -g rg1 -n clidisk --size-gb 1024 -l westus --sku UltraSSD_LRS --max-shares 5 --disk-iops-read-write 2000 --disk-mbps-read-write 200 --disk-iops-read-only 100 --disk-mbps-read-only 1

#Updating an Ultra shared Disk 
az disk update -g rg1 -n clidisk --disk-iops-read-write 3000 --disk-mbps-read-write 300 --set diskIopsReadOnly=100 --set diskMbpsReadOnly=1

#Show shared disk properties:
az disk show -g rg1 -n clidisk
```

##### Zonal disk example

The following Azure CLI commands create an Ultra Disk as a shared disk in availability zone 1, update its performance settings, and show its properties.

```azurecli
#Creating an Ultra shared Disk 
az disk create -g rg1 -n clidisk --size-gb 1024 -l westus --sku UltraSSD_LRS --max-shares 5 --disk-iops-read-write 2000 --disk-mbps-read-write 200 --disk-iops-read-only 100 --disk-mbps-read-only 1 --zone 1

#Updating an Ultra shared Disk 
az disk update -g rg1 -n clidisk --disk-iops-read-write 3000 --disk-mbps-read-write 300 --set diskIopsReadOnly=100 --set diskMbpsReadOnly=1

#Show shared disk properties:
az disk show -g rg1 -n clidisk
```

# [Azure PowerShell](#tab/azure-powershell)

##### Regional disk example

The following Azure PowerShell commands create a regional Ultra Disk as a shared disk.

```azurepowershell-interactive
$datadiskconfig = New-AzDiskConfig -Location 'WestCentralUS' -DiskSizeGB 1024 -AccountType UltraSSD_LRS -CreateOption Empty -DiskIOPSReadWrite 2000 -DiskMBpsReadWrite 200 -DiskIOPSReadOnly 100 -DiskMBpsReadOnly 1 -MaxSharesCount 5

New-AzDisk -ResourceGroupName 'myResourceGroup' -DiskName 'mySharedDisk' -Disk $datadiskconfig
```

##### Zonal disk example

The following Azure PowerShell commands create an Ultra Disk as a shared disk in availability zone 1.

```azurepowershell-interactive
$datadiskconfig = New-AzDiskConfig -Location 'WestCentralUS' -DiskSizeGB 1024 -AccountType UltraSSD_LRS -CreateOption Empty -DiskIOPSReadWrite 2000 -DiskMBpsReadWrite 200 -DiskIOPSReadOnly 100 -DiskMBpsReadOnly 1 -MaxSharesCount 5 -Zone 1

New-AzDisk -ResourceGroupName 'myResourceGroup' -DiskName 'mySharedDisk' -Disk $datadiskconfig
```

# [Resource Manager template](#tab/azure-resource-manager)

##### Regional disk example

Before using the following template, replace the default values for the `diskName`, `location`, `dataDiskSizeGB`, `maxShares`, `diskIOPSReadWrite`, `diskMBpsReadWrite`, `diskIOPSReadOnly`, and `diskMBpsReadOnly` parameters with your own values.

```rest
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "diskName": {
      "type": "string",
      "defaultValue": "uShared30"
    },
    "location": {
        "type": "string",
        "defaultValue": "westus",
        "metadata": {
                "description": "Location for all resources."
        }
    },
    "dataDiskSizeGB": {
      "type": "int",
      "defaultValue": 1024
    },
    "maxShares": {
      "type": "int",
      "defaultValue": 2
    },
    "diskIOPSReadWrite": {
      "type": "int",
      "defaultValue": 2048
    },
    "diskMBpsReadWrite": {
      "type": "int",
      "defaultValue": 20
    },    
    "diskIOPSReadOnly": {
      "type": "int",
      "defaultValue": 100
    },
    "diskMBpsReadOnly": {
      "type": "int",
      "defaultValue": 1
    } 
  }, 
  "resources": [
    {
        "type": "Microsoft.Compute/disks",
        "name": "[parameters('diskName')]",
        "location": "[parameters('location')]",
        "apiVersion": "2019-07-01",
        "sku": {
            "name": "UltraSSD_LRS"
        },
        "properties": {
            "creationData": {
                "createOption": "Empty"
            },
            "diskSizeGB": "[parameters('dataDiskSizeGB')]",
            "maxShares": "[parameters('maxShares')]",
            "diskIOPSReadWrite": "[parameters('diskIOPSReadWrite')]",
            "diskMBpsReadWrite": "[parameters('diskMBpsReadWrite')]",
            "diskIOPSReadOnly": "[parameters('diskIOPSReadOnly')]",
            "diskMBpsReadOnly": "[parameters('diskMBpsReadOnly')]"
        }
    }
  ]
}
```


##### Zonal disk example

Before using the following template, replace the default values for the `diskName`, `location`, `dataDiskSizeGB`, `maxShares`, `diskIOPSReadWrite`, `diskMBpsReadWrite`, `diskIOPSReadOnly`, `diskMBpsReadOnly`, and `zone` parameters with your own values.

```rest
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "diskName": {
      "type": "string",
      "defaultValue": "uShared30"
    },
    "location": {
        "type": "string",
        "defaultValue": "westus",
        "metadata": {
                "description": "Location for all resources."
        }
    },
    "dataDiskSizeGB": {
      "type": "int",
      "defaultValue": 1024
    },
    "maxShares": {
      "type": "int",
      "defaultValue": 2
    },
    "diskIOPSReadWrite": {
      "type": "int",
      "defaultValue": 2048
    },
    "diskMBpsReadWrite": {
      "type": "int",
      "defaultValue": 20
    },    
    "diskIOPSReadOnly": {
      "type": "int",
      "defaultValue": 100
    },
    "diskMBpsReadOnly": {
      "type": "int",
      "defaultValue": 1
    },
	"zone": {
		"type": "int",
		"allowedValues": [
			1,
			2,
			3
		],
	  "metadata": {
			"description": "Zone to deploy to."
		}
	} 
  }, 
  "resources": [
    {
        "type": "Microsoft.Compute/disks",
        "name": "[parameters('diskName')]",
        "location": "[parameters('location')]",
		"zones": ["[parameters('zone')]"],
        "apiVersion": "2019-07-01",
        "sku": {
            "name": "UltraSSD_LRS"
        },
        "properties": {
            "creationData": {
                "createOption": "Empty"
            },
            "diskSizeGB": "[parameters('dataDiskSizeGB')]",
            "maxShares": "[parameters('maxShares')]",
            "diskIOPSReadWrite": "[parameters('diskIOPSReadWrite')]",
            "diskMBpsReadWrite": "[parameters('diskMBpsReadWrite')]",
            "diskIOPSReadOnly": "[parameters('diskIOPSReadOnly')]",
            "diskMBpsReadOnly": "[parameters('diskMBpsReadOnly')]"
        }
    }
  ]
}
```


---

## Share an existing disk

To share an existing disk or update how many VMs can mount it, set the `maxShares` parameter by using either Azure PowerShell or Azure CLI. To disable sharing, set `maxShares` to 1.

> [!IMPORTANT]
> Host caching isn't supported for shared disks.
> 
> The value of `maxShares` can only be set or changed when a disk is unmounted from all VMs. See the [Disk sizes](#disk-sizes) for the allowed values for `maxShares`.
> Before detaching a disk, record the LUN ID for when you reattach it.

### Azure PowerShell

The following Azure PowerShell commands update the sharing configuration of an existing disk.

```azurepowershell
$datadiskconfig = Get-AzDisk -DiskName "mySharedDisk"
$datadiskconfig.maxShares = 3

Update-AzDisk -ResourceGroupName 'myResourceGroup' -DiskName 'mySharedDisk' -Disk $datadiskconfig
```

### Azure CLI

The following Azure CLI command updates the sharing configuration of an existing disk.

```azurecli
#Modifying a disk to enable or modify sharing configuration

az disk update --name mySharedDisk --max-shares 5 --resource-group myResourceGroup
```

## Use Azure shared disks with your VMs

After you deploy a shared disk with `maxShares > 1`, you can mount the disk to one or more of your VMs.

> [!NOTE]
> Host caching isn't supported for shared disks.
> 
> If you're deploying an Ultra Disk, make sure it matches the necessary requirements. See [Using Azure Ultra Disks](disks-enable-ultra-ssd.md) for details.

The following Azure PowerShell commands create a VM and attach the shared disk as a data disk.

```azurepowershell-interactive

$resourceGroup = "myResourceGroup"
$location = "WestCentralUS"

$vm = New-AzVm -ResourceGroupName $resourceGroup -Name "myVM" -Location $location -VirtualNetworkName "myVnet" -SubnetName "mySubnet" -SecurityGroupName "myNetworkSecurityGroup" -PublicIpAddressName "myPublicIpAddress"

$dataDisk = Get-AzDisk -ResourceGroupName $resourceGroup -DiskName "mySharedDisk"

$vm = Add-AzVMDataDisk -VM $vm -Name "mySharedDisk" -CreateOption Attach -ManagedDiskId $dataDisk.Id -Lun 0

update-AzVm -VM $vm -ResourceGroupName $resourceGroup
```

## Supported SCSI persistent reservation commands

After you mount the shared disk to your VMs in your cluster, you can establish quorum and read/write to the disk by using Small Computer System Interface (SCSI) persistent reservation (PR) commands.

The following SCSI persistent reservation commands are available when using Azure shared disks:

- `PR_REGISTER_KEY`
- `PR_REGISTER_AND_IGNORE`
- `PR_GET_CONFIGURATION`
- `PR_RESERVE`
- `PR_PREEMPT_RESERVATION`
- `PR_CLEAR_RESERVATION`
- `PR_RELEASE_RESERVATION`

When you use `PR_RESERVE`, `PR_PREEMPT_RESERVATION`, or `PR_RELEASE_RESERVATION`, provide one of the following persistent reservation types:

- `PR_NONE`
- `PR_WRITE_EXCLUSIVE`
- `PR_EXCLUSIVE_ACCESS`
- `PR_WRITE_EXCLUSIVE_REGISTRANTS_ONLY`
- `PR_EXCLUSIVE_ACCESS_REGISTRANTS_ONLY`
- `PR_WRITE_EXCLUSIVE_ALL_REGISTRANTS`
- `PR_EXCLUSIVE_ACCESS_ALL_REGISTRANTS`

You also need to provide a persistent reservation key when you use `PR_RESERVE`, `PR_REGISTER_AND_IGNORE`, `PR_REGISTER_KEY`, `PR_PREEMPT_RESERVATION`, `PR_CLEAR_RESERVATION`, or `PR_RELEASE_RESERVATION`.


## Next steps

If you have more questions, see the [shared disks](/azure/virtual-machines/faq-for-disks#azure-shared-disks) section of the FAQ.