---
title: Create a VM with adaptive placement (preview)
description: Learn how to create an Azure virtual machine with adaptive placement, so Azure deploys your preferred VM size or a compatible alternative when your preferred size isn't available.
author: lavankk
ms.author: lavanyak
ms.reviewer: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: how-to
ms.date: 11/04/2026
ms.custom: references_regions
ai-usage: ai-assisted
---

# Create a VM with adaptive placement (preview)

> [!IMPORTANT]
> VM adaptive placement is currently in preview. See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, in preview, or otherwise not yet released into general availability.

Adaptive placement is a way to create a virtual machine. With a standard VM create, you name one exact VM size, and the deployment fails if that size isn't available. With adaptive placement, you name a *preferred* size instead. Azure deploys your preferred size when it can. If the size isn't available in the region, Azure deploys a compatible alternative size with the same number of vCPUs and the same or newer generation.

The result: fewer allocation failures and less time spent retrying deployments or hand-picking fallback sizes.

## When to use adaptive placement

Use adaptive placement when:

- You create **individual VMs** rather than Virtual Machine Scale Sets.
- Your workload can run on more than one VM size within a family, for example any D-series size with 4 vCPUs.
- Getting the VM deployed matters more to you than running on one specific hardware generation.

If you deploy at scale with Virtual Machine Scale Sets and want to spread across many sizes, use [instance mix for Virtual Machine Scale Sets](../virtual-machine-scale-sets/instance-mix-overview.md) instead.

## How adaptive placement works

1. **You specify a preferred size.** In the VM create request, you set `flexProfile.preferredVMSize` (for example `Standard_D4s_v5`) instead of `hardwareProfile.vmSize`.
1. **Azure checks your preferred size first.** If the region has capacity for the preferred size, Azure deploys it.
1. **Azure picks an alternative when needed.** If the preferred size can't be used, Azure selects a compatible size that:
   - Has the **same vCPU count** as the preferred size.
   - Is the **same or a newer version** than the preferred size (for example, v5 or later for a v5 preferred size).
   - Is in a supported family (D, E, or F).
   - Delivers **performance equivalent to your preferred size**.
1. **Azure falls back safely.** If Azure can't evaluate alternatives in time, it attempts to deploy your preferred size, the same as a standard VM create.

## Supported VM sizes

During preview, adaptive placement supports the following families as the preferred size:

| Family | Supported |
|---|---|
| D-series (general purpose) | Yes |
| E-series (memory optimized) | Yes |
| F-series (compute optimized) | Yes |
| All other families (for example B, L, M, N, H) | No |

## Region availability

Adaptive placement is available in the following regions: `centraluseuap`, `eastasia`, `eastus2euap`, `uksouth`, `australiacentral2`, `australiaeast`, `brazilsouth`, `canadacentral`, `francesouth`, `germanywestcentral`, `japaneast`, `koreasouth`, `northcentralus`, `northeurope`, `southafricawest`, `swedencentral`, `switzerlandwest`, `ukwest`, and `westcentralus`.

## Prerequisites

- An Azure subscription registered for the `Microsoft.Compute/FlexProfileEnabled` preview feature. To request registration, run:

  ```azurecli
  az feature register --namespace Microsoft.Compute --name FlexProfileEnabled
  ```

