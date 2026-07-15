---
title: Restrict managed disk import and export with Private Links
description: Use Azure CLI or Azure PowerShell to configure private links for managed disks and restrict import and export access to your virtual network.
author: roygara
ms.service: azure-disk-storage
ms.topic: how-to
ms.date: 07/09/2026
ms.author: rogarana
ms.custom: references_regions, devx-track-azurecli, devx-track-azurepowershell, linux-related-content, windows-related-content
ai-usage: ai-assisted
# Customer intent: "As a cloud administrator, I want to implement Private Links for managed disks by using Azure CLI or Azure PowerShell, so that I can securely control import and export access within my virtual network and enhance data security."
---

# Restrict managed disk import and export with Private Links

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets

Use [private endpoints](/azure/private-link/private-endpoint-overview) to restrict managed disk import and export. Securely access data over [Private Link](/azure/private-link/private-link-overview) from clients in your Azure virtual network. The private endpoint uses an IP address from your virtual network address space for your managed disks service. Traffic between clients in your virtual network and managed disks stays on the virtual network and a private link on the Microsoft backbone network, which reduces exposure to the public internet.

To use Private Links for managed disk import or export, create a disk access resource and link it to a virtual network in the same subscription by creating a private endpoint. Then associate a disk or snapshot with a disk access resource. Finally, set the network access policy of the disk or snapshot to `AllowPrivate` to limit access to your virtual network.

Set the network access policy to `DenyAll` to prevent anyone from exporting data from a disk or snapshot. The default network access policy is `AllowAll`.

## Prerequisites

Before you begin, install the latest [Azure CLI](/cli/azure/install-azure-cli) or [Azure PowerShell module](/powershell/azure/install-azure-powershell).

## Limitations

[!INCLUDE [virtual-machines-disks-private-links-limitations](../includes/virtual-machines-disks-private-links-limitations.md)]


## Sign in and set variables

Choose a tab to use Azure CLI or Azure PowerShell.

# [Azure CLI](#tab/azure-cli)

```azurecli-interactive
subscriptionId=yourSubscriptionId
resourceGroupName=yourResourceGroupName
region=northcentralus
diskAccessName=yourDiskAccessForPrivateLinks
vnetName=yourVNETForPrivateLinks
subnetName=yourSubnetForPrivateLinks
privateEndPointName=yourPrivateLinkForSecureMDExportImport
privateEndPointConnectionName=yourPrivateLinkConnection
privateDnsZoneName=privatelink.blob.core.windows.net
privateDnsZoneLinkName=yourDNSLink
privateDnsZoneGroupName=yourZoneGroup

# The name of an existing disk that is the source of the snapshot.
sourceDiskName=yourSourceDiskForSnapshot

# The name of the new snapshot secured with Private Links.
snapshotNameSecuredWithPL=yourSnapshotNameSecuredWithPL

az login
az account set --subscription $subscriptionId
```

# [Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
$subscriptionId = "yourSubscriptionId"
$resourceGroupName = "yourResourceGroupName"
$location = "NorthCentralUS"
$diskAccessName = "yourDiskAccessForPrivateLinks"
$vnetName = "yourVnetForPrivateLinks"
$subnetName = "yourSubnetForPrivateLinks"
$privateEndpointName = "yourPrivateEndpointForSecureMDExportImport"
$privateEndpointConnectionName = "yourPrivateEndpointConnection"
$privateDnsZoneName = "privatelink.blob.core.windows.net"
$privateDnsZoneLinkName = "yourDnsLink"
$privateDnsZoneGroupName = "yourZoneGroup"

# The name of an existing disk that is the source of the snapshot.
$sourceDiskName = "yourSourceDiskForSnapshot"

# The name of the new snapshot secured with Private Links.
$snapshotNameSecuredWithPL = "yourSnapshotNameSecuredWithPL"

Connect-AzAccount
Set-AzContext -Subscription $subscriptionId
```

---

## Create a disk access resource

# [Azure CLI](#tab/azure-cli)

```azurecli
az disk-access create -n $diskAccessName -g $resourceGroupName -l $region

diskAccessId=$(az disk-access show -n $diskAccessName -g $resourceGroupName --query [id] -o tsv)
```

# [Azure PowerShell](#tab/azure-powershell)

```azurepowershell
$diskAccess = New-AzDiskAccess -ResourceGroupName $resourceGroupName -Name $diskAccessName -Location $location
$diskAccessId = $diskAccess.Id
```

---

## Create a virtual network

Network policies such as network security groups (NSGs) aren't supported for private endpoints. To deploy a private endpoint on a subnet, disable private endpoint network policies on that subnet.

# [Azure CLI](#tab/azure-cli)

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

```azurepowershell
$subnetConfig = New-AzVirtualNetworkSubnetConfig -Name $subnetName -AddressPrefix "10.0.0.0/24" -PrivateEndpointNetworkPoliciesFlag "Disabled"

