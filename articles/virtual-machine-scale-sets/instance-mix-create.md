---
title: Create a Virtual Machine Scale Set with instance mix
description: How to create a virtual machine scale set using instance mix on different platforms. 
author: brittanyrowe 
ms.author: brittanyrowe
ms.topic: conceptual
ms.service: azure-virtual-machine-scale-sets
ms.date: 1/10/2025
ms.reviewer: jushiman
---

# Deploy a scale set using instance mix
The article walks through how to deploy a scale set using instance mix.

> [!IMPORTANT]
> Instance mix for Virtual Machine Scale Sets with Flexible Orchestration Mode is currently in preview. Previews are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature may change prior to general availability (GA). 

## Prerequisites
Before using instance mix, complete feature registration for the `FlexVMScaleSetSkuProfileEnabled` feature flag using the [az feature register](/cli/azure/feature#az-feature-register) command:

```azurecli-interactive
az feature register --namespace "Microsoft.Compute" --name "FlexVMScaleSetSkuProfileEnabled"
```

It takes a few moments for the feature to register. Verify the registration status by using the [az feature show](/cli/azure/feature#az-feature-register) command:

```azurecli-interactive
az feature show --namespace "Microsoft.Compute" --name "FlexVMScaleSetSkuProfileEnabled"
```

## Create a scale set using instance mix
### [Azure portal](#tab/portal-1)
1. Go to **Virtual machine scale sets**.
2. Select the **Create** button to go to the **Create a virtual machine scale set** view.
3. In the **Basics** tab, fill out the required fields. If the field isn't called out in the next sections, you can set the fields to what works best for your scale set.
4. Ensure that you select a region that instance mix is supported in.
5. Be sure **Orchestration mode** is set to **Flexible**.
6. In the **Size** section, click **Select up to 5 sizes (preview)** and the **Select a VM size** page appears.
7. Use the size picker to select up to five VM sizes. Once you've selected your VM sizes, click the **Select** button at the bottom of the page to return to the scale set Basics tab.
8. In the **Allocation strategy (preview)** field, select your allocation strategy.
9. When using the `Prioritized` allocation strategy, the **Rank size** section appears below the Allocation strategy section. Clicking on the bottom **Rank priority** brings up the prioritization blade, where you can adjust the priority of your VM sizes.
10. You can specify other properties in subsequent tabs, or you can go to **Review + create** and select the **Create** button at the bottom of the page to start your instance mix scale set deployment.

### [Azure CLI](#tab/cli-1)
Before using CLI commands with instance mix, please be sure you're using the correct CLI version. Make sure you're using version `2.66.0` or greater.

You can use the following basic command to create a scale set using instance mix using the following command, which will default to using the `lowestPrice` allocation strategy:
 
```azurecli-interactive
az vmss create \
  --name {myVMSS} \
  --resource-group {myResourceGroup} \
  --image ubuntu2204 \
  --vm-sku Mix \
  --skuprofile-vmsizes Standard_DS1_v2 Standard_D2s_v4
```
 
To specify the allocation strategy, use the `--skuprofile-allocation-strategy` parameter, like the following:
```azurecli-interactive
az vmss create \
  --name {myVMSS} \
  --resource-group {myResourceGroup} \
  --image ubuntu2204 \
  --vm-sku Mix \
  --skuprofile-vmsizes Standard_DS1_v2 Standard_D2s_v4 \
  --skuprofile-allocation-strategy CapacityOptimized
```
 
### [Azure PowerShell](#tab/powershell-1)
You can use the following basic command to create a scale set using instance mix using the following command, which will default to using the `lowestPrice` allocation strategy:
 
```azurepowershell-interactive
New-AzVmss `
  -ResourceGroupName $resourceGroupName `
  -Credential $credentials `
  -VMScaleSetName $vmssName `
  -DomainNameLabel $domainNameLabel1 `
  -VMSize "Mix" `
  -SkuProfileVmSize @("Standard_D4s_v3", "Standard_D4s_v4");
```
 
To specify the allocation strategy, use the `SkuProfileAllocationStrategy` parameter, like the following:
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
To deploy an instance mix scale set through REST API, use a `PUT` call to the scale set:
```json
PUT https://management.azure.com/subscriptions/{YourSubscriptionId}/resourceGroups/{YourResourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{youScaleSetName}?api-version=2023-09-01
```

In the request body, ensure `sku.name` is set to Mix:
```json
  "sku": {
    "name": "Mix",
    "capacity": {TotalNumberVMs}
  },
```
Ensure you reference your existing subnet:
```json
"subnet": {
    "id": "/subscriptions/{YourSubscriptionId}/resourceGroups/{YourResourceGroupName}/providers/Microsoft.Network/virtualNetworks/{YourVnetName}/subnets/default"
},
```
Lastly, be sure to specify the `skuProfile` with **up to five** VM sizes. This sample uses three:
```json
    "skuProfile": {
      "vmSizes": [
        {
          "name": "Standard_D8s_v5"
        },
        {
          "name": "Standard_E16s_v5"
        },
        {
          "name": "Standard_D2s_v5"
        }
      ],
      "allocationStrategy": "lowestPrice"
    },
```

When using the `prioritized` allocation strategy, you can specify the priority ranking of the `vmSizes` specified:
```json
    "skuProfile": {
      "vmSizes": [
        {
          "name": "Standard_D8s_v5", "rank": 1
        },
        {
          "name": "Standard_E16s_v5", "rank": 2
        },
        {
          "name": "Standard_D2s_v5", "rank": 1
        }
      ],
      "allocationStrategy": "Prioritized"
    },
```

---

## Next steps
Learn how to [update](instance-mix-update.md) your instance mix enabled scale set.