---
title: Enable Trusted launch on existing Uniform scale set
description: Enable Trusted launch on existing Uniform scale set
author: AjKundnani
ms.author: ajkundna
ms.reviewer: cynthn
ms.service: azure-virtual-machine-scale-sets
ms.subservice: trusted-launch
ms.topic: how-to
ms.date: 06/18/2025
ms.custom: template-how-to, devx-track-azurepowershell, devx-track-arm-template
# Customer intent: "As a cloud administrator, I want to enable Trusted launch on existing Uniform scale sets, so that I can enhance the security of virtual machines against advanced threats like boot kits and rootkits."
---

# Enable Trusted launch on existing Uniform scale set

**Applies to:** :heavy_check_mark: Uniform scale set :heavy_check_mark: Flex scale set :x: Service fabric

Azure Virtual machine Scale sets supports enabling Trusted launch on existing [Uniform Scale sets](../virtual-machine-scale-sets/overview.md) virtual machine (VM) by upgrading to [Trusted launch](trusted-launch.md) security type.

[Trusted launch](trusted-launch.md) enables foundational compute security on [Azure Generation 2](generation-2.md) virtual machines & scale sets and protects them against advanced and persistent attack techniques like boot kits and rootkits. It does so by combining infrastructure technologies like Secure Boot, vTPM, and Boot Integrity Monitoring on your Scale set.

## Limitations

- Enabling Trusted launch on existing [virtual machine Scale sets with data disks attached](../virtual-machine-scale-sets/virtual-machine-scale-sets-attached-disks.md) requires upgrade mode set to [Rolling upgrade with max surge](../virtual-machine-scale-sets/virtual-machine-scale-sets-maxsurge.md)
  - To validate if scale set is configured with data disk, navigate to scale set -> **Disks** under **Settings** menu -> check under heading **Data disks**
    :::image type="content" source="./media/trusted-launch/virtual-machine-scale-sets-data-disks.png" alt-text="Screenshot of the scale set with data disks." lightbox="./media/trusted-launch/virtual-machine-scale-sets-data-disks.png":::

