---
title: Increase performance of Premium SSDs and Standard SSD/HDDs
description: Increase the performance of Azure Premium SSDs and Standard SSD/HDDs using performance plus.
author: roygara
ms.service: azure-disk-storage
ms.topic: how-to
ms.date: 04/15/2025
ms.author: rogarana
ms.custom: devx-track-azurepowershell
# Customer intent: As an IT administrator, I want to enable performance enhancements on Azure disks, so that I can increase IOPS and throughput for demanding workloads without incurring additional costs.
---

# Increase IOPS and throughput limits for Azure Premium SSDs and Standard SSD/HDDs

The Input/Output Operations Per Second (IOPS) and throughput limits for Azure Premium solid-state drives (SSD), Standard SSDs, and Standard hard disk drives (HDD) that are 513 GiB and larger can be increased by enabling performance plus. Enabling performance plus improves the experience for workloads that require high IOPS and throughput, such as database and transactional workloads. There's no extra charge for enabling performance plus on a disk.

Once enabled, the IOPS and throughput limits for an eligible disk increase to the higher maximum limits. To see the new IOPS and throughput limits for eligible disks, consult the columns that begin with "*Expanded" in the [Scalability and performance targets for VM disks](disks-scalability-targets.md) article.

## Limitations

- Can only be enabled on Standard HDD, Standard SSD, and Premium SSD managed disks that are 513 GiB or larger
- Can only be enabled during disk creation
    - To work around this, create a snapshot of your disk, then create a new disk from the snapshot
    - Can't be enabled on disks created during virtual machine creation.
- Not supported for disks recovered with Azure Site Recovery

## Prerequisites

Either use the Azure Cloud Shell to run your commands or install a version of the [Azure PowerShell module](/powershell/azure/install-azure-powershell) 9.5 or newer, or a version of the [Azure CLI](/cli/azure/install-azure-cli) that is 2.44.0 or newer.

## Enable performance plus

You need to create a new disk to use performance plus. The following scripts show how to create a disk with performance plus enabled and, if desired, attach it to a VM. The commands have been organized into self-contained steps for reliability.

# [Azure CLI](#tab/azure-cli)

### Create a resource group

This step creates a resource group with a unique name.

```azurecli
export RANDOM_SUFFIX=$(openssl rand -hex 3)
export MY_RG="PerfPlusRG$RANDOM_SUFFIX"
export REGION="WestUS2"
az group create -g $MY_RG -l $REGION
```

Results:

<!-- expected_similarity=0.3 -->
```JSON
{
  "id": "/subscriptions/xxxxx/resourceGroups/PerfPlusRGxxx",
  "location": "WestUS2",
  "name": "PerfPlusRGxxx",
  "properties": {
    "provisioningState": "Succeeded"
  }
}
```

### Create a new disk with performance plus enabled

This step creates a new disk of 513 GiB (or larger) with performance plus enabled using a valid SKU value.

```azurecli
export MY_DISK="PerfPlusDisk$RANDOM_SUFFIX"
export SKU="Premium_LRS"
export DISK_SIZE=513
az disk create -g $MY_RG -n $MY_DISK --size-gb $DISK_SIZE --sku $SKU -l $REGION --performance-plus true
```

Results:

<!-- expected_similarity=0.3 -->
```JSON
{
  "id": "/subscriptions/xxxxx/resourceGroups/PerfPlusRGxxx/providers/Microsoft.Compute/disks/PerfPlusDiskxxx",
  "location": "WestUS2",
  "name": "PerfPlusDiskxxx",
  "properties": {
    "provisioningState": "Succeeded",
    "diskSizeGb": 513,
    "sku": "Premium_LRS",
    "performancePlus": true
  },
  "type": "Microsoft.Compute/disks"
}
```

### Attempt to attach the disk to a VM

This optional step attempts to attach the disk to an existing VM. It first checks if the VM exists and then proceeds accordingly.

```azurecli
export MY_VM="NonExistentVM"
if az vm show -g $MY_RG -n $MY_VM --query "name" --output tsv >/dev/null 2>&1; then
    az vm disk attach --vm-name $MY_VM --name $MY_DISK --resource-group $MY_RG 
else
    echo "VM $MY_VM not found. Skipping disk attachment."
fi
```

Results:

