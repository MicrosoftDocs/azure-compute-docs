---
title: Create a Virtual Machine Scale Set with instance mix
description: How to create a virtual machine scale set using instance mix on different platforms. 
author: brittanyrowe 
ms.author: brittanyrowe
ms.topic: concept-article
ms.service: azure-virtual-machine-scale-sets
ms.date: 08/19/2025
ms.reviewer: jushiman
# Customer intent: As a cloud administrator, I want to create a virtual machine scale set using instance mix, so that I can optimize resource allocation with different VM sizes based on my application requirements.
---

# Create a scale set using instance mix
This article shows how to create a Virtual Machine Scale Set (VMSS) with instance mix. Instance mix lets you specify multiple virtual machine (VM) sizes for a single scale set and control how Azure chooses sizes at provisioning time using an allocation strategy.

## Before you begin

Confirm these prerequisites before you create an instance mix enabled scale set:

- You intend to deploy a scale set that uses Flexible orchestration mode.
- Consistent VM characteristics across selected sizes: same CPU architecture (x64 or Arm64), compatible disk interface (SCSI or NVMe), and compatible security profile.
- Sufficient quota for each VM size in the target subscription and region.
- Choose a region that supports the VM sizes you want to include.
- (CLI users) Azure CLI 2.66.0 or later is recommended. For PowerShell, use the latest `Az.Compute` module.


## Create a scale set using instance mix
### [Azure portal](#tab/portal-1)
1. Go to **Virtual machine scale sets**.
2. Select the **Create** button to go to the **Create a virtual machine scale set** view.
3. In the **Basics** tab, fill out the required fields. If the field isn't called out in the next sections, you can set the fields to what works best for your scale set.
4. Ensure that you select a region that instance mix is supported in.
5. Be sure **Orchestration mode** is set to **Flexible**.
6. In the **Size** section, click **Select up to 5 sizes** and the **Select a VM size** page appears.
7. Use the size picker to select up to five VM sizes. Once you select your VM sizes, click the **Select** button at the bottom of the page to return to the scale set Basics tab.
8. In the **Allocation strategy** field, select your allocation strategy.
9. If you use the `Prioritized (preview)` allocation strategy, the **Rank size** section appears below the Allocation strategy section. Select **Rank priority** to open the prioritization blade, where you can adjust the priority of your VM sizes.
10. You can specify other properties in subsequent tabs, or you can go to **Review + create** and select the **Create** button at the bottom of the page to start your instance mix scale set deployment.

### [Azure CLI](#tab/cli-1)
Before using CLI commands with instance mix, be sure you're using the correct CLI version. Make sure you're using version `2.66.0` or greater.

You can use the following basic command to create a scale set using instance mix, which defaults to using the `lowestPrice` allocation strategy:
 
```azurecli-interactive
az vmss create \
  --name {myVMSS} \
  --resource-group {myResourceGroup} \
  --image ubuntu2204 \
  --vm-sku Mix \
  --skuprofile-vmsizes Standard_D2s_v5 Standard_D2as_v5 \
  --authentication-type ssh \
  --generate-ssh-keys
```
 
To specify the allocation strategy, use the `--skuprofile-allocation-strategy` parameter, like the following command:
```azurecli-interactive
az vmss create \
  --name {myVMSS} \
  --resource-group {myResourceGroup} \
  --image ubuntu2204 \
  --vm-sku Mix \
  --skuprofile-vmsizes Standard_D2s_v5 Standard_D2as_v5 \
  --skuprofile-allocation-strategy CapacityOptimized \
  --authentication-type ssh \
  --generate-ssh-keys
```
 
### [Azure PowerShell](#tab/powershell-1)
You can use the following basic command to create a scale set using instance mix using the following command, which defaults to using the `lowestPrice` allocation strategy:
 
```azurepowershell-interactive
New-AzVmss `
  -ResourceGroupName $resourceGroupName `
  -Credential $credentials `
  -VMScaleSetName $vmssName `
  -DomainNameLabel $domainNameLabel1 `
  -VMSize "Mix" `
  -SkuProfileVmSize @("Standard_D4s_v3", "Standard_D4s_v4");
```
 
