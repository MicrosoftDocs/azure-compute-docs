---
title: Azure HPC Guest Health Reporting - FAQ 
description: Frequently asked questions for Guest Health Reporting.
author: rolandnyamo 
ms.author: ronyamo 
ms.topic: faq 
ms.service: azure 
ms.date: 09/18/2025 
ms.custom: template-overview 
---

# FAQ for Guest Health Reporting (preview)

Here are answers to common questions about Guest Health Reporting.

> [!IMPORTANT]
> Guest Health Reporting is currently in preview. For legal terms that apply to Azure features that are in beta, in preview, or otherwise not yet released into general availability, see the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## What happens if I don't deallocate a node after sending the request to Guest Health Reporting?

For regular Guest Health Reporting requests to mark a node as unallocatable (UA) or out for repair (OFR), if you don't deallocate virtual machines (VMs) within 30 days after the node becomes UA, the node automatically enters a **HumanInvestigate** status.

For a reset request, there's no timeout because the request doesn't require you to deallocate VMs.

For a restart request, if you don't deallocate VMs within 30 days after the node becomes UA, the node is set to **Available**. This status means that your request to restart the node is ignored.

## How do I upload logs?

1. Get an access token to your storage account or container via
`/subscriptions/[subscriptionId]/providers/Microsft.Impact/getUploadtoken?api-version=2025-01-01preview`.

2. Upload logs by using the upload URL or the shared access signature (SAS) token:

    ```bash
    az storage blob upload –file "path/to/local/file.zip" –blob-url
    https://[storageAccount].blob.core.windows.net/[container]/[datetime]_[randomHash].zip?[SasToken]
    ```

3. Trim off the SAS token and send the report with `LogUrl` filled in:

    ```json
    {
        "properties": {
            "startDateTime": "2025-09-15T01:06:21.3886467Z",
            "impactCategory": "Resource.Hpc.Unhealthy.IBPerformance",
            "impactDescription": "IB low bandwidth",
            "impactedResourceId": "/subscriptions/111111-f1122-2233-11bc-bb00123/resourceGroups/<rg_name>/providers/Microsoft.Compute/virtualMachines/<vm_name>",
            "additionalProperties": {
                    "LogUrl": "https://someurl.blob.core.windows.net/rma",
                    "PhysicalHostName": "GGBB90904476",
                    "VmUniqueId": "1111111-22dr-3345-22rf-34454g89j"
            }
        }
    }

    ```

## Related content

* [What is Guest Health Reporting?](guest-health-overview.md)
* [Report node health by using Guest Health Reporting](guest-health-impact-report.md)
