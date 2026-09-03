---
title: Restrict import and export access for managed disks using Private Link
description: Use Azure CLI or Azure PowerShell to configure Private Link for managed disks and restrict import and export access to your virtual network.
author: roygara
ms.service: azure-disk-storage
ms.topic: how-to
ms.date: 09/02/2026
ms.author: rogarana
ms.custom: references_regions, devx-track-azurecli, devx-track-azurepowershell, linux-related-content, windows-related-content
ai-usage: ai-assisted
# Customer intent: "As a cloud administrator, I want to configure Private Link for managed disks by using Azure CLI or Azure PowerShell, so that I can restrict import and export access to my virtual network."
---

# Restrict import and export access for managed disks by using Private Link

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets

Use [private endpoints](/azure/private-link/private-endpoint-overview) to restrict managed disk import and export. Securely access data over [Azure Private Link](/azure/private-link/private-link-overview) from clients in your Azure virtual network. The private endpoint uses an IP address from your virtual network address space for your managed disks service. Traffic between clients in your virtual network and managed disks stays on the virtual network and Private Link on the Microsoft backbone network, which reduces exposure to the public internet.

This article shows how to configure Private Link for managed disk import and export by using Azure CLI or Azure PowerShell. Create a disk access resource and link it to a virtual network in the same subscription by creating a private endpoint. Then associate a disk or snapshot with the disk access resource. Finally, set the network access policy of the disk or snapshot to `AllowPrivate` to limit access to your virtual network.

Set the network access policy to `DenyAll` to prevent anyone from exporting data from a disk or snapshot. The default network access policy is `AllowAll`.

## Prerequisites

Before you begin, install the latest [Azure CLI](/cli/azure/install-azure-cli) or [Azure PowerShell module](/powershell/azure/install-azure-powershell).

## Limitations

[!INCLUDE [virtual-machines-disks-private-links-limitations](../includes/virtual-machines-disks-private-links-limitations.md)]


## Sign in and set variables

Choose a tab to use Azure CLI or Azure PowerShell. Set values for your subscription, resource group, region, disk access resource, virtual network, subnet, private endpoint, and private DNS zone. The remaining procedures reuse these variables. To create a protected snapshot, also provide the name of an existing source disk and a name for the new snapshot.

# [Azure CLI](#tab/azure-cli)

```azurecli-interactive
subscriptionId=yourSubscriptionId
resourceGroupName=yourResourceGroupName
region=northcentralus
diskAccessName=yourDiskAccessForPrivateLink
vnetName=yourVnetForPrivateLink
subnetName=yourSubnetForPrivateLink
privateEndpointName=yourPrivateEndpointForSecureMDExportImport
privateEndpointConnectionName=yourPrivateEndpointConnection
privateDnsZoneName=privatelink.blob.core.windows.net
privateDnsZoneLinkName=yourDnsLink
privateDnsZoneGroupName=yourZoneGroup

# The name of an existing disk that is the source of the snapshot.
sourceDiskName=yourSourceDiskForSnapshot

# The name of the new snapshot secured with Private Link.
snapshotNameSecuredWithPrivateLink=yourSnapshotNameSecuredWithPrivateLink

az login
az account set --subscription $subscriptionId
```

# [Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
$subscriptionId = "yourSubscriptionId"
$resourceGroupName = "yourResourceGroupName"
$location = "NorthCentralUS"
$diskAccessName = "yourDiskAccessForPrivateLink"
$vnetName = "yourVnetForPrivateLink"
$subnetName = "yourSubnetForPrivateLink"
$privateEndpointName = "yourPrivateEndpointForSecureMDExportImport"
$privateEndpointConnectionName = "yourPrivateEndpointConnection"
$privateDnsZoneName = "privatelink.blob.core.windows.net"
$privateDnsZoneLinkName = "yourDnsLink"
$privateDnsZoneGroupName = "yourZoneGroup"

# The name of an existing disk that is the source of the snapshot.
$sourceDiskName = "yourSourceDiskForSnapshot"

# The name of the new snapshot secured with Private Link.
$snapshotNameSecuredWithPrivateLink = "yourSnapshotNameSecuredWithPrivateLink"

Connect-AzAccount
Set-AzContext -Subscription $subscriptionId
```

---

## Create a disk access resource

# [Azure CLI](#tab/azure-cli)

Use the resource group, region, and disk access resource name that you defined earlier. The [az disk-access create](/cli/azure/disk-access#az-disk-access-create) command creates the disk access resource. Then, [az disk-access show](/cli/azure/disk-access#az-disk-access-show) stores its resource ID in `$diskAccessId` for later procedures.

```azurecli
az disk-access create -n $diskAccessName -g $resourceGroupName -l $region

