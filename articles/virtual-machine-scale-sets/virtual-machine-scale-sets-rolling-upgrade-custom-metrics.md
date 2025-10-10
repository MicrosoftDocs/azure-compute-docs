---
title: Custom metrics for rolling upgrades on Virtual Machine Scale Sets
description: Learn about how to configure custom metrics for rolling upgrades on Virtual Machine Scale Sets.
author: mimckitt
ms.author: mimckitt
ms.topic: how-to
ms.service: azure-virtual-machine-scale-sets
ms.date: 11/7/2024
ms.reviewer: ju-shim
ms.custom: upgradepolicy, N-Phase, ignite-2024
# Customer intent: "As a system administrator, I want to configure custom metrics for rolling upgrades on Virtual Machine Scale Sets so that I can control the order and conditions under which my virtual machines are upgraded, ensuring minimal downtime and optimal application performance."
---
# Configure custom metrics for rolling upgrades on Virtual Machine Scale Sets

Custom metrics for rolling upgrades enables you to utilize the [application health extension](virtual-machine-scale-sets-health-extension.md) to emit custom metrics to your Virtual Machine Scale Set. These custom metrics can be used to tell the scale set the order in which virtual machines should be updated when a rolling upgrade is triggered. The custom metrics can also inform your scale set when an upgrade should be skipped on a specific instance. This allows you to have more control over the ordering and the update process itself. 

Custom metrics can be used in combination with other rolling upgrade functionality such as [automatic OS upgrades](virtual-machine-scale-sets-automatic-upgrade.md), [automatic extension upgrades](../virtual-machines/automatic-extension-upgrade.md) and [MaxSurge rolling upgrades](virtual-machine-scale-sets-maxsurge.md). 

## Requirements

- When using custom metrics for rolling upgrades on Virtual Machine Scale Sets, the scale set must also use the [application health extension with rich health states](virtual-machine-scale-sets-health-extension.md) to report phase ordering or skip upgrade information. Custom metrics upgrades aren't supported when using the application health extension with binary states.
- The application health extension must be set up to use HTTP or HTTPS in order to receive the custom metrics information. TCP isn't supported for integration with custom metrics for rolling upgrades.  

## Concepts

### Phase ordering
A phase is a grouping construct for virtual machines. Each phase is determined by setting metadata emitted from the [application health extension](virtual-machine-scale-sets-health-extension.md) via the `customMetrics` property. The Virtual Machine Scale Set takes the information retrieved from the custom metrics and uses it to place virtual machines into their assigned phases. Within each phase, the Virtual Machine Scale set will also assign upgrade batches. Each batch is configured using the [rolling upgrade policy](virtual-machine-scale-sets-configure-rolling-upgrades.md) which takes into consideration the update domains (UD), fault domains (FD), and zone information of each virtual machine. 

When a rolling upgrade is initiated, the virtual machines are placed into their designated phases. The phased upgrades are performed in numerical sequence order. Virtual Machines in all batches within a phase will be completed before moving onto the next phase. If no phase ordering is received for a virtual machine, the scale set will place it into the last phase  

**Regional scale set**
:::image type="content" source="./media/upgrade-policy/n-phase-regional-scale-set.png" alt-text="Diagram that shows a high level diagram of what happens when using n-phase upgrades on a regional scale set.":::

**Zonal scale set**
:::image type="content" source="./media/upgrade-policy/n-phase-zonal-scale-set.png" alt-text="Diagram that shows a high level diagram of what happens when using n-phase upgrades on a zonal scale set.":::


To specify phase number the virtual machine should be associated with, use `phaseOrderingNumber` parameter.  

```HTTP
{
     “applicationHealthState”: “Healthy”,
      “customMetrics”: "{ \"rollingUpgrade\": { \"PhaseOrderingNumber\": 0 } }"
}
```



### Skip upgrade

Skip upgrade functionality enables an individual instance to be omitted from an upgrade during the rolling upgrade. This is similar to utilizing instance protection but can more seamlessly integrate into the rolling upgrade workflow and into instance level applications. Similar to phase ordering, the skip upgrade information is passed to the Virtual Machine Scale Set via the application health extension and custom metrics settings. When the rolling upgrade is triggered, the Virtual Machine Scale Set checks the response of the application health extensions custom metrics and if skip upgrade is set to true, the instance is not included in the rolling upgrade. 

:::image type="content" source="./media/upgrade-policy/skip-upgrade-zonal.png" alt-text="Diagram that shows a high level diagram of what happens when using skip upgrade on a zonal scale set.":::

For skipping an upgrade on a virtual machine, use `SkipUpgrade` parameter. This tells the rolling upgrade to skip over this virtual machine when performing the upgrades.  

```HTTP
{
     “applicationHealthState”: “Healthy”,
      “customMetrics”: "{ \"rollingUpgrade\": { \"SkipUpgrade\": true} }"
}
```

Skip upgrade and phase order can also be used together: 

```HTTP
{
     “applicationHealthState”: “Healthy”,
      “customMetrics”: "{ \"rollingUpgrade\": { \"SkipUpgrade\": false, \"PhaseOrderingNumber\": 0 } }"
}
```
## Configure the application health extension