- Enabling Trusted launch on existing [virtual machine Scale sets Flex](../virtual-machine-scale-sets/virtual-machine-scale-sets-orchestration-modes.md) is currently in preview. [Register for preview of enabling Trusted launch on existing Flex scale set preview](https://aka.ms/TrustedLaunchUpgrade/FlexPreview)
- Enabling Trusted launch on existing [Service fabric clusters](../service-fabric/service-fabric-overview.md) and [Service fabric managed clusters](../service-fabric/overview-managed-cluster.md) is currently not supported.

## Prerequisites

- Scale set isn't dependent on [features currently not supported with Trusted launch](trusted-launch.md#unsupported-features).
- Scale set should be configured with [Trusted launch supported size family](trusted-launch.md#virtual-machines-sizes)
    > [!NOTE]
    >
    > - Virtual machine size can be changed along with Trusted launch upgrade. Ensure quota for new VM Size is in-place to avoid upgrade failures. Refer to [Check vCPU quotas](quotas.md).
    > - Change to Virtual machine size re-creates the Virtual machine instance with new size and requires downtime of individual Virtual machine instance. It can be done in a Rolling Upgrade fashion to avoid Scale set downtime.
- Scale set should be configured with [Trusted launch supported OS Image](trusted-launch.md#operating-systems-supported). For [Azure compute gallery OS image](azure-compute-gallery.md), ensure image definition is marked as [TrustedLaunchSupported](trusted-launch-portal.md#deploy-a-trusted-launch-vm-from-an-azure-compute-gallery-image)

## Enable Trusted launch on existing Scale set Uniform

### [Portal](#tab/portal)

Following steps details how to enable Trusted launch on existing uniform scale set using Azure portal.

1. (Optional) **Scale set Size**: Navigate to `Size` under `Availability + scale` -> Modify the Scale set size if current size family isn't [supported with Trusted launch](trusted-launch.md#virtual-machines-sizes) security configuration -> Click **Apply**.
    :::image type="content" source="./media/trusted-launch/virtual-machine-scale-sets-portal-size-change.png" alt-text="Screenshot of the scale set size change." lightbox="./media/trusted-launch/virtual-machine-scale-sets-portal-size-change.png":::

2. **OS Image**: Navigate to `Operating system` under `Settings` -> Click on `Change image reference`.
    :::image type="content" source="./media/trusted-launch/virtual-machine-scale-sets-portal-os-change.png" alt-text="Screenshot of the scale set OS image change." lightbox="./media/trusted-launch/virtual-machine-scale-sets-portal-os-change.png":::

3. Update the OS Image reference to Gen2-Trusted launch supported OS image. Make sure the source Gen2 image has `TrustedLaunchSupported` security type if using Azure Compute Gallery OS image  -> Click **Apply**.
    :::image type="content" source="./media/trusted-launch/virtual-machine-scale-sets-portal-os-change-01.png" alt-text="Screenshot of the OS image change options." lightbox="./media/trusted-launch/virtual-machine-scale-sets-portal-os-change-01.png":::

4. **Security type**: Click on **Standard** `Security type` on `Overview` page of scale set OR navigate to `Configuration` under `Settings`.

    :::image type="content" source="./media/trusted-launch/virtual-machine-scale-sets-portal-click-security-type.png" alt-text="Screenshot of the overview page." lightbox="./media/trusted-launch/virtual-machine-scale-sets-portal-click-security-type.png":::

5. Update the security type drop-down on `Configuration` page from `Standard` to `Trusted launch` with `Enable secure boot` and `Enable vTPM` checked to enable Trusted Launch security configuration. Click `Yes` to confirm changes.

    > [!NOTE]
    >
    > - **vTPM** is enabled by default.
    > - **Secure Boot** should be enabled (not enabled by default) if you aren't using custom unsigned kernel or drivers. Secure Boot preserves boot integrity and enables foundational security for VM.

    :::image type="content" source="./media/trusted-launch/virtual-machine-scale-sets-portal-apply-security-type.png" alt-text="Screenshot of the Trusted launch security type drop-down." lightbox="./media/trusted-launch/virtual-machine-scale-sets-portal-apply-security-type.png":::

6. Validate the changes on the `Overview` page of scale set.
    :::image type="content" source="./media/trusted-launch/virtual-machine-scale-sets-portal-validate-security-type.png" alt-text="Screenshot of the validation on overview page." lightbox="./media/trusted-launch/virtual-machine-scale-sets-portal-validate-security-type.png":::

7. (Recommended) **Guest Attestation Extension**: Add [Guest Attestation (GA) extension](trusted-launch.md#microsoft-defender-for-cloud-integration) for Scale set resource, which enables [Boot integrity monitoring](boot-integrity-monitoring-overview.md) for Scale set.

8. Update the VM instances manually if Scale set uniform [upgrade mode](../virtual-machine-scale-sets/virtual-machine-scale-sets-upgrade-policy.md) is set to `Manual`.
    :::image type="content" source="./media/trusted-launch/virtual-machine-scale-sets-portal-update-instances.png" alt-text="Screenshot of the scale set instance update." lightbox="./media/trusted-launch/virtual-machine-scale-sets-portal-update-instances.png":::

### [Template](#tab/template)

Following steps details using an [ARM template](/azure/azure-resource-manager/templates/overview) to enable Trusted launch on existing uniform scale set.

Make the following modifications to enable Trusted launch using existing ARM template deployment code. For complete template, refer to [Quickstart Trusted launch Scale set ARM template](https://github.com/Azure/azure-quickstart-templates/blob/master/quickstarts/microsoft.compute/vmss-trustedlaunch-windows/azuredeploy.json).

> [!IMPORTANT]
>
> Trusted launch security type is available with Scale set `apiVersion` `2020-12-01` or higher. Ensure API version is set correctly before upgrade.

1. **OS Image**: Update the OS Image reference to Gen2-Trusted launch supported OS image. Make sure the source Gen2 image has `TrustedLaunchSupported` security type if using Azure Compute Gallery OS image.

    ```json
    "storageProfile": { 
            "osDisk": { 
                "createOption": "FromImage", 
                "caching": "ReadWrite" 
            }, 
            "imageReference": { 
                "publisher": "MicrosoftWindowsServer", 
                "offer": "WindowsServer", 
                "sku": "2022-datacenter-azure-edition", 
                "version": "latest" 
            } 
    }
    ```

2. (Optional) **Scale set Size**: Modify the Scale set size if current size family isn't [supported with Trusted launch](trusted-launch.md#virtual-machines-sizes) security configuration.

    ```json
        "sku": { 
            "name": "Standard_D2s_v3", 
            "tier": "Standard", 
            "capacity": "[parameters('instanceCount')]" 
        } 
    ```

3. **Security Profile**: Add `securityProfile` block under `virtualMachineProfile` to enable Trusted Launch security configuration.
    > [!NOTE]
    >
    > Recommended settings: `vTPM`: `true` and `secureBoot`: `true`
    > `secureBoot` should be set to `false` if you're using any unsigned custom driver or kernel on OS.

    ```json
    "securityProfile": { 
        "securityType": "TrustedLaunch", 
        "uefiSettings": { 
          "secureBootEnabled": true, 
          "vTpmEnabled": true
        } 
    }
    ```

4. (Recommended) **Guest Attestation Extension**: Add [Guest Attestation (GA) extension](trusted-launch.md#microsoft-defender-for-cloud-integration) for Scale set resource, which enables [Boot integrity monitoring](boot-integrity-monitoring-overview.md) for Scale set.
    > [!Important]
    >
    > Guest attestation extension requires `secureBoot` and `vTPM` set to `true`.

    ```json
    { 
        "condition": "[and(parameters('vTPM'), parameters('secureBoot'))]", 
        "type": "Microsoft.Compute/virtualMachineScaleSets/extensions", 
        "apiVersion": "2022-03-01", 
        "name": "[format('{0}/{1}', parameters('vmssName'), GuestAttestation)]", 
        "location": "[parameters('location')]", 
        "properties": { 
          "publisher": "Microsoft.Azure.Security.WindowsAttestation", 
          "type": "GuestAttestation", 
          "typeHandlerVersion": "1.0", 
          "autoUpgradeMinorVersion": true, 
          "enableAutomaticUpgrade": true, 
          "settings": { 
            "AttestationConfig": { 
              "MaaSettings": { 
                "maaEndpoint": "[substring('emptystring', 0, 0)]", 
                "maaTenantName": "GuestAttestation" 
              } 
            } 
          } 
        }, 
        "dependsOn": [ 
          "[resourceId('Microsoft.Compute/virtualMachineScaleSets', parameters('vmssName'))]" 
        ] 
    } 
    ```

    Name of extension publisher:

    OS Type    |    Extension publisher name
    -|-
    Windows    |    Microsoft.Azure.Security.WindowsAttestation
    Linux    |    Microsoft.Azure.Security.LinuxAttestation

5. Review the changes made to template.

    [!INCLUDE [json](./includes/enable-trusted-launch-vmss-arm-template.md)]

6. Execute the ARM template deployment.

    ```azurepowershell-interactive
    $resourceGroupName = "myResourceGroup"
    $parameterFile = "folderPathToFile\parameters.json"
    $templateFile = "folderPathToFile\template.json"
    
    New-AzResourceGroupDeployment `
        -ResourceGroupName $resourceGroupName `
        -TemplateFile $templateFile -TemplateParameterFile $parameterFile
    ```

7. Verify that the deployment is successful. Check for the security type and UEFI settings of the Scale set uniform using Azure portal. Check the Security type section in the Overview page.

    :::image type="content" source="./media/trusted-launch/virtual-machine-scale-sets-portal-validate-security-type.png" alt-text="Screenshot of the validation on overview page." lightbox="./media/trusted-launch/virtual-machine-scale-sets-portal-validate-security-type.png":::

8. Update the VM instances manually if Scale set uniform [upgrade mode](../virtual-machine-scale-sets/virtual-machine-scale-sets-upgrade-policy.md) is set to `Manual`.

    ```azurepowershell-interactive
    $resourceGroupName = "myResourceGroup"
    $vmssName = "VMScaleSet001"
    Update-AzVmssInstance -ResourceGroupName $resourceGroupName -VMScaleSetName $vmssName -InstanceId "0"
    ```

### [CLI](#tab/cli)

This section steps through using the Azure CLI to enable Trusted launch on existing Azure Scale set Uniform resource.

Make sure latest version of [Azure CLI](/cli/azure/install-az-cli2) is installed and logged in to an Azure account with [az login](/cli/azure/reference-index).

> [!NOTE]
>
> - **vTPM** is enabled by default.
> - **Secure Boot** should be enabled (not enabled by default) if you aren't using custom unsigned kernel or drivers. Secure Boot preserves boot integrity and enables foundational security for VM.

1. Log in to Azure Subscription

    ```azurecli-interactive
    az login
    
    az account set --subscription 00000000-0000-0000-0000-000000000000
    ```

2. Enable Trusted launch using command [az vmss update](/cli/azure/reference-index) by setting `--security-type`: `TrustedLaunch`, `virtualMachineProfile.storageProfile.imageReference.sku`: Gen2-Trusted launch supported OS image, `--vm-sku`: Gen2-Trusted launch supported VM size.

    ```azurecli-interactive
    az vmss update --name MyScaleSet `
        --resource-group MyResourceGroup `
        --set virtualMachineProfile.storageProfile.imageReference.sku='2022-datacenter-azure-edition' `
        --security-type TrustedLaunch --enable-secure-boot $true --enable-vtpm $true
    ```

    > [!NOTE]
    >
    > OS Image SKU used in the `az vmss update` should be from same OS Image Publisher and Offer.

3. Validate output of previous command. `securityProfile` configuration is returned with command output.

    ```json
    {
      "securityProfile": {
        "securityType": "TrustedLaunch",
        "uefiSettings": {
          "secureBootEnabled": true,
          "vTpmEnabled": true
        }
      }
    }
    ```

4. Update the VM instances manually if Scale set uniform [upgrade mode](../virtual-machine-scale-sets/virtual-machine-scale-sets-upgrade-policy.md) is set to `Manual`.

    ```azurecli-interactive
    az vmss update-instances --instance-ids 1 --name MyScaleSet --resource-group MyResourceGroup
    ```

5. Start rolling upgrade if Scale set uniform [upgrade mode](../virtual-machine-scale-sets/virtual-machine-scale-sets-upgrade-policy.md) is set to `RollingUpgrade`.

    ```azurecli-interactive
    az vmss rolling-upgrade start --name MyScaleSet --resource-group MyResourceGroup
    ```

### [PowerShell](#tab/powershell)

This section steps through using the Azure PowerShell to enable Trusted launch on existing Azure Virtual machine Scale set Uniform.

Make sure the latest [Azure PowerShell](/powershell/azure/install-azps-windows) is installed and logged in to an Azure account with [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount).

> [!NOTE]
>
> - **vTPM** is enabled by default.
> - **Secure Boot** should be enabled (not enabled by default) if you aren't using custom unsigned kernel or drivers. Secure Boot preserves boot integrity and enables foundational security for VM.

1. Log in to Azure Subscription

    ```azurepowershell-interactive
    Connect-AzAccount -SubscriptionId 00000000-0000-0000-0000-000000000000
    ```

2. Enable Trusted launch using command [Update-AzVMSS](/powershell/module/az.compute/update-azvmss) by setting `-SecurityType`: `TrustedLaunch`, `ImageReferenceSku`: Gen2-Trusted launch supported OS image, `-SkuName`: Gen2-Trusted launch supported VM size.

    ```azurepowershell-interactive
    $vmss = Get-AzVmss -VMScaleSetName MyVmssName -ResourceGroupName MyResourceGroup

    # Enable Trusted Launch
    Set-AzVmssSecurityProfile -virtualMachineScaleSet $vmss -SecurityType TrustedLaunch

    # Enable Trusted Launch settings
    Set-AzVmssUefi -VirtualMachineScaleSet $vmss -EnableVtpm $true -EnableSecureBoot $true

    Update-AzVmss -ResourceGroupName $vmss.ResourceGroupName `
        -VMScaleSetName $vmss.Name -VirtualMachineScaleSet $vmss `
        -SecurityType TrustedLaunch -EnableVtpm $true -EnableSecureBoot $true `
        -ImageReferenceSku 2022-datacenter-azure-edition
    ```

    > [!NOTE]
    >
    > OS Image SKU used in Update-AzVMSS command should be from same OS Image Publisher and Offer.

3. Validate `securityProfile` configuration is returned with [Get-AzVMSS](/powershell/module/az.compute/get-azvmss) command output.

    ```json
    {
      "securityProfile": {
        "securityType": "TrustedLaunch",
        "uefiSettings": {
          "secureBootEnabled": true,
          "vTpmEnabled": true
        }
      }
    }
    ```

4. Update the VM instances manually if Scale set uniform [upgrade mode](../virtual-machine-scale-sets/virtual-machine-scale-sets-upgrade-policy.md) is set to `Manual`.

    ```azurepowershell-interactive
    Update-AzVmssInstance -ResourceGroupName "MyResourceGroup" -VMScaleSetName "MyScaleSet" -InstanceId "0"
    ```

5. Start rolling upgrade if Scale set uniform [upgrade mode](../virtual-machine-scale-sets/virtual-machine-scale-sets-upgrade-policy.md) is set to `RollingUpgrade`.

    ```azurepowershell-interactive
    Start-AzVmssRollingOSUpgrade -ResourceGroupName "MyResourceGroup" -VMScaleSetName "MyScaleSet"
    ```

---

## Roll-back

To roll-back changes from Trusted launch to previous known good configuration, you need to set `securityType` of Scale set to **Standard**.

### [Portal](#tab/portal)

1. **OS Image**: Navigate to `Operating system` under `Settings`. Click on `Change image reference`.
    :::image type="content" source="./media/trusted-launch/virtual-machine-scale-sets-portal-os-change.png" alt-text="Screenshot of the scale set OS image change." lightbox="./media/trusted-launch/virtual-machine-scale-sets-portal-os-change.png":::

2. Update the OS Image reference to last known good configuration  -> Click **Apply**.
    :::image type="content" source="./media/trusted-launch/virtual-machine-scale-sets-portal-os-change-01.png" alt-text="Screenshot of the OS image change options." lightbox="./media/trusted-launch/virtual-machine-scale-sets-portal-os-change-01.png":::

3. **Security type**: Navigate to `Configuration` page under `Settings` -> Update the security type drop-down on `Configuration` page from `Trusted launch` to `Standard` for disabling Trusted Launch security configuration. Click `Yes` to confirm changes.
    :::image type="content" source="./media/trusted-launch/virtual-machine-scale-sets-portal-rollback.png" alt-text="Screenshot of the Standard security type drop-down." lightbox="./media/trusted-launch/virtual-machine-scale-sets-portal-rollback.png":::

4. Validate the changes on the `Overview` page of scale set.
    :::image type="content" source="./media/trusted-launch/virtual-machine-scale-sets-portal-rollback-01.png" alt-text="Screenshot of the validation of rollback on overview page." lightbox="./media/trusted-launch/virtual-machine-scale-sets-portal-rollback-01.png":::

5. Update the VM instances manually if Scale set uniform [upgrade mode](../virtual-machine-scale-sets/virtual-machine-scale-sets-upgrade-policy.md) is set to `Manual`.
    :::image type="content" source="./media/trusted-launch/virtual-machine-scale-sets-portal-update-instances.png" alt-text="Screenshot of the scale set instance update." lightbox="./media/trusted-launch/virtual-machine-scale-sets-portal-update-instances.png":::

### [Template](#tab/template)

To roll-back changes from Trusted launch to previous known good configuration, set `securityProfile` to **Standard** as shown. Optionally, you can also revert other parameter changes - OS image, VM size, and repeat steps 5-8 described with [Enable Trusted launch on existing scale set](#enable-trusted-launch-on-existing-scale-set-uniform)

```json
"securityProfile": {
    "securityType": "Standard",
    "uefiSettings": "[null()]"
}
```

### [CLI](#tab/cli)

> [!NOTE]
>
> Required Azure CLI version **2.62.0** or above for roll-back of uniform scale set from Trusted launch to Non-Trusted launch configuration.

To roll-back changes from Trusted launch to previous known good configuration, set `--security-type` to `Standard` as shown. Optionally, you can also revert other parameter changes - OS image, virtual machine size, and repeat steps 2-5 described with [Enable Trusted launch on existing scale set](#enable-trusted-launch-on-existing-scale-set-uniform)

```azurecli-interactive
az vmss update --name MyScaleSet `
        --resource-group MyResourceGroup `
        --security-type Standard
```

### [PowerShell](#tab/powershell)

To roll-back changes from Trusted launch to previous known good configuration, set `-SecurityType` to `Standard` as shown. Optionally, you can also revert other parameter changes - OS image, virtual machine size, and repeat steps 2-5 described with [Enable Trusted launch on existing scale set](#enable-trusted-launch-on-existing-scale-set-uniform)

```azurepowershell-interactive
$vmss = Get-AzVmss -VMScaleSetName MyVmssName -ResourceGroupName MyResourceGroup

# Roll-back Trusted Launch
Set-AzVmssSecurityProfile -virtualMachineScaleSet $vmss -SecurityType Standard

Update-AzVmss -ResourceGroupName $vmss.ResourceGroupName `
    -VMScaleSetName $vmss.Name -VirtualMachineScaleSet $vmss `
    -SecurityType Standard `
    -ImageReferenceSku 2022-datacenter-azure-edition
```

---

## Next steps

**(Recommended)** Post-Upgrades enable [Boot integrity monitoring](trusted-launch.md#microsoft-defender-for-cloud-integration) to monitor the health of the VM using Microsoft Defender for Cloud.

Learn more about [Trusted launch](trusted-launch.md) and review [frequently asked questions](trusted-launch-faq.md).