<!-- expected_similarity=0.3 -->
```text
VM NonExistentVM not found. Skipping disk attachment.
```

### Create a new disk from an existing disk or snapshot with performance plus enabled

This series of steps creates a separate resource group and then creates a new disk from an existing disk or snapshot. Replace the SOURCE_URI with a valid source blob URI that belongs to the same region (WestUS2) as the disk.

#### Create a resource group for migration

```azurecli
export RANDOM_SUFFIX=$(openssl rand -hex 3)
export MY_MIG_RG="PerfPlusMigrRG$RANDOM_SUFFIX"
export REGION="WestUS2"
az group create -g $MY_MIG_RG -l $REGION
```

Results:

<!-- expected_similarity=0.3 -->
```JSON
{
  "id": "/subscriptions/xxxxx/resourceGroups/PerfPlusMigrRGxxx",
  "location": "WestUS2",
  "name": "PerfPlusMigrRGxxx",
  "properties": {
    "provisioningState": "Succeeded"
  }
}
```

#### Create the disk from an existing snapshot or disk

```azurecli
# Create a snapshot from the original disk
export MY_SNAPSHOT_NAME="PerfPlusSnapshot$RANDOM_SUFFIX"
echo "Creating snapshot from original disk..."
az snapshot create \
  --name $MY_SNAPSHOT_NAME \
  --resource-group $MY_RG \
  --source $MY_DISK

# Get the snapshot ID for use as source
SNAPSHOT_ID=$(az snapshot show \
  --name $MY_SNAPSHOT_NAME \
  --resource-group $MY_RG \
  --query id \
  --output tsv)

echo "Using snapshot ID: $SNAPSHOT_ID"

# Create the new disk using the snapshot as source
export MY_MIG_DISK="PerfPlusMigrDisk$RANDOM_SUFFIX"
export SKU="Premium_LRS"
export DISK_SIZE=513

az disk create \
  --name $MY_MIG_DISK \
  --resource-group $MY_MIG_RG \
  --size-gb $DISK_SIZE \
  --performance-plus true \
  --sku $SKU \
  --source $SNAPSHOT_ID \
  --location $REGION
```

Results:

<!-- expected_similarity=0.3 -->
```JSON
{
  "id": "/subscriptions/xxxxx/resourceGroups/PerfPlusMigrRGxxx/providers/Microsoft.Compute/disks/PerfPlusMigrDiskxxx",
  "location": "WestUS2",
  "name": "PerfPlusMigrDiskxxx",
  "properties": {
    "provisioningState": "Succeeded",
    "diskSizeGb": 513,
    "sku": "Premium_LRS",
    "performancePlus": true,
    "source": "https://examplestorageaccount.blob.core.windows.net/snapshots/sample-westus2.vhd"
  },
  "type": "Microsoft.Compute/disks"
}
```

# [Azure PowerShell](#tab/azure-powershell)

### Create a resource group

This step creates a resource group with a unique name.

```azurepowershell
$RANDOM_SUFFIX = (New-Guid).Guid.Substring(0,6)
$myRG = "PerfPlusRG$RANDOM_SUFFIX"
$region = "WestUS2"
New-AzResourceGroup -Name $myRG -Location $region
```

Results:

<!-- expected_similarity=0.3 -->
```JSON
{
  "ResourceGroupName": "PerfPlusRGxxx",
  "Location": "WestUS2",
  "ProvisioningState": "Succeeded"
}
```

### Create a new disk with performance plus enabled

This step creates a new disk with performance plus enabled using a valid SKU value.

```azurepowershell
$myDisk = "PerfPlusDisk$RANDOM_SUFFIX"
$sku = "Premium_LRS"
$size = 513
$diskConfig = New-AzDiskConfig -Location $region -CreateOption Empty -DiskSizeGB $size -SkuName $sku -PerformancePlus $true 
$dataDisk = New-AzDisk -ResourceGroupName $myRG -DiskName $myDisk -Disk $diskConfig
```

Results:

<!-- expected_similarity=0.3 -->
```JSON
{
  "ResourceGroup": "PerfPlusRGxxx",
  "Name": "PerfPlusDiskxxx",
  "Location": "WestUS2",
  "Sku": "Premium_LRS",
  "DiskSizeGB": 513,
  "PerformancePlus": true,
  "ProvisioningState": "Succeeded"
}
```

### Attempt to attach the disk to a VM

This optional step checks whether the specified VM exists before attempting the disk attachment.

```azurepowershell
$myVM = "NonExistentVM"
if (Get-AzVM -ResourceGroupName $myRG -Name $myVM -ErrorAction SilentlyContinue) {
    Add-AzVMDataDisk -VMName $myVM -ResourceGroupName $myRG -DiskName $myDisk -Lun 0 -CreateOption Empty -ManagedDiskId $dataDisk.Id
} else {
    Write-Output "VM $myVM not found. Skipping disk attachment."
}
```

Results:

<!-- expected_similarity=0.3 -->
```text
VM NonExistentVM not found. Skipping disk attachment.
```

### Create a new disk from an existing disk or snapshot with performance plus enabled

This series of steps creates a separate resource group and then creates a new disk from an existing disk or snapshot. Replace the $sourceURI with a valid source blob URI that belongs to the same region (WestUS2) as the disk.

#### Create a resource group for migration

```azurepowershell
$RANDOM_SUFFIX = (New-Guid).Guid.Substring(0,6)
$myMigrRG = "PerfPlusMigrRG$RANDOM_SUFFIX"
$region = "WestUS2"
New-AzResourceGroup -Name $myMigrRG -Location $region
```

Results:

<!-- expected_similarity=0.3 -->
```JSON
{
  "ResourceGroupName": "PerfPlusMigrRGxxx",
  "Location": "WestUS2",
  "ProvisioningState": "Succeeded"
}
```

#### Create the disk from an existing snapshot or disk

```azurepowershell
$myDisk = "PerfPlusMigrDisk$RANDOM_SUFFIX"
$sku = "Premium_LRS"
$size = 513
$sourceURI = "https://examplestorageaccount.blob.core.windows.net/snapshots/sample-westus2.vhd"  # Replace with a valid source blob URI in WestUS2
$diskConfig = New-AzDiskConfig -Location $region -CreateOption Copy -DiskSizeGB $size -SkuName $sku -PerformancePlus $true -SourceResourceID $sourceURI
$dataDisk = New-AzDisk -ResourceGroupName $myMigrRG -DiskName $myDisk -Disk $diskConfig
```

Results:

<!-- expected_similarity=0.3 -->
```JSON
{
  "ResourceGroup": "PerfPlusMigrRGxxx",
  "Name": "PerfPlusMigrDiskxxx",
  "Location": "WestUS2",
  "Sku": "Premium_LRS",
  "DiskSizeGB": 513,
  "PerformancePlus": true,
  "SourceResourceID": "https://examplestorageaccount.blob.core.windows.net/snapshots/sample-westus2.vhd",
  "ProvisioningState": "Succeeded"
}
```

#### Attempt to attach the migrated disk to a VM

This optional step verifies the existence of the specified VM before attempting disk attachment.

```azurepowershell
$myVM = "NonExistentVM"
if (Get-AzVM -ResourceGroupName $myMigrRG -Name $myVM -ErrorAction SilentlyContinue) {
    Add-AzVMDataDisk -VMName $myVM -ResourceGroupName $myMigrRG -DiskName $myDisk -Lun 0 -CreateOption Empty -ManagedDiskId $dataDisk.Id
} else {
    Write-Output "VM $myVM not found. Skipping disk attachment."
}
```

Results:

<!-- expected_similarity=0.3 -->
```text
VM NonExistentVM not found. Skipping disk attachment.
```

# [Azure portal](#tab/portal)

### Create a new disk with performance plus enabled

1. Sign in to the [Azure portal](https://portal.azure.com).
1. Search for and navigate to **Disks** and create a new disk.
1. On **Basics**, fill out the required fields.
1. Select the **Source type** that you'd like.
1. Then select **Change size** and select the disk type you'd like and choose a size larger than 513 GiB.
1. Proceed to **Advanced** and select the checkbox next to **Enable performance plus**.
1. Select **Review + create** and then deploy your disk.

:::image type="content" source="media/disks-enable-performance/disks-performance-plus-enable.png" alt-text="Screenshot of disks advanced page in the disk portal deployment." lightbox="media/disks-enable-performance/disks-performance-plus-enable.png":::

---
