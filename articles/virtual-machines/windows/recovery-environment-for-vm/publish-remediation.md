---
title: Publish a custom remediation to Azure Compute Gallery (Preview)
description: Publish a tested custom remediation for remote recovery as an Azure Compute Gallery script version, verify regional replication, and retain its resource ID.
author: GabstaMSFT
ms.author: gabsta
ms.service: azure-virtual-machines
ms.subservice: extensions
ms.topic: how-to
ms.date: 11/09/2026
ai-usage: ai-assisted
---

# Publish a custom remediation to Azure Compute Gallery (Preview)

**Applies to:** :heavy_check_mark: Windows VMs

Publish a tested custom remediation for remote recovery as a gallery script version in Azure Compute Gallery. Publication creates a version resource ID that you use to assign the remediation to a VM.

> [!IMPORTANT]
> Custom remediations through remote recovery for Azure VMs are in **preview**. Review the [preview access and scope](overview.md#preview-access-and-scope) and [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) before publishing a remediation.

## Prerequisites

- Confirm preview access and gallery-script availability for your subscription.
- Identify every target VM's [supported region](overview.md#supported-regions) so you can replicate the script version to those regions.
- [Author and test your remediation](author-remediation.md) with the target WinRE runtime and architecture. The fictional example isn't a real repair.
- Obtain the [gallery publishing and Blob Storage permissions](overview.md#permissions), and verify network access.
- Use an existing Azure Compute Gallery, or [create a gallery](/cli/azure/sig#az-sig-create). Register the `Microsoft.Compute` resource provider.

## Prepare the artifact

Use `Microsoft.Compute/galleries/scripts` and `Microsoft.Compute/galleries/scripts/versions`, not gallery applications or images.

Follow the [remediation packaging and execution contract](author-remediation.md#remediation-contract). Exclude credentials, logs, test fixtures, and runtime boot-failure data from the artifact. Use an explicit version, such as `1.0.0`.

The plugin supplies only the detection or repair switch. It doesn't supply a context object or extra arguments. Don't declare required gallery parameters for this workflow.

## Publish the script version

The following procedure links to API references with request examples for version `2025-03-03`. Run management commands on your management workstation, not in WinRE.

1. [Upload the tested artifact to private Blob Storage](/azure/storage/blobs/storage-quickstart-blobs-cli). The artifact can be a `.ps1`, `.exe`, or `.zip` file.
1. [Create a user-delegation shared access signature (SAS)](/azure/storage/blobs/storage-blob-user-delegation-sas-create-cli) for the source blob. Grant read-only, HTTPS-only access with a lifetime that covers import and required replication.
1. [Create the gallery script definition](/rest/api/compute/gallery-scripts/create-or-update?view=rest-compute-2025-03-03&preserve-view=true). Set `properties.supportedOSType` to `Windows`.
1. [Create the gallery script version](/rest/api/compute/gallery-script-versions/create-or-update?view=rest-compute-2025-03-03&preserve-view=true). Set `properties.publishingProfile.source.scriptLink` to the source blob's SAS URI. Include the publication region and every VM consumption region in `properties.publishingProfile.targetRegions`.
1. [Retrieve the gallery script version](/rest/api/compute/gallery-script-versions/get?view=rest-compute-2025-03-03&preserve-view=true). Follow the replication-status example, and verify readiness as described in the next section.

Keep the SAS URI out of source control and logs. A SAS doesn't bypass storage network restrictions. The gallery service must be able to download the blob.

## Verify publication and replication

Before using the version, confirm all of the following values in the gallery script version response:

| Property | Required result |
| --- | --- |
| `properties.provisioningState` | `Succeeded` |
| `properties.replicationStatus.aggregatedState` | `Completed` |
| `properties.replicationStatus.summary` | Every required consumption region appears with state `Completed`. |

Wait for all three checks to pass. If a target region or its status is missing, or replication fails, resolve that condition before assigning the version to a VM in that region.

## Retain the published version ID

Retain the explicit version resource ID, not the source URL or SAS. The following example identifies version `1.0.0`:

```text
/subscriptions/<subscription-id>/resourceGroups/rg-windows-recovery/providers/Microsoft.Compute/galleries/WindowsRecoveryGallery/scripts/ContosoBootRepair/versions/1.0.0
```

Use this ID to [pre-stage or apply the remediation](apply-remediation.md).

## Manage artifact retention

Keep the source blob accessible until import and replication complete. Retain gallery versions and replicas while VMs reference them. Before removing test artifacts, verify that no other VM or recovery workflow depends on them, and preserve required diagnostics.

## Next steps

- [Pre-stage or apply the published version](apply-remediation.md).
- [Monitor recovery mode and progress](monitor-recovery.md).
