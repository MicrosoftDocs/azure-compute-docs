---
title: VM Processor Mode
description: Feature that lets you choose between fixed-frequency (Deterministic) and higher-frequency (Opportunistic) processor behavior on Azure Cobalt 200 virtual machines.
author: noahwood28
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 11/07/2026
ai-usage: ai-generated
ms.author: noahwood
ms.reviewer: mimckitt, mattmcinnes
# Customer intent: "As a cloud architect, I want to control whether my Arm-based VM runs at a fixed frequency or can run at higher frequencies, so that I can balance consistent performance against peak performance for my workload."
---
# VM Processor Mode feature: Deterministic and Opportunistic modes

VM Processor Mode is an Azure VM feature that gives you control over how the processor frequency of your virtual machine behaves. It's currently available on Azure VM sizes powered by Azure Cobalt 200 processors, such as Standard_D16ps_v7.

Processor Mode offers two settings:

**Deterministic**: The VM's processors run at a consistent base frequency and don't burst above it. This mode gives predictable, repeatable performance with low run-to-run variance, which suits benchmarking, tightly coupled workloads, and applications that need consistent response times.

**Opportunistic**: The VM's processors can burst above the base frequency when thermal and power headroom is available. This mode gives higher peak performance for bursty, throughput-oriented, or latency-sensitive workloads that benefit from extra clock speed when it's available.

If Processor Mode isn't set during deployment, the VM deploys in the default mode for its size. For Cobalt 200 VMs, the default is Deterministic. Processor Mode is configurable both at deployment time and at runtime, allowing customers to tune each VM as workload needs change. There's *no extra charge* to use either processor mode. The VM price is the same whether the VM runs in Deterministic or Opportunistic mode.

## Processor Mode values

The setting is exposed as the `processorMode` property under the VM's `hardwareProfile`. It accepts the following values:

| Value | What it does | Typical use |
| --- | --- | --- |
| Deterministic | Runs the processor at a consistent base frequency without bursting above it. This value is the default for Cobalt 200 VMs. | Workloads that need consistent, repeatable performance and low run-to-run variance. |
| Opportunistic | Allows the processor to burst above base frequency when thermal and power headroom permits. | Bursty, throughput-oriented, or latency-sensitive workloads that benefit from higher peak performance. |

> [!NOTE]
> Opportunistic mode allows higher frequencies but doesn't guarantee them. The actual frequency at any moment depends on the host, the workload, and thermal and power conditions.

## Supported VM sizes

Processor Mode is supported on the following VM series, all powered by [Azure Cobalt 200 processors](./sizes/cobalt-overview.md):

| Size family | Series |
| --- | --- |
| [General purpose D-family](./sizes/general-purpose/d-family.md) | Dpsv7, Dpdsv7, Dplsv7, Dpldsv7 |
| [Memory optimized E-family](./sizes/memory-optimized/e-family.md) | Epsv7, Epdsv7 |
| [Memory optimized M-family](./sizes/memory-optimized/m-family.md) | Mpsv4, Mpdsv4 |
| [Storage optimized L-family](./sizes/storage-optimized/l-family.md) | Lpsv5 |

## Processor Mode configuration

You can configure Processor Mode by using the Azure portal, the Azure CLI, Azure PowerShell, or Azure Resource Manager (ARM) templates.

## Azure portal

In the Azure portal, the VM creation workflow includes a setting for Processor Mode.

Start creating a VM as usual (for example, select **Create a resource > Virtual machine** and fill out the Basics tab).

1. In the Size section of the Basics tab, select a supported VM size, such as Standard_D16ps_v7. Make sure you also select an Arm64 image.

2. Go to the **Advanced** tab.

3. In the **Performance** section, set **Processor mode**. Select **Opportunistic** to allow higher frequencies, or **Deterministic** for fixed frequency. On Cobalt 200 VMs, Deterministic is the default.

4. Continue with the rest of the VM creation (set up disks, networking, and so on) and create the VM.

## Azure CLI

Azure CLI version 2.91.0 or later supports the `--processor-mode` parameter on `az vm create` and `az vm update`. To check your version, run `az version`. To upgrade, run `az upgrade`.

To create a VM in Opportunistic mode, specify `Opportunistic` for `--processor-mode`. The following example creates an Arm64 Ubuntu VM on a Cobalt 200 size:

```azurecli
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --location <region> \
  --image Canonical:ubuntu-24_04-lts:server-arm64:latest \
  --size Standard_D16ps_v7 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --processor-mode Opportunistic
```

To deploy with fixed frequency instead, specify `--processor-mode Deterministic`. Because Deterministic is the default for Cobalt 200 VMs, you can also omit the parameter.

> [!TIP]
> Cobalt 200 VMs require an Arm64 OS image. Use an image URN with an Arm64 SKU, such as `server-arm64`. A generic image alias might resolve to an x64 image, which causes the deployment to fail.

## PowerShell

Use Azure PowerShell and set the property on the VM configuration object before you create the VM.

```powershell
$vmConfig = New-AzVMConfig -VMName "myVM" -VMSize "Standard_D16ps_v7"

$vmConfig.HardwareProfile.ProcessorMode = "Opportunistic"
```

Then set the OS, image, and network configuration as usual, and use `New-AzVM` to create the VM.

## ARM template (Azure Resource Manager)

For automation or infrastructure-as-code deployments, you can set Processor Mode in an ARM template. In the virtual machine resource definition, specify `processorMode` under the `hardwareProfile` property:

`processorMode` - Set this to `Opportunistic` to allow higher frequencies, or `Deterministic` for fixed frequency. If you omit this property, the VM uses the default mode for its size, which is `Deterministic` for Cobalt 200 VMs.

For example:

```json
"properties": {
  "hardwareProfile": {
    "vmSize": "Standard_D16ps_v7",
    "processorMode": "Opportunistic"
  },
  ...
}
```

Make sure to use an API version **2026-04-01 or later** for the Microsoft.Compute/virtualMachines resource in your template, as that's when the `processorMode` property was introduced.

## Change processor mode on an existing VM

You can change the processor mode of an existing VM that runs on a supported size. The change applies without a restart or deallocation, in either direction.

**Using Azure portal**

Go to your VM, select **Settings** > **Configuration**, and then set **Processor mode**.

**Using Azure CLI**

```azurecli
az vm update \
  --resource-group myResourceGroup \
  --name myVM \
  --processor-mode Opportunistic
```

To switch back to fixed frequency, run the same command with `--processor-mode Deterministic`.

**Using PowerShell**

```powershell
$vm = Get-AzVM -ResourceGroupName "myResourceGroup" -Name "myVM"

$vm.HardwareProfile.ProcessorMode = "Opportunistic"

Update-AzVM -ResourceGroupName "myResourceGroup" -VM $vm
```

## Verify the processor mode

To check the processor mode configured on a VM, run:

```azurecli
az vm show \
  --resource-group myResourceGroup \
  --name myVM \
  --query "hardwareProfile.processorMode" \
  --output tsv
```

The command returns `Opportunistic` or `Deterministic`. If it returns nothing, the property isn't set and the VM uses the default mode for its size (`Deterministic` for Cobalt 200 VMs).

> [!NOTE]
> This command confirms the configured setting. It doesn't report the processor's current frequency, and it doesn't guarantee that higher frequencies are active at that moment.

## Identify supported VM sizes

Processor Mode support depends on the VM size. To check whether a size supports Processor Mode, you can use the Azure CLI or the Azure portal.

**Using Azure CLI**

Run the following command to retrieve the capabilities of a VM size. Use any region where the size is available:

```azurecli
az vm list-skus \
  --location <region> \
  --resource-type virtualMachines \
  --size Standard_D16ps_v7 \
  --query "[].capabilities[?name=='SupportedProcessorModes']" \
  --output json
```

The `SupportedProcessorModes` capability lists the processor modes the size supports. If the capability isn't returned, the size doesn't support Processor Mode.

**Using Azure portal**

If you select a VM size that doesn't support Processor Mode, the **Processor mode** setting isn't available.

## Considerations

There are some important rules and considerations to understand when using Processor Mode:

- Processor Mode is supported only on the VM series listed in [Supported VM sizes](#supported-vm-sizes).

- Support depends on the VM size. If you request a processor mode on an unsupported size, the request fails with an error. The platform doesn't silently ignore the setting.

- `Deterministic` and `Opportunistic` are the only supported values. Values are case-sensitive and must match exactly.

- If you don't set Processor Mode, the VM runs in the default mode for its size. For Cobalt 200 VMs, the default is Deterministic.

- You can set Processor Mode when you create a VM or by updating an existing VM. Changing the mode on an existing VM doesn't require a restart or deallocation.

- Processor Mode is supported on virtual machine scale sets in both Uniform and Flexible orchestration modes.

- Opportunistic mode allows higher frequency when headroom is available, but exact boost behavior depends on the host, the workload, and thermal and power conditions. Choose Deterministic when run-to-run consistency matters more than peak throughput.

## Next steps

- [VM vCore Customization](vm-customization.md)
- [Azure Cobalt processor-based virtual machines](./sizes/cobalt-overview.md)
- [az vm reference](/cli/azure/vm)