diskAccessId=$(az disk-access show -n $diskAccessName -g $resourceGroupName --query [id] -o tsv)
```

# [Azure PowerShell](#tab/azure-powershell)

Use the resource group, location, and disk access resource name that you defined earlier. The [New-AzDiskAccess](/powershell/module/az.compute/new-azdiskaccess) cmdlet creates the disk access resource and stores it in `$diskAccess`. The next command stores its resource ID in `$diskAccessId` for later procedures.

```azurepowershell
$diskAccess = New-AzDiskAccess -ResourceGroupName $resourceGroupName -Name $diskAccessName -Location $location
$diskAccessId = $diskAccess.Id
```

---

## Create a virtual network

Network policies such as network security groups (NSGs) aren't supported for private endpoints. To deploy a private endpoint on a subnet, disable private endpoint network policies on that subnet.

# [Azure CLI](#tab/azure-cli)

Use the resource group, virtual network, and subnet names that you defined earlier. The [az network vnet create](/cli/azure/network/vnet#az-network-vnet-create) command creates the virtual network and subnet. Then, [az network vnet subnet update](/cli/azure/network/vnet/subnet#az-network-vnet-subnet-update) disables private endpoint network policies on the subnet.

```azurecli
az network vnet create --resource-group $resourceGroupName \
    --name $vnetName \
    --subnet-name $subnetName

az network vnet subnet update --resource-group $resourceGroupName \
    --name $subnetName \
    --vnet-name $vnetName \
    --private-endpoint-network-policies Disabled
```

# [Azure PowerShell](#tab/azure-powershell)

Use the resource group, location, virtual network, and subnet names that you defined earlier. The [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) cmdlet creates a subnet configuration with private endpoint network policies disabled. Then, [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) creates the virtual network and subnet and stores the virtual network in `$vnet`.

```azurepowershell
$subnetConfig = New-AzVirtualNetworkSubnetConfig -Name $subnetName -AddressPrefix "10.0.0.0/24" -PrivateEndpointNetworkPoliciesFlag "Disabled"

$vnet = New-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name $vnetName -Location $location -AddressPrefix "10.0.0.0/16" -Subnet $subnetConfig
```

---

## Create a private endpoint for the disk access resource

# [Azure CLI](#tab/azure-cli)

Use the resource group, private endpoint, virtual network, subnet, and connection names that you defined earlier. This procedure also uses `$diskAccessId` from the disk access resource procedure. The [az network private-endpoint create](/cli/azure/network/private-endpoint#az-network-private-endpoint-create) command creates a private endpoint for the disk access resource.

```azurecli
az network private-endpoint create --resource-group $resourceGroupName \
    --name $privateEndpointName \
    --vnet-name $vnetName \
    --subnet $subnetName \
    --private-connection-resource-id $diskAccessId \
    --group-ids disks \
    --connection-name $privateEndpointConnectionName
```

# [Azure PowerShell](#tab/azure-powershell)

Use the resource group, location, private endpoint, subnet, and connection names that you defined earlier. This procedure also uses `$vnet` and `$diskAccessId` from the preceding procedures. The cmdlets get the subnet, create a private link service connection to the disk access resource, and then create the private endpoint.

```azurepowershell
$subnet = Get-AzVirtualNetworkSubnetConfig -Name $subnetName -VirtualNetwork $vnet

$privateLinkServiceConnection = New-AzPrivateLinkServiceConnection -Name $privateEndpointConnectionName -PrivateLinkServiceId $diskAccessId -GroupId "disks"

$privateEndpoint = New-AzPrivateEndpoint -ResourceGroupName $resourceGroupName -Name $privateEndpointName -Location $location -Subnet $subnet -PrivateLinkServiceConnection $privateLinkServiceConnection
```

---

## Configure the private DNS zone

Create a private DNS zone for the storage blob domain, create a virtual network link, and then create a DNS zone group that associates the private endpoint with the private DNS zone.

# [Azure CLI](#tab/azure-cli)

Use the resource group, private DNS zone, virtual network link, private endpoint, and DNS zone group names that you defined earlier. The following commands create the private DNS zone, link it to the virtual network, and associate it with the private endpoint.

```azurecli
az network private-dns zone create --resource-group $resourceGroupName \
    --name $privateDnsZoneName

az network private-dns link vnet create --resource-group $resourceGroupName \
    --zone-name $privateDnsZoneName \
    --name $privateDnsZoneLinkName \
    --virtual-network $vnetName \
    --registration-enabled false

az network private-endpoint dns-zone-group create \
   --resource-group $resourceGroupName \
    --endpoint-name $privateEndpointName \
   --name $privateDnsZoneGroupName \
   --private-dns-zone $privateDnsZoneName \
   --zone-name disks
```

# [Azure PowerShell](#tab/azure-powershell)

Use the resource group, private DNS zone, virtual network link, private endpoint, and DNS zone group names that you defined earlier. The following cmdlets create the private DNS zone, link it to `$vnet`, configure the zone, and associate it with the private endpoint.

```azurepowershell
$privateDnsZone = New-AzPrivateDnsZone -ResourceGroupName $resourceGroupName -Name $privateDnsZoneName

New-AzPrivateDnsVirtualNetworkLink -ResourceGroupName $resourceGroupName -ZoneName $privateDnsZoneName -Name $privateDnsZoneLinkName -VirtualNetworkId $vnet.Id