- A target region listed in the [Region availability](#region-availability) section.
- Microsoft.Compute REST API version `2026-04-01` or later.
- Quota in the target region for the VM family of your preferred size. For the best results, also get quota for the Intel and AMD versions of your preferred size within the same VM family.

## Create a VM with adaptive placement

### [REST API](#tab/rest)

Send a `PUT` request to create the VM. Include `flexProfile` and **omit** `hardwareProfile.vmSize`.

```http
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmName}?api-version=2026-04-01
```

```json
{
  "location": "eastasia",
  "properties": {
    "flexProfile": {
      "preferredVMSize": "Standard_D4s_v5"
    },
    "storageProfile": {
      "imageReference": {
        "publisher": "Canonical",
        "offer": "ubuntu-24_04-lts",
        "sku": "server",
        "version": "latest"
      },
      "osDisk": {
        "createOption": "FromImage",
        "managedDisk": { "storageAccountType": "Premium_LRS" }
      }
    },
    "osProfile": {
      "computerName": "myVM",
      "adminUsername": "azureuser",
      "linuxConfiguration": {
        "disablePasswordAuthentication": true,
        "ssh": {
          "publicKeys": [
            {
              "path": "/home/azureuser/.ssh/authorized_keys",
              "keyData": "<your-public-key>"
            }
          ]
        }
      }
    },
    "networkProfile": {
      "networkInterfaces": [
        { "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/networkInterfaces/{nicName}" }
      ]
    }
  }
}
```

### [Azure CLI](#tab/cli)

Azure CLI version 2.92.0 or later supports adaptive placement through the `--preferred-size` parameter on `az vm create` and `az vm update`.

### [Azure PowerShell](#tab/powershell)

Use Az.Compute version 11.11.0 or later, included in Az version 16.5.0.

Create the VM configuration with `-PreferredVMSize` instead of `-VMSize`:

```powershell
$vmConfig = New-AzVMConfig -VMName "myVM" -PreferredVMSize "Standard_D4s_v5"
```

Configure the operating system, image, disks, and network interface on this object, then pass the completed configuration to `New-AzVM -VM $vmConfig`. Don't specify `-VMSize` alongside `-PreferredVMSize`.

---

## Check which size was deployed

After the VM is created, get the VM to see the size Azure deployed:

```http
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmName}?api-version=2026-04-01
```

In the response, `hardwareProfile.vmSize` shows the VM size that Azure deployed.

## Pricing and billing

There's no extra charge for using adaptive placement. You're billed at standard rates for the **VM size that Azure deploys**, which might be your preferred size or a compatible alternative. Check the deployed size after creation so you know which rate applies.

Azure Reservations and other pricing benefits apply based on the deployed size, using the same rules as any other VM. For current rates, see [Virtual Machines pricing](https://azure.microsoft.com/pricing/details/virtual-machines/).

## Lifecycle operations

VMs created with adaptive placement support the same lifecycle operations as other VMs, including:

- Start, stop, and deallocate.
- Hibernate and resume.
- Restart and redeploy.

Deallocating and starting a VM created with adaptive placement doesn't automatically reselect its size. The VM starts on the previously deployed size, so the billed size doesn't change because of the stop and start.

If you update `flexProfile.preferredVMSize` while the VM is deallocated, that update can select a different compatible size. The VM then starts on the newly selected size, and billing follows that deployed size.

VM size updates aren't supported while the VM is hibernated.

## Limitations

- `flexProfile` and `hardwareProfile.vmSize` are mutually exclusive. A request that sets both is rejected.
- Adaptive placement supports single VMs only. Virtual Machine Scale Sets aren't supported.
- Only D, E, and F families are supported as the preferred size.
- Alternative sizes always keep the same vCPU count as the preferred size.
- You can't use the Azure portal to create a VM with adaptive placement during preview.
- Adaptive placement doesn't support VMs associated with Capacity Reservations, Azure Dedicated Hosts, Availability Sets, or Proximity Placement Groups.

## Frequently asked questions

**Does Azure ever give me fewer vCPUs than I ask for?**
No. Any alternative size has the same vCPU count as your preferred size.

**Does Azure deploy an older generation than my preferred size?**
No. Alternatives are the same or a newer version than your preferred size.

**Does an alternative size perform worse than my preferred size?**
No. Any alternative size delivers performance equivalent to your preferred size.

**What am I billed for?**
The size Azure deploys, at standard rates.

**What happens if no alternative size is available?**
Azure attempts to deploy your preferred size, the same as a standard VM create. If that size isn't available, the deployment fails with the usual allocation error.

**Do I need quota for alternative sizes?**
Yes. Azure can deploy an alternative size only if your subscription has quota for it. To get the most benefit from adaptive placement, request quota in the target region for both the Intel and AMD versions of your preferred size within the same VM family. For example, if your preferred size is `Standard_D4s_v5` (Intel), also request quota for `Standard_D4as_v5` (AMD).

**Can I use adaptive placement on an existing VM?**
Yes. You can enable adaptive placement on an existing VM by updating it to specify `flexProfile.preferredVMSize` instead of `hardwareProfile.vmSize`. The update can change the deployed size.

You can also switch back to a standard VM by submitting a `PUT` request that omits `flexProfile` and specifies `hardwareProfile.vmSize`.

## Next steps

- [Sizes for virtual machines in Azure](sizes/overview.md).
- [Check vCPU quotas](quotas.md).
