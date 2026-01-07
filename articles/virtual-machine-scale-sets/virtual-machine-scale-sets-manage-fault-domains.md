---
title: Manage fault domains in Azure Virtual Machine Scale Sets
description: Learn how to choose the right number of fault domains while creating a Virtual Machine Scale Set.
author: mimckitt
ms.author: mimckitt
ms.topic: concept-article
ms.service: azure-virtual-machine-scale-sets
ms.subservice: availability
ms.date: 09/02/2025
ms.reviewer: cynthn
ms.custom: mimckitt, devx-track-azurecli

# Customer intent: As an IT administrator managing cloud resources, I want to select the appropriate number of fault domains for my Virtual Machine Scale Set, so that I can ensure high availability and minimize the risk of downtime due to hardware failures.
---
# Choosing the right number of fault domains for Virtual Machine Scale Set

A fault domain is a fault isolation group within an availability zone or datacenter of hardware nodes that share the same power, networking, cooling, and platform maintenance schedule. Virtual machine (VM) instances that are on different fault domains are not likely to be impacted by the same planned or unplanned outage.

You can specify how instances are spread across fault domains within a region or zone. The fault domain configuration options available to you vary depending on the orchestration mode your scale set uses, and whether [your scale set uses availability zones](./virtual-machine-scale-sets-use-availability-zones.md).

## Supported fault domain configurations

There are two types of fault domain spreading: max and fixed.

The following table summarizes the supported `platformFaultDomainCount` values for different orchestration modes and deployment types:

| Orchestration Mode | Deployment Type | Supported Values | Default Value |
|-------------------|----------------|------------------|---------------|
| Uniform | Zonal or zone-spanning | 1, 5 | 1 |
| Uniform | Nonzonal (regional)) | 1, 2, 3, 4, 5 | 5 |
| Flexible | Zonal or zone-spanning | 1 | 1 |
| Flexible | Nonzonal (regional) | 1, 2, 3 | 1 |

### Max spreading

Azure spreads the scale set's VM instances across as many fault domains as possible, on a best-effort basis. It might use greater or fewer than five fault domains.

Set `platformFaultDomainCount` to a value of `1` for max spreading.

When you look at the scale set's VM instances, you only see one fault domain. This is expected behavior. The spreading is implicit.

> [!NOTE]
> We recommend using max spreading (`platformFaultDomainCount = 1`) for most scale sets, because it provides the best spreading in most cases. If you need your instances to be spread across distinct hardware isolation units, we recommend you configure max spreading and use multiple availability zones. Azure spreads the instances across fault domains within each zone.

### Fixed spreading

When your scale set uses fixed spreading, Azure spreads the VM instances across the specified number of fault domains. If the scale set can't allocate VMs that meet specified fault domain count, the request fails.

Set the `platformFaultDomainCount` to a value greater than 1 for fixed spreading.

There are specific numbers of fault domains you can select depending on your orchestration mode and deployment type.

## Uniform orchestration mode

Scale sets with Uniform orchestration support different fault domain configurations depending on the deployment type:

- **Zonal or zone-spanning scale sets:** Use max spreading by default (`platformFaultDomainCount = 1`). You can optionally configure fixed spreading with five fault domains (`platformFaultDomainCount = 5`).

- **Nonzonal scale sets:** Use fixed spreading with five fault domains by default (`platformFaultDomainCount = 5`). You can optionally configure max spreading (`platformFaultDomainCount = 1`).

  You can also consider aligning the number of scale set fault domains with the number of managed disk fault domains (for example, `platformFaultDomainCount = 2`). This alignment can help prevent loss of quorum if an entire disk fault domain goes down. The fault domain count can be set to less than or equal to the number of managed disks fault domains available in each of the regions. Refer to the [availability sets documentation](../virtual-machines/availability-set-overview.md) to learn about the number of managed disk fault domains by region.

## Flexible orchestration mode

Scale sets with Flexible orchestration support different fault domain configurations depending on the deployment type:

- **Zonal or zone-spanning scale sets**: Only support max spreading (`platformFaultDomainCount = 1`).

- **Nonzonal scale sets**: Use max spreading (`platformFaultDomainCount = 1`) by default. You can optionally configure fault domain counts of `2` or `3`.

## Fault domains and tools

You can configure fault domains by using standard Azure deployment tools and APIs.

### REST API

Configure the scale set's fault domain spreading by setting the property `properties.platformFaultDomainCount`. Refer to the REST API documentation for [Virtual Machine Scale Sets](/rest/api/compute/virtualmachinescalesets/createorupdate).

### Azure CLI

> [!IMPORTANT]
> VM scale sets created using PowerShell and Azure CLI default to Flexible orchestration mode if no orchestration mode is specified.

Configure the scale set's fault domain spreading by setting the `--platform-fault-domain-count` parameter. Refer to the Azure CLI documentation for [Virtual Machine Scale Sets](/cli/azure/vmss#az-vmss-create).

The following examples show how to use the Azure CLI to deploy scale sets with varying configurations:

- **Zone-spanning Flexible scale set that uses three zones, with max spreading in each zone:**

  ```azurecli-interactive
  az vmss create \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --orchestration-mode Flexible \
    --image Ubuntu2204 \
    --admin-username azureuser \
    --zones 1 2 3 \
    --generate-ssh-keys
  ```

- **Zonal (single-zone) Flexible scale set with max spreading:**

  ```azurecli-interactive
  az vmss create \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --orchestration-mode Flexible \
    --image Ubuntu2204 \
    --admin-username azureuser \
    --zones 1 \
    --generate-ssh-keys
  ```

> [!NOTE]
> For zone-spanning and zonal Flexible virtual machine scale set deployments, the fault domain count is automatically set to 1 (max spreading) and can't be configured to a different value.

- **Nonzonal Flexible scale set with fixed spreading:**

  ```azurecli-interactive
  az vmss create \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --orchestration-mode Flexible \
    --image Ubuntu2204 \
    --admin-username azureuser \
    --platform-fault-domain-count 3 \
    --generate-ssh-keys
  ```

- **Nonzonal Uniform scale set with max spreading:** 

  ```azurecli-interactive
  az vmss create \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --orchestration-mode Uniform \
    --image Ubuntu2204 \
    --admin-username azureuser \
    --platform-fault-domain-count 1 \
    --generate-ssh-keys
  ```

It takes a few minutes to create and configure all the scale set resources and VMs.

### Instance View API

When you use the [Virtual Machines - Instance View REST API](/rest/api/compute/virtualmachines/instanceview) with Flexible orchestration mode scale sets, VM instances are distributed across the configured fault domains, but this information isn't exposed through the API. The API responses don't include fault domain or update domain information. This behavior is by design and differs from Uniform orchestration mode where these properties are returned.

## Next steps

- Learn more about [availability and redundancy features](../virtual-machines/availability.md) for Azure environments.
