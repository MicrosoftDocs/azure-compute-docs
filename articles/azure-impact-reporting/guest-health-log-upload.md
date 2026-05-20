---
title: Azure HPC Guest Health Reporting - Provide Log File 
description: Provide a log file along with guest health report. 
author: mvrequa 
ms.author: mirequa 
ms.service: azure 
ms.topic: overview
ms.date: 04/08/2026 
ms.custom: template-overview 
---


# How to upload log files to Guest Health Reporting

Providing a log file helps make more precise repair actions. A log file should be uploaded prior to making the guest health report (GHR), with the GHR request containing a `LogUrl` field inside the `additionalProperties` section of the request body. The `LogUrl` points to the uploaded log file from one of the supported platform-specific diagnostic tools (for example Nvidia bug report). 


## Log capture

1. Collect the log that captures the diagnostic information for the virtual machine associated with the GHR.  
  For this payload to be automatically processed, the payload format **MUST** conform to specific requirements as defined in the *Preparing Log Files for Upload* section below.

1. Request an access token to your specified storage account/container via POST request to: `https://management.azure.com/subscriptions/[subId]/providers/Microsoft.Impact/getUploadToken?api-version=2025-01-01-preview`. 

1.	Locate compressed file and upload using the provided upload URL/token. This can be done using any library that you prefer (AzPowershell, Python, etc.). A few examples:

    - AzCli:
`az storage blob upload --file "path/to/local/file.gz" --blob-url "https://ghrloguploadprod.blob.core.windows.net/[container]/[datetime]_[randomHash].gz?[SasToken]"`
    - Curl:
    `curl -X PUT -H "x-ms-blob-type: BlockBlob" -H "Content-Type: application/octet-stream" --data-binary @"path/to/local/file.gz" "https://ghrloguploadprod.blob.core.windows.net/[container]/[datetime]_[randomHash].tar.xz?[SasToken]"`    

1.	Trim off the trailing SAS token from the provided URL. In this example, the uploaded file was `https://ghrloguploadprodblob.core.windows.net/[ContainerName]/20250210213205_f71b77d3.gz?[SasTokenHere]`. Reference this trimmed LogUrl in the LogUrl field and send the impact as normal (GHR request to `https://management.azure.com/subscriptions/[subId]/providers/Microsoft.Impact/workloadImpacts/[NameOfImpact]?api-version=2025-01-01-preview`):

```
{
    "properties": {
        "startDateTime": "2025-02-20T01:06:21.3886467Z",
        "impactCategory": "Resource.Hpc.Unhealthy.HpcMissingGpu",
        "impactDescription": "Missing GPU device",
        "additionalProperties": {
            "LogUrl": "https://ghrloguploadprodblob.core.windows.net/customerNameHere/20250210213205_f71b77d3.gz",
            "PhysicalHostName": "dummyPhysicalHostNameForDemo",
            "Manufacturer": "Nvidia",
            "SerialNumber": "1324567897654",
            "ModelNumber": "NV1LB960",
            "Location": "2"
        }
    }
}
```

## Prepare log files for upload

For GHR to automatically process log files and influence the repair, the uploaded log MUST be platform-relevant issue reporting format - for example an Nvidia Bug Report (NVBR) log file. Other files can be uploaded, such as dmesg output, as long as they're gzipped for offline investigation. GHR accepts log files as a single file compressed as a .gz file.

### Upload a gzip file (*.gz)

Collect the log and compress it via gzip (if the output isn't already a .gz file).  When uploading the source is this local file and the destination is the LogUrl as returned by the /getUploadToken endpoint, described in the next section. The LogUrl grants temporary write access to the container via the provided SAS token. The LogUrl has a blob name which renames the file during upload.

### Understand the LogUrl to upload

The /getUploadToken endpoint returns a LogUrl. Here's an example return value:
`https://ghrloguploadprod.blob.core.windows.net/hpcdemo/20260323194745_99e980b9.gz?sasTokenHere`
This contains several key pieces of information. Using the above return value as an example:
-	The destination account: ghrloguploadprod
-	The destination container: hpcdemo
-	The suggested file name: 20260323194745_99e980b9.gz
    -Notice that the returned file name consists of:
        - The current UTC date and time formatted as a 14-character string: four-digit year, two-digit month, two-digit day, two-digit hour, two-digit minute, two-digit second (for example, 20260323194745)
        - An underscore, followed by a random suffix of eight hexadecimal characters. (_99e980b9)
        - The suggested file type (.gz)

When uploading, don't change any of the above sections of the LogUrl.

After uploading the log file, trim off the trialing SAS token (as a security best practice), then provide this LogUrl in the GHR request:
- The destination account and file name are validated to ensure that they contain a known storage account, container, a datetime and the 8 character random hash, and the uploaded file type .gz.
- If the LogUrl does not meet these requirements, the GHR request returns "Bad Request" with code "INVALID_FORMAT" response.