The [application health extension](virtual-machine-scale-sets-health-extension.md) requires an HTTP or HTTPS request with an associated port or request path. TCP probes are supported when using the application health extension, but can't set the `ApplicationHealthState` through the probe response body and can't be used with rolling upgrades with custom metrics. 

```json
{
  "extensionProfile" : {
     "extensions" : [
      {
        "name": "HealthExtension",
        "properties": {
          "publisher": "Microsoft.ManagedServices",
          "type": "<ApplicationHealthLinux or ApplicationHealthWindows>",
          "autoUpgradeMinorVersion": true,
          "typeHandlerVersion": "2.0",
          "settings": {
            "protocol": "<protocol>",
            "port": <port>,
            "requestPath": "</requestPath>",
            "intervalInSeconds": 5,
            "numberOfProbes": 1
          }
        }
      }
    ]
  }
}
```


| Name | Value / Example | Data Type |
| ---- | --------------- | --------- |
| protocol | `http` or `https`| string |
| port | Optional when protocol is `http` or `https` | int |
| requestPath | Mandatory when using `http` or `https` | string |
| intervalInSeconds | Optional, default is 5 seconds. This setting is the interval between each health probe. For example, if intervalInSeconds == 5, a probe is sent to the local application endpoint once every 5 seconds. | int |
| numberOfProbes | Optional, default is 1. This setting is the number of consecutive probes required for the health status to change. For example, if numberOfProbles == 3, you need 3 consecutive "Healthy" signals to change the health status from "Unhealthy"/"Unknown" into "Healthy" state. The same requirement applies to change health status into "Unhealthy" or "Unknown" state.  | int |
| gracePeriod | Optional, default = `intervalInSeconds` * `numberOfProbes`; maximum grace period is 7200 seconds | int |

### Install the application health extension

#### [CLI](#tab/azure-cli)