To specify the allocation strategy, use the `SkuProfileAllocationStrategy` parameter, like the following command:
```azurepowershell-interactive
New-AzVmss `
-ResourceGroupName $resourceGroupName `
-Credential $credentials `
-VMScaleSetName $vmssName `
-DomainNameLabel $domainNameLabel1 `
-SkuProfileVmSize @("Standard_D4s_v3", "Standard_D4s_v4") `
-SkuProfileAllocationStrategy "CapacityOptimized";
```
 
To create a scale set using a scale set configuration object utilizing instance mix, use the following command:
```azurepowershell-interactive
$vmss = New-AzVmssConfig -Location $loc -SkuCapacity 2 -UpgradePolicyMode 'Manual' -EncryptionAtHost -SecurityType $stnd -SkuProfileVmSize @("Standard_D4s_v3", "Standard_D4s_v4") -SkuProfileAllocationStrategy "CapacityOptimized"`
            | Add-AzVmssNetworkInterfaceConfiguration -Name 'test' -Primary $true -IPConfiguration $ipCfg `
            | Set-AzVmssOSProfile -ComputerNamePrefix 'test' -AdminUsername $adminUsername -AdminPassword $adminPassword `
            | Set-AzVmssStorageProfile -OsDiskCreateOption 'FromImage' -OsDiskCaching 'None' `
            -ImageReferenceOffer $imgRef.Offer -ImageReferenceSku $imgRef.Skus -ImageReferenceVersion 'latest' `
            -ImageReferencePublisher $imgRef.PublisherName;
 
$vmssResult = New-AzVmss -ResourceGroupName $resourceGroupName -Name $vmssName -VirtualMachineScaleSet $vmss
```

### [REST API](#tab/arm-1)
To deploy a scale set with instance mix using the REST API, make a `PUT` request to the following endpoint:

```http
PUT https://management.azure.com/subscriptions/{YourSubscriptionId}/resourceGroups/{YourResourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{yourScaleSetName}?api-version=2023-09-01
```

In the request body, set `sku.name` to `Mix` and specify the total number of VMs:

```json
"sku": {
  "name": "Mix",
  "capacity": {TotalNumberVMs}
},
```

Reference your existing subnet, as follows:

```json
"subnet": {
  "id": "/subscriptions/{YourSubscriptionId}/resourceGroups/{YourResourceGroupName}/providers/Microsoft.Network/virtualNetworks/{YourVnetName}/subnets/default"
},
```

Specify the `skuProfile` with up to five VM sizes. The following example uses three sizes and the `lowestPrice` allocation strategy:

```json
"skuProfile": {
  "vmSizes": [
    { "name": "Standard_D8s_v5"},
    { "name": "Standard_D8as_v5"},
    { "name": "Standard_D8s_v4"}
  ],
  "allocationStrategy": "lowestPrice"
},
```

If you use the `Prioritized (preview)` allocation strategy, you can assign a priority ranking to each VM size. For example:

```json
"skuProfile": {
  "vmSizes": [
    { "name": "Standard_D8s_v5", "rank": 1 },
    { "name": "Standard_D8as_v5", "rank": 2 },
    { "name": "Standard_D8s_v4", "rank": 3 }
  ],
  "allocationStrategy": "Prioritized"
},
```

- Replace placeholders, such as `{YourSubscriptionId}`, with your actual values.
- You can specify up to five VM sizes in the `vmSizes` array.
- The `rank` property is required only when using the `Prioritized (preview)` allocation strategy.

Tips for REST deployments:

- Ensure `sku.name` is set to `"Mix"` and that `sku.tier` isn't set (or is `null`).
- The `rank` property is only required for the `Prioritized` strategy. Ranks with lower numbers are higher priority.
- To confirm VM size availability and quota before deployment, validate the template against the target subscription and region.

---

## Next steps
Learn how to [view your scale set's instance mix configurations](instance-mix-view.md).