---
title: Get standby pool and instance details
description: Learn how to get details about your standby pool for Virtual Machine Scale Sets.
author: mimckitt
ms.author: mimckitt
ms.service: azure-virtual-machine-scale-sets
ms.custom:
  - ignite-2024
ms.topic: how-to
ms.date: 5/6/2025
ms.reviewer: ju-shim
---

# Get standby pool and instance details

> [!IMPORTANT]
> For standby pools to successfully create and manage resources, it requires access to the associated resources in your subscription. Ensure the correct permissions are assigned to the standby pool resource provider in order for your standby pool to function properly. For detailed instructions, see **[configure role permissions for standby pools](standby-pools-configure-permissions.md)**.

A standby pool is a dynamic resource that stays in sync with your scale set as your workload scales up and down. This article discusses how to retrieve various information about your standby pool and the instances within it. 

## Standby pool details
Use the standby pool runtime view APIs to get the current status of your standby pool including how many instances are available, the provisioning state, and what zones are being utilized. 


### [CLI](#tab/cli)

```azurecli
az standby-vm-pool status --resource-group myResourceGroup --name myStandbyPool

{
  "id": "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyVirtualMachinePools/myStandbyPool/runtimeViews/latest",
    {
      "zone": 1
    },
      "instanceCountsByState": [
        {
          "count": 5,
          "state": "Creating"
        },
        {
          "count": 0,
          "state": "Starting"
        },
        {
          "count": 5,
          "state": "Running"
        },
        {
          "count": 0,
          "state": "Deallocating"
        },
        {
          "count": 10,
          "state": "Deallocated"
        },
        {
            "count": 0
            "state": "Hibernating",
        },
        {
            "count": 0
            "state": "Hibernated",
        },
        {
          "count": 0,
          "state": "Deleting"
        }
      ],
      "zone": 2
    },
    {
      "instanceCountsByState": [
        {
          "count": 0,
          "state": "Creating"
        },
        {
          "count": 10,
          "state": "Starting"
        },
        {
          "count": 0,
          "state": "Running"
        },
        {
          "count": 5,
          "state": "Deallocating"
        },
        {
          "count": 5,
          "state": "Deallocated"
        },
        {
            "count": 0
            "state": "Hibernating",
        },
        {
            "count": 0
            "state": "Hibernated",
        },
        {
          "count": 0,
          "state": "Deleting"
        }
      ],
      "zone": 3
    },
    {
      "instanceCountsByState": [
        {
          "count": 0,
          "state": "Creating"
        },
        {
          "count": 0,
          "state": "Starting"
        },
        {
          "count": 5,
          "state": "Running"
        },
        {
          "count": 10,
          "state": "Deallocating"
        },
        {
          "count": 3,
          "state": "Deallocated"
        },
        {
            "count": 0
            "state": "Hibernating",
        },
        {
            "count": 0
            "state": "Hibernated",
        },
        {
          "count": 5,
          "state": "Deleting"
        }
      ],
  "name": "latest",
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
  "type": "Microsoft.StandbyPool/standbyVirtualMachinePools/runtimeViews"
}

```


### [PowerShell](#tab/powershell)

```azurepowershell
Get-AzStandbyVMPoolStatus -ResourceGroupName myResourceGroup -Name myStandbyPool

Id: /subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyVirtualMachinePools/mmyStandbyPool/runtimeViews/latest
InstanceCountSummary: {{
        {
        "zone": 1
        },
        "instanceCountsByState": [
        {
            "state": "Creating",
            "count": 0
        },
        {
            "state": "Starting",
            "count": 5
        },
        {
            "state": "Running",
            "count": 0
        },
        {
            "state": "Deallocating",
            "count": 10
        },
        {
            "state": "Deallocated",
            "count": 5
        },
        {
            "state": "Hibernating",
            "count": 0
        },
        {
            "state": "Hibernated",
            "count": 0
        },
        {
            "state": "Deleting",
            "count": 0
        }
        ],
        "zone": 2
        },
        {
        "instanceCountsByState": [
        {
            "state": "Creating",
            "count": 5
        },
        {
            "state": "Starting",
            "count": 0
        },
        {
            "state": "Running",
            "count": 5
        },
        {
            "state": "Deallocating",
            "count": 10
        },
        {
            "state": "Deallocated",
            "count": 0
        },
        {
            "state": "Hibernating",
            "count": 0
        },
        {
            "state": "Hibernated",
            "count": 0
        },
        {
            "state": "Deleting",
            "count": 0
        }
        ],
        "zone": 3
        },
        {
        "instanceCountsByState": [
        {
            "state": "Creating",
            "count": 0
        },
        {
            "state": "Starting",
            "count": 0
        },
        {
            "state": "Running",
            "count": 5
        },
        {
            "state": "Deallocating",
            "count": 10
        },
        {
            "state": "Deallocated",
            "count": 5
        },
        {
            "state": "Hibernating",
            "count": 0
        },
        {
            "state": "Hibernated",
            "count": 0
        },
        {
            "state": "Deleting",
            "count": 0
        }
        ]
     }
  }
Name                         : latest
ProvisioningState            : Succeeded
ResourceGroupName            : myResourceGroup
Type                         : Microsoft.StandbyPool/standbyVirtualMachinePools/runtimeViews

```


### [REST](#tab/rest)