Use [az vmss extension set](/cli/azure/vmss/extension#az-vmss-extension-set) to add the application health extension to the scale set model definition.

Create a json file called `extensions.json` with the desired settings.

```json
{
  "protocol": "<protocol>",
  "port": <port>,
  "requestPath": "</requestPath>",
  "gracePeriod": <healthExtensionGracePeriod>
}
```

Apply the application health extension. 

```azurecli-interactive
az vmss extension set \
  --name ApplicationHealthLinux \
  --publisher Microsoft.ManagedServices \
  --version 2.0 \
  --resource-group myResourceGroup \
  --vmss-name myScaleSet \
  --settings ./extension.json
```

Upgrade the virtual machines in the scale set. This step is only required if your scale set is using a manual upgrade policy. For more information on upgrade policies, see [upgrade policies for Virtual Machine Scale Sets](virtual-machine-scale-sets-upgrade-policy.md)

```azurecli-interactive
az vmss update-instances \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --instance-ids "*"
```

#### [PowerShell](#tab/azure-powershell)

Use the [Add-AzVmssExtension](/powershell/module/az.compute/add-azvmssextension) cmdlet to add the application health extension to the scale set model definition.

```azurepowershell-interactive
# Define the scale set variables
$vmScaleSetName = "myScaleSet"
$vmScaleSetResourceGroup = "myResourceGroup"

# Define the application health extension properties
$publicConfig = @{"protocol" = "http"; "port" = 80; "requestPath" = "/healthEndpoint"; "gracePeriod" = 600};
$extensionName = "myHealthExtension"
$extensionType = "ApplicationHealthWindows"
$publisher = "Microsoft.ManagedServices"

# Get the scale set object
$vmScaleSet = Get-AzVmss `
  -ResourceGroupName $vmScaleSetResourceGroup `
  -VMScaleSetName $vmScaleSetName

# Add the application health extension to the scale set model
Add-AzVmssExtension -VirtualMachineScaleSet $vmScaleSet `
  -Name $extensionName `
  -Publisher $publisher `
  -Setting $publicConfig `
  -Type $extensionType `
  -TypeHandlerVersion "2.0" `
  -AutoUpgradeMinorVersion $True

# Update the scale set
Update-AzVmss -ResourceGroupName $vmScaleSetResourceGroup `
  -Name $vmScaleSetName `
  -VirtualMachineScaleSet $vmScaleSet
  
# Upgrade instances to install the extension.
Update-AzVmssInstance -ResourceGroupName $vmScaleSetResourceGroup `
  -VMScaleSetName $vmScaleSetName `
  -InstanceId '*'

```

##### [REST](#tab/rest-api)

Apply the application health extension using [create or update](/rest/api/compute/virtual-machine-scale-set-vm-extensions/create-or-update).

```http
PUT `/subscriptions/subscription_id/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/extensions/myHealthExtension?api-version=2018-10-01`

Request body
{
  "name": "myHealthExtension",
  "location": "<location>",
  "properties": {
    "publisher": "Microsoft.ManagedServices",
    "type": "ApplicationHealthWindows",
    "autoUpgradeMinorVersion": true,
    "typeHandlerVersion": "2.0",
    "settings": {
      "protocol": "<protocol>",
      "port": <port>,
      "requestPath": "</requestPath>",
      "intervalInSeconds": <intervalInSeconds>,
      "numberOfProbes": <numberOfProbes>,
      "gracePeriod": <gracePeriod>
    }
  }
}
```

Upgrade the virtual machines in the scale set. This step is only required if your scale set is using a manual upgrade policy. For more information on upgrade policies, see [upgrade policies for Virtual Machine Scale Sets](virtual-machine-scale-sets-upgrade-policy.md)

```http
POST `/subscriptions/<subscriptionId>/resourceGroups/<myResourceGroup>/providers/Microsoft.Compute/virtualMachineScaleSets/< myScaleSet >/manualupgrade?api-version=2022-08-01`

Request body
{
  "instanceIds": ["*"]
}
```

---

### Configure the application health extension response
Configuring the custom metrics response can be accomplished in many different ways. It can be integrated into existing applications, dynamically updated and be used along side various functions to provide an output based on a specific situation. 

These sample applications include phase order and skip upgrade parameters into the custom metrics response. 

##### [Bash](#tab/bash)

```bash
#!/bin/bash

# Open firewall port (replace with your firewall rules as needed)
sudo iptables -A INPUT -p tcp --dport 8000 -j ACCEPT

# Create Python HTTP server for responding with JSON
cat <<EOF > server.py
import json
from http.server import BaseHTTPRequestHandler, HTTPServer

# Function to generate the JSON response
def generate_response_json():
    return json.dumps({
        "ApplicationHealthState": "Healthy",
        "CustomMetrics": json.dumps({
            "RollingUpgrade": {
                "PhaseOrderingNumber": 1,
                "SkipUpgrade": "false"
            }
        })
    })

class RequestHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        # Respond with HTTP 200 and JSON content
        self.send_response(200)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        response = generate_response_json()
        self.wfile.write(response.encode('utf-8'))

# Set up the HTTP server
def run(server_class=HTTPServer, handler_class=RequestHandler):
    server_address = ('localhost', 8000)
    httpd = server_class(server_address, handler_class)
    print('Starting server on port 8000...')
    httpd.serve_forever()

if __name__ == "__main__":
    run()
EOF

# Run the server in the background
python3 server.py &

# Store the process ID of the server
SERVER_PID=$!

# Wait a few seconds to ensure the server starts
sleep 2

# Confirm execution
echo "Server has been started on port 8000 with PID $SERVER_PID"

```

##### [PowerShell](#tab/powershell)

```powershell
# Define the script path
$scriptPath = "$env:TEMP\server.ps1"

# Create the PowerShell HTTP Server Script
$serverScript = @"
`$Hso = New-Object Net.HttpListener
`$Hso.Prefixes.Add('http://localhost:8000/')
`$Hso.Start()

Write-Host 'Starting server on port 8000...'

# Function to Generate JSON Response (Matching Python Format)
function GenerateResponseJson {
    # Create JSON string for CustomMetrics
    `$customMetricsJson = (@{
        'RollingUpgrade' = @{
            'PhaseOrderingNumber' = 1
            'SkipUpgrade' = 'false'
        }
    } | ConvertTo-Json -Depth 10) -replace "`n", "" -replace "\s{2,}", ""  # Ensuring a single-line JSON string

    # Create main JSON response
    `$response = @{
        'ApplicationHealthState' = 'Healthy'
        'CustomMetrics' = `$customMetricsJson  # Embed JSON string inside JSON
    }

    return (`$response | ConvertTo-Json -Depth 10)
}

# Keep the server running
while (`$Hso.IsListening) {
    try {
        `$context = `$Hso.GetContext()
        `$response = `$context.Response
        `$response.StatusCode = 200
        `$response.ContentType = 'application/json'
        `$responseJson = GenerateResponseJson
        `$responseBytes = [System.Text.Encoding]::UTF8.GetBytes(`$responseJson)
        `$response.OutputStream.Write(`$responseBytes, 0, `$responseBytes.Length)
        `$response.Close()
        Write-Host "Responded to request at $(Get-Date)"
    }
    catch {
        Write-Host "Error occurred: $_"
    }
}
"@

# Write the script to a file
$serverScript | Out-File -FilePath $scriptPath -Encoding UTF8

# Verify if the file exists before starting the process
if (Test-Path $scriptPath) {
    Write-Host "Server script successfully created at $scriptPath"
    Start-Process -NoNewWindow -FilePath "powershell.exe" -ArgumentList "-ExecutionPolicy Bypass -File `"$scriptPath`"" -PassThru | ForEach-Object {
        # Store Process ID
        $SERVER_PID = $_.Id
        Write-Host "Server has been started on port 8000 with PID $SERVER_PID"
    }
} else {
    Write-Host "Error: Server script not found at $scriptPath"
}

```

---

For more response configuration examples, see [application health samples](https://github.com/Azure-Samples/application-health-samples)


## Next steps
Learn how to [perform manual upgrades](virtual-machine-scale-sets-perform-manual-upgrades.md) on Virtual Machine Scale Sets. 