$privateDnsZoneConfig = New-AzPrivateDnsZoneConfig -Name $privateDnsZoneName -PrivateDnsZoneId $privateDnsZone.ResourceId

New-AzPrivateDnsZoneGroup -ResourceGroupName $resourceGroupName -PrivateEndpointName $privateEndpointName -Name $privateDnsZoneGroupName -PrivateDnsZoneConfig $privateDnsZoneConfig
```

---

## Create a managed disk protected with Private Link

# [Azure CLI](#tab/azure-cli)

Define the managed disk name, storage SKU, and size for this procedure. The procedure reuses the resource group, region, and disk access resource name that you defined earlier. The [az disk create](/cli/azure/disk#az-disk-create) command creates an empty managed disk, associates it with the disk access resource, and sets its network access policy to `AllowPrivate`.

```azurecli-interactive
# These variables are specific to this step.
diskName=yourDiskName
diskSkuName=Standard_LRS
diskSizeGB=128

diskAccessId=$(az resource show -n $diskAccessName -g $resourceGroupName --namespace Microsoft.Compute --resource-type diskAccesses --query [id] -o tsv)

az disk create -n $diskName \
    -g $resourceGroupName \
    -l $region \
    --size-gb $diskSizeGB \
    --sku $diskSkuName \
    --network-access-policy AllowPrivate \
    --disk-access $diskAccessId
```

# [Azure PowerShell](#tab/azure-powershell)

Define the managed disk name, storage SKU, and size for this procedure. The procedure reuses the resource group, location, and `$diskAccessId` values that you defined earlier. The [New-AzDiskConfig](/powershell/module/az.compute/new-azdiskconfig) cmdlet creates a disk configuration that uses `AllowPrivate`, and [New-AzDisk](/powershell/module/az.compute/new-azdisk) creates the empty managed disk.

```azurepowershell-interactive
$diskName = "yourDiskName"
$diskSkuName = "Standard_LRS"
$diskSizeGB = 128

$diskConfig = New-AzDiskConfig -Location $location -SkuName $diskSkuName -CreateOption Empty -DiskSizeGB $diskSizeGB -NetworkAccessPolicy AllowPrivate -DiskAccessId $diskAccessId

New-AzDisk -ResourceGroupName $resourceGroupName -DiskName $diskName -Disk $diskConfig
```

---

## Create a snapshot protected with Private Link

# [Azure CLI](#tab/azure-cli)

Use the existing source disk, new snapshot, resource group, region, and disk access resource names that you defined earlier. The [az disk show](/cli/azure/disk#az-disk-show) command gets the source disk ID. Then, [az snapshot create](/cli/azure/snapshot#az-snapshot-create) creates the snapshot, associates it with the disk access resource, and sets its network access policy to `AllowPrivate`.

```azurecli-interactive
# This step reuses variables defined in the first CLI sample.

diskId=$(az disk show -n $sourceDiskName -g $resourceGroupName --query [id] -o tsv)

diskAccessId=$(az resource show -n $diskAccessName -g $resourceGroupName --namespace Microsoft.Compute --resource-type diskAccesses --query [id] -o tsv)

az snapshot create -n $snapshotNameSecuredWithPrivateLink \
    -g $resourceGroupName \
    -l $region \
    --source $diskId \
    --network-access-policy AllowPrivate \
    --disk-access $diskAccessId
```

# [Azure PowerShell](#tab/azure-powershell)

Use the existing source disk, new snapshot, resource group, location, and disk access resource ID that you defined earlier. The [Get-AzDisk](/powershell/module/az.compute/get-azdisk) cmdlet gets the source disk. Then, [New-AzSnapshotConfig](/powershell/module/az.compute/new-azsnapshotconfig) creates a snapshot configuration that uses `AllowPrivate`, and [New-AzSnapshot](/powershell/module/az.compute/new-azsnapshot) creates the snapshot.

```azurepowershell-interactive
$sourceDisk = Get-AzDisk -ResourceGroupName $resourceGroupName -DiskName $sourceDiskName

$snapshotConfig = New-AzSnapshotConfig -Location $location -CreateOption Copy -SourceResourceId $sourceDisk.Id -NetworkAccessPolicy AllowPrivate -DiskAccessId $diskAccessId

New-AzSnapshot -ResourceGroupName $resourceGroupName -SnapshotName $snapshotNameSecuredWithPrivateLink -Snapshot $snapshotConfig
```

---

## Next steps

- To upload a VHD to Azure or copy a managed disk to another region, use the [Azure CLI](disks-upload-vhd-to-managed-disk-cli.md) or the [Azure PowerShell module](../windows/disks-upload-vhd-to-managed-disk-powershell.md).
- To download a VHD, see [Windows](../windows/download-vhd.md) or [Linux](download-vhd.md).
- [FAQ about Private Link for managed disks](/azure/virtual-machines/faq-for-disks#private-links-for-managed-disks).
- Export or copy managed snapshots as VHD to a storage account in a different region with [Azure CLI](/previous-versions/azure/virtual-machines/scripts/virtual-machines-cli-sample-copy-managed-disks-vhd).