```HTTP
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.StandbyPool/standbyVirtualMachinePools/{standbyVirtualMachinePoolName}/runtimeViews/latest?api-version=2025-03-01

{
  "properties": {
    "instanceCountSummary": [
      {
        "zone": 1,
        "instanceCountsByState": [
          {
            "state": "creating",
            "count": 5
          },
          {
            "state": "running",
            "count": 0
          },
          {
            "state": "deallocating",
            "count": 10
          },
          {
            "state": "deallocated",
            "count": 5
          },
          {
            "state": "starting",
            "count": 0
          },
        {
            "state": "Hibernating",
            "count": 0
        },
        {
            "state": "Hibernated",
            "count": 0
        },
          {
            "state": "deleting",
            "count": 0
          }
        ]
      },
      {
        "zone": 2,
        "instanceCountsByState": [
          {
            "state": "creating",
            "count": 0
          },
          {
            "state": "running",
            "count": 5
          },
          {
            "state": "deallocating",
            "count": 10
          },
          {
            "state": "deallocated",
            "count": 10
          },
          {
            "state": "starting",
            "count": 0
          },
        {
            "state": "Hibernating",
            "count": 0
        },
        {
            "state": "Hibernated",
            "count": 0
        },
          {
            "state": "deleting",
            "count": 0
          }
        ]
      },
      {
        "zone": 3,
        "instanceCountsByState": [
          {
            "state": "creating",
            "count": 0
          },
          {
            "state": "running",
            "count": 0
          },
          {
            "state": "deallocating",
            "count": 15
          },
          {
            "state": "deallocated",
            "count": 15
          },
          {
            "state": "starting",
            "count": 0
          },
        {
            "state": "Hibernating",
            "count": 0
        },
        {
            "state": "Hibernated",
            "count": 0
        },
          {
            "state": "deleting",
            "count": 0
          }
        ]
      }
    },
    "provisioningState": "Succeeded"
  },
  "id": "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyVirtualMachinePools/pool/runtimeViews/latest",
  "name": "myStandbyPool",
  "type": "Microsoft.StandbyPool/standbyVirtualMachinePools/runtimeViews",
  "systemData": {
    "createdBy": "pooluser@microsoft.com",
    "createdByType": "User",
    "createdAt": "2024-02-14T23:31:59.679Z",
    "lastModifiedBy": "pooluser@microsoft.com",
    "lastModifiedByType": "User",
    "lastModifiedAt": "2024-02-14T23:31:59.679Z"
  }


```

---


## Instance details

When a virtual machine is in a standby pool, the `isVmInStandbyPool` parameter is set to true. When the virtual machine is moved from the pool instance the scale set, the parameter is automatically updated to false. This can be useful in determining when a virtual machine is ready to receive traffic or not.

> [!NOTE]
> The `isVmInStandbyPool` property is only displayed when a virtual machine has been created by the standby pool. Virtual machines created directly by the scale set or attached to the scale set won't have this property. 

### [CLI](#tab/cli)

```azurecli
az vm get-instance-view --resource-group myResourceGroup --name myVM

    "extensions": null,
    "hyperVGeneration": "V2",
    "isVmInStandbyPool": true,
    "maintenanceRedeployStatus": null,
    "statuses": [
      {
        "code": "ProvisioningState/succeeded",
        "displayStatus": "Provisioning succeeded",
        "level": "Info",
        "message": null,
        "time": "2024-08-02T17:22:46.295536+00:00"
      },
      {
        "code": "PowerState/deallocated",
        "displayStatus": "VM deallocated",
        "level": "Info",
        "message": null,
        "time": null
      }
    ],
```

### [PowerShell](#tab/powershell)

```azurepowershell
Get-AzVM -ResourceGroupName myResourceGroup -Name myVM -Status


ResourceGroupName : myResourceGroup
Name              : myVM
HyperVGeneration  : V2
IsVMInStandbyPool : True
BootDiagnostics   : 
Disks[0]          : 
  Name            : myVM_1a995f8c_OsDisk_1_c250259c33b942eb910504cf6512332d
  Statuses[0]     : 
    Code          : ProvisioningState/succeeded
    Level         : Info
    DisplayStatus : Provisioning succeeded
    Time          : 10/4/2024 7:45:11 PM
Statuses[0]       : 
  Code            : ProvisioningState/succeeded
  Level           : Info
  DisplayStatus   : Provisioning succeeded
  Time            : 10/4/2024 7:45:11 PM
Statuses[1]       : 
  Code            : PowerState/deallocated
  Level           : Info
  DisplayStatus   : VM deallocated
```


### [REST](#tab/rest)

```HTTP
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM/instanceView?api-version=2025-03-01

{
  "bootDiagnostics": {},
  "isVMInStandbyPool": true,
  "hyperVGeneration": "V2",
  "statuses": [
    {
      "code": "ProvisioningState/succeeded",
      "level": "Info",
      "displayStatus": "Provisioning succeeded",
      "time": "2024-08-02T17:22:46.2955369+00:00"
    },
    {
      "code": "PowerState/deallocated",
      "level": "Info",
      "displayStatus": "VM deallocated"
    }
  ]
}
```

---

## Next steps
Review the most [frequently asked questions](standby-pools-faq.md) about standby pools for Virtual Machine Scale Sets.