$vnet = New-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name $vnetName -Location $location -AddressPrefix "10.0.0.0/16" -Subnet $subnetConfig
```

---

## Create a private endpoint for the disk access resource

# [Azure CLI](#tab/azure-cli)

```azurecli
az network private-endpoint create --resource-group $resourceGroupName \
    --name $privateEndPointName \
    --vnet-name $vnetName \
    --subnet $subnetName \
    --private-connection-resource-id $diskAccessId \
    --group-ids disks \
    --connection-name $privateEndPointConnectionName
```

# [Azure PowerShell](#tab/azure-powershell)

```azurepowershell
$subnet = Get-AzVirtualNetworkSubnetConfig -Name $subnetName -VirtualNetwork $vnet

$privateLinkServiceConnection = New-AzPrivateLinkServiceConnection -Name $privateEndpointConnectionName -PrivateLinkServiceId $diskAccessId -GroupId "disks"

$privateEndpoint = New-AzPrivateEndpoint -ResourceGroupName $resourceGroupName -Name $privateEndpointName -Location $location -Subnet $subnet -PrivateLinkServiceConnection $privateLinkServiceConnection
```

---

## Configure the private DNS zone

Create a private DNS zone for the storage blob domain, create a virtual network link, and then create a DNS zone group that associates the private endpoint with the private DNS zone.

# [Azure CLI](#tab/azure-cli)

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
   --endpoint-name $privateEndPointName \
   --name $privateDnsZoneGroupName \
   --private-dns-zone $privateDnsZoneName \
   --zone-name disks
```

# [Azure PowerShell](#tab/azure-powershell)

```azurepowershell
$privateDnsZone = New-AzPrivateDnsZone -ResourceGroupName $resourceGroupName -Name $privateDnsZoneName

New-AzPrivateDnsVirtualNetworkLink -ResourceGroupName $resourceGroupName -ZoneName $privateDnsZoneName -Name $privateDnsZoneLinkName -VirtualNetworkId $vnet.Id

$privateDnsZoneConfig = New-AzPrivateDnsZoneConfig -Name $privateDnsZoneName -PrivateDnsZoneId $privateDnsZone.ResourceId

New-AzPrivateDnsZoneGroup -ResourceGroupName $resourceGroupName -PrivateEndpointName $privateEndpointName -Name $privateDnsZoneGroupName -PrivateDnsZoneConfig $privateDnsZoneConfig
```

---

## Create a managed disk protected with Private Links

# [Azure CLI](#tab/azure-cli)

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

```azurepowershell-interactive
$diskName = "yourDiskName"
$diskSkuName = "Standard_LRS"
$diskSizeGB = 128

$diskConfig = New-AzDiskConfig -Location $location -SkuName $diskSkuName -CreateOption Empty -DiskSizeGB $diskSizeGB -NetworkAccessPolicy AllowPrivate -DiskAccessId $diskAccessId

New-AzDisk -ResourceGroupName $resourceGroupName -DiskName $diskName -Disk $diskConfig
```

---

## Create a snapshot protected with Private Links

# [Azure CLI](#tab/azure-cli)

```azurecli-interactive
# This step reuses variables defined in the first CLI sample.

diskId=$(az disk show -n $sourceDiskName -g $resourceGroupName --query [id] -o tsv)

diskAccessId=$(az resource show -n $diskAccessName -g $resourceGroupName --namespace Microsoft.Compute --resource-type diskAccesses --query [id] -o tsv)

az snapshot create -n $snapshotNameSecuredWithPL \
    -g $resourceGroupName \
    -l $region \
    --source $diskId \
    --network-access-policy AllowPrivate \
    --disk-access $diskAccessId
```

# [Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
$sourceDisk = Get-AzDisk -ResourceGroupName $resourceGroupName -DiskName $sourceDiskName

$snapshotConfig = New-AzSnapshotConfig -Location $location -CreateOption Copy -SourceResourceId $sourceDisk.Id -NetworkAccessPolicy AllowPrivate -DiskAccessId $diskAccessId

New-AzSnapshot -ResourceGroupName $resourceGroupName -SnapshotName $snapshotNameSecuredWithPL -Snapshot $snapshotConfig
```

---

## Next steps

- To upload a VHD to Azure or copy a managed disk to another region, use the [Azure CLI](disks-upload-vhd-to-managed-disk-cli.md) or the [Azure PowerShell module](../windows/disks-upload-vhd-to-managed-disk-powershell.md).
- To download a VHD, see [Windows](../windows/download-vhd.md) or [Linux](download-vhd.md).
- [FAQ on Private Links](/azure/virtual-machines/faq-for-disks#private-links-for-managed-disks)
- Export or copy managed snapshots as VHD to a storage account in a different region with [Azure CLI](/previous-versions/azure/virtual-machines/scripts/virtual-machines-cli-sample-copy-managed-disks-vhd).
