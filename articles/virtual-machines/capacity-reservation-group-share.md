---
title: Share a Capacity Reservation Group in Azure
description: Learn how to share a Capacity Reservation Group.
author: tferdou1
ms.author: tferdous
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 10/14/2025
ms.reviewer: jushiman, bidefore
ms.custom: template-how-to, devx-track-azurecli, devx-track-azurepowershell
---

# Share a Capacity Reservation Group (Preview)

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Uniform scale set :heavy_check_mark: Flexible scale sets

> [!IMPORTANT]
> This feature is currently in **Preview**. See the [Preview Terms of Use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability. 

On-demand Capacity Reservation Group (CRG) can be shared with other subscriptions. Using this option can make it easier to manage some common configuration needs: 

- **Reuse of capacity reserved for disaster recovery.** Reserved capacity is the primary means to obtain capacity assurance in another region or zone in the event of the primary region or zone becoming unavailable. Reservation sharing supports reuse of disaster recovery capacity by subscriptions, hosting less critical workloads such as development and testing, or subscriptions used to run job-oriented workloads. Capacity reuse can save on total capacity costs and drive more value from the reserved capacity.

- **Central management of capacity.** Quota requests and term commitments are often administered by a central operations team as part of cost management. Now, reserved capacity needs can be assessed and managed more centrally to align on capacity and cost management.

- **Separate security and capacity concerns.** Applications implemented with multiple subscriptions for security reasons can operate from a common pool of capacity. This pattern is common with service providers serving their own end customers.

- **More cost-effective scale-out with capacity assurance.** Applications that scale at different rates and times can share one pool of reserved capacity.

## How to share a Capacity Reservation Group

Sharing reserved capacity requires at least two subscriptions: 

- **Provider subscription** - the subscription that creates and hosts the Capacity Reservation Group and member Capacity Reservations. 
- **Consumer subscription** - another subscription that is granted access to the reserved capacity, obtaining the ability to deploy virtual machines (VMs) with the Capacity Reservation Service Level Agreement (SLA). 

A given Capacity Reservation Group can be shared with up to 100 consumer subscriptions. All the member Capacity Reservations in the Group will be accessible from consumer subscriptions. 

Configuring a Capacity Reservation Group sharing relationship has three steps: 
1. In the Consumer subscription, configure an On Demand capacity Reservation (ODCR) owner from the Producer subscription with the rights `Microsoft.Compute/capacityReservationGroups/share/action`.
2. In the Producer subscription, add the Consumer subscription ID to the Capacity Reservation Group *shared* list. See [Share a capacity Reservation Group](#share-a-capacity-reservation-group) to learn how to add a Consumer subscription to the sharing profile.
3. In the Producer subscription, configure at least one VM owner in the Consumer subscription with the following rights: 
	- `Microsoft.Compute/capacityReservationGroups/read`
	- `Microsoft.Compute/capacityReservationGroups/deploy`
	- `Microsoft.Compute/capacityReservationGroups/capacityReservations/read`
	- `Microsoft.Compute/capacityReservationGroups/capacityReservations/deploy`

If the ODCR owner and the VM owner already have either Owner or Contributor Azure role in both the Provider and Consumer subscriptions, then no further action is needed for granting share, read, and deploy permissions. To learn more on how to assign an Azure role, see [Role Assignment Steps](/azure/role-based-access-control/role-assignments-steps) or [Azure custom role](/azure/role-based-access-control/custom-roles)

Once complete, a VM owner in the Consumer subscription can enumerate the shared CRG. [View the list of Capacity Reservation Groups for a subscription](#view-the-list-of-capacity-reservation-groups-for-a-subscription) and deploy VMs by setting the `capacityReservationGroup` property on Virtual Machines or Virtual Machine Scale Sets. Keep in mind, the ability to modify the Capacity Reservation Group (CRG) remains only with the ODCR administrator in the producer subscription.

> [!NOTE]
> There are no extra charges for using the shared Capacity Reservation Group feature. Unused reservations are charged to the subscription that owns the reservation. VM usage is charged to the subscription that uses the capacity reservation as it does today. For details on how Reserved Instance (RI) applies to the feature, see [Use of Reserved Instances with shared Capacity Reservation Groups](#use-of-reserved-instances-with-shared-capacity-reservation-groups) section.

## Usage patterns 

The Provider subscription sharing a Capacity Reservation Group can allow: 
- Consumer subscriptions to access a specific Capacity Reservation Group in the Provider subscription
- Consumer subscription to access all Capacity Reservation Groups created in the Provider subscription
- All Consumer subscriptions in a specific Management Group to access the Capacity Reservation Group

> [!NOTE]
> Azure strongly recommends using one main Provider subscription to host Capacity Reservation Groups for each application, workload, or usage scope to share across other subscriptions. Creating Capacity Reservation Groups in many different subscriptions and then cross-sharing in a matrix fashion will create management challenges and lead to confusion at VM deployment.

## Prerequisites for sharing a Capacity Reservation Group 

- An ODCR owner in the Provider subscription must have sufficient rights to be able to share a CRG
- A VM owner in the Consumer subscription must have sufficient rights to be able to make deployments in capacity reservation (CR) in shared CRG
- A VM being deployed in the shared CRG must match the VM SKU, region, and zone if applicable 

## Limitations of sharing a Capacity Reservation Group

Limitations by design:
- Sharing works with an explicit list of target Consumer subscriptions. Azure doesn't support wildcard or tenant level sharing.
- A CRG can be shared with a maximum of 100 Consumer subscriptions.
- Sharing is done per Capacity Reservation Group, which grants access to all member Capacity Reservations. Individual Capacity Reservations can't be shared. To isolate specific Capacity Reservations, create multiple Capacity Reservation Groups and share only those Capacity Reservations that contain shared capacity. 
- By default, Capacity Reservation Group administrators in the subscription owning a Capacity Reservation Group can't modify VM instances deployed by other subscriptions. If such VM access is desired, more rights to VMs on the shared subscriptions must be granted separately. 
  
Limitations for Preview:
- Azure portal support isn't available; API and other Azure clients are available.  
- Reprovisioning of Virtual Machine Scale Set VMs using a shared Capacity Reservation Group isn't supported during a zone outage.
- There is a known issue of [Capacity Reservation Groups - List by Subscription ID](#capacity-reservation-groups---list-by-subscription-id) not giving the right response if there is no CRG created by the subscription making the `GET` call to list shared CRGs in the region. To get the correct response, ensure you have a local CRG created in the subscription making the API call in the same region where you would like to enumerate the shared CRGs. Alternatively, use the [Azure Resource Graph](#azure-resource-graph) query provided to get the list of CRGs shared with your subscription.

## Share a Capacity Reservation Group 

Capacity Reservation Groups are shared by adding Consumer subscriptions in the sharing profile of newly created or existing Capacity Reservation Groups. Once shared, the subscriptions which are part of the sharing profile can deploy VMs or Virtual Machine Scale Sets in the shared Capacity Reservation Group. 

### Add sharing profile to a new Capacity Reservation Group
Share a Capacity Reservation Group during creation by adding subscriptions in the sharing profile.

#### [API](#tab/api-1)
To share a Capacity Reservation group on creation, construct the following PUT request: 

```rest
PUT https://management.azure.com/subscriptions/{provider-subscription-id}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/capacityReservationGroups/myCapacityReservationGroup?api-version=2024-03-01 
``` 
    
In the request body, include the subscription ID to share the Capacity Reservation Group in the `sharing profile`:
    
```json
{
    "location": "westus",
    "tags": {
        "department": "finance"
    },
    "zones": [
        "1"
    ],
    "properties": {
        "sharingProfile": {
            "subscriptionIds": [{
                    "id": "/subscriptions/{consumer-subscription-id1}"
                }, {
                    "id": "/subscriptions/{consumer-subscription-id2}"
                }
            ]
        }
    }
}
``` 

This example is to create a shared CRG in zone 1 of Region West US.

To learn more, see [Capacity Reservation Groups - Create Or Update](/rest/api/compute/capacity-reservation-groups/create-or-update).
 
#### [CLI](#tab/cli-1)
Create a Capacity Reservation Group with a sharing profile using `az capacity reservation group create`. 

The following example creates a shared capacity reservation group in the West Europe location: 

 ```azurecli-interactive
 az capacity reservation group create 
 -n reservationGroupName 
 -g myResourceGroup
 --sharing-profile "subscriptions/{consumer-subscription-id1}" "subscriptions/{consumer-subscription-id2}" -l westEurope 
 ```

To learn more, go to Azure PowerShell command [AzCapacityReservation](/cli/azure/capacity/reservation/group).

#### [PowerShell](#tab/powershell-1)
Create a shared Capacity Reservation group with `New-AzCapacityReservationGroup`. The following example creates a shared Capacity Reservation group in the East US location. 

```powershell-interactive
New-AzCapacityReservationGroup 
-ResourceGroupName myRG
-Location eastus
-Name myCapacityReservationGroup
-SharingProfile "/subscriptions/{consumer-subscription-id1}", "/subscriptions/{consumer-subscription-id2}" 
```

To learn more, see Azure PowerShell command [New-AzCapacityReservation](/powershell/module/az.compute/new-azcapacityreservationgroup).

#### [ARM Template](#tab/armtemplatesample-1)
An [ARM template](/azure/azure-resource-manager/templates/overview) is a JavaScript Object Notation (JSON) file that defines the infrastructure and configuration for your project. The template uses declarative syntax. In declarative syntax, you describe your intended deployment without writing the sequence of programming commands to create the deployment. 

ARM templates let you deploy groups of related resources. In a single template, you can share a Capacity Reservation Group. Deploy templates through the Azure portal, Azure CLI, Azure PowerShell, or from continuous integration/continuous delivery (CI/CD) pipelines. 

The following ARM template creates a CRG shared with subscriptions `consumerSubscriptionID1` and `consumerSubscriptionID2`. Alternatively, you can remove the *zone* information if you would like to deploy a regional shared capacity reservation group.

If your environment meets the prerequisites and you're familiar with using ARM templates, use the following template:

 ```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "resourcename": {
            "defaultValue": "SharedZonalCRGResourceName",
            "type": "String"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Compute/capacityReservationGroups",
            "apiVersion": "2023-09-01",
            "name": "[parameters('resourcename')]",
            "location": "SharedZonalCRGRegionName",
             "zones": [
                "1",
                "2",
                "3"
            ],
            "properties": {
				"sharingProfile" :
				{
					"subscriptionIds" : [
                        {
                          "id": "/subscriptions/consumerSubscriptionID1"
                        },
                        {
                          "id": "/subscriptions/consumerSubscriptionID2"
                        }
					]
				}
			}
        }
    ]
}

```

--- 
<!-- The three dashes above show that your section of tabbed content is complete. -->

### Add sharing profile to an existing Capacity Reservation Group 
Add a sharing profile and share with subscriptions for an existing Capacity Reservation Group. 

#### [API](#tab/api-2)
To add a sharing profile to an existing CRG, construct the following PUT request.  

The following example adds sharing profile to an existing Capacity Reservation Group called `myCapacityReservationGroup` and is shared with three subscription IDs.

```rest
PUT https://management.azure.com/subscriptions/{provider-subscription-id}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/capacityReservationGroups/myCapacityReservationGroup?api-version=2024-03-01 
``` 
    
In the request body, include the consumer subscription IDs to share the Capacity Reservation Group in the `sharing profile`.
    
```json
{
    "location": "westus",
    "tags": {
        "department": "finance"
    },
    "zones": [
        "1"
    ],
    "properties": {
        "sharingProfile": {
            "subscriptionIds": [
                {
                    "id": "/subscriptions/{consumer-subscription-id1}"
                },
                {
                    "id": "/subscriptions/{consumer-subscription-id2}"
                },
                {
                    "id": "/subscriptions/{consumer-subscription-id3}"
                }
            ]
        }
    }
}
```

To learn more, see [Capacity Reservation Groups - Create Or Update](/rest/api/compute/capacity-reservation-groups/create-or-update).
 
#### [CLI](#tab/cli-2)
Update a Capacity Reservation Group with a sharing profile using `az capacity reservation group update` with the following command: 

 ```azurecli-interactive
 az capacity reservation group update
 -n ReservationGroupName 
 -g myResourceGroup
 --sharing-profile "subscriptions/{consumer-subscription-id1}"
 ```

To learn more, go to Azure PowerShell command [AzCapacityReservationGroupUpdate](/cli/azure/capacity/reservation/group).

#### [PowerShell](#tab/powershell-2)
Update a Capacity Reservation Group with a sharing profile using `Update-AzCapacityReservationGroup`.

The following example is to update an existing CRG named `myCapacityReservationGroup` with a sharing profile having one subscription ID and later adding a new subscription ID. 

```powershell-interactive
Update-AzCapacityReservationGroup
-ResourceGroupName myRG
-Name myCapacityReservationGroup
-SharingProfile "/subscriptions/{consumer-subscription-id1}", "/subscriptions/{consumer-subscription-id2}" 
```

To learn more, see Azure PowerShell command [Update-AzCapacityReservation](/powershell/module/az.compute/update-azcapacityreservationgroup).

--- 
<!-- The three dashes above show that your section of tabbed content is complete. -->

## Stop sharing a Capacity Reservation Group

The Capacity Reservation Group owner can stop sharing a Capacity Reservation Group with a subscription or all subscriptions at any time. 

Unsharing of a Capacity Reservation Group with a cross subscription ID is done while a VM or Virtual Machine Scale Set from cross subscriptions remains associated with the Capacity Reservation Group. The associated resources get SLA until deallocation or reallocation. 

Once unsharing happens, any VM or scale set previously associated to the CRG would fail to associate upon deallocation or reallocation. Avoid this failure by removing the association from the CRG.

### Unsharing a Capacity Reservation Group with a subscription
To unshare a Capacity Reservation Group with a subscription from the sharing profile, remove the subscription from the sharing profile. 

In the following examples, a Capacity Reservation Group was shared with Consumer Subscription ID 1, Consumer Subscription ID 2, and Consumer Subscription ID 3.  

#### [API](#tab/api-3)
To remove a subscription from the sharing profile of an existing Capacity Reservation Group, construct the following PUT request.  

The following example removes Consumer Subscription ID 3 from the sharing profile to an existing Capacity Reservation Group called `myCapacityReservationGroup` that was previously shared with three subscription IDs. 

```rest
PUT https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/capacityReservationGroups/myCapacityReservationGroup?api-version=2024-03-01
``` 
    
In the request body, include the subscription ID to share the Capacity Reservation Group in the `sharing profile`.
    
```json
{
    "location": "westus",
    "tags": {
        "department": "finance"
    },
    "zones": [
        "1"
    ],
    "properties": {
        "sharingProfile": {
            "subscriptionIds": [
                {
                    "id": "/subscriptions/{consumer-subscription-id1}"
                },
                {
                    "id": "/subscriptions/{consumer-subscription-id2}"
                }
            ]
        }
    }
}
```

To learn more, see [Capacity Reservation Groups - Create Or Update](/rest/api/compute/capacity-reservation-groups/create-or-update).
 
#### [CLI](#tab/cli-3)
Remove a subscription ID from the sharing profile of an existing Capacity Reservation Group using `az capacity reservation group update`. 

The following example removes two consumer subscription IDs from the sharing profile of an existing Capacity Reservation that was shared with three consumer subscription IDs: 

 ```azurecli-interactive
 az capacity reservation group update
 -n ReservationGroupName 
 -g myResourceGroup
 --sharing-profile "subscriptions/{consumer-subscription-id1}"
 ```

To learn more, go to [AzCapacityReservationGroupUpdate](/cli/azure/capacity/reservation/group).

#### [PowerShell](#tab/powershell-3)
Remove a consumer subscription ID from the sharing profile of an existing CRG using `Update-AzCapacityReservationGroup`.  

The following example is to remove two subscription IDs from the sharing profile of an existing capacity reservation group named `myCapacityReservationGroup` that was shared with three consumer subscription IDs. 

```powershell-interactive
Update-AzCapacityReservationGroup
-ResourceGroupName myRG
-Name myCapacityReservationGroup
-SharingProfile "/subscriptions/{consumer-subscription-id1}"
```

To learn more, see [Update-AzCapacityReservation](/powershell/module/az.compute/update-azcapacityreservationgroup).

--- 
<!-- The three dashes above show that your section of tabbed content is complete. -->

### Unsharing a Capacity Reservation Group with all subscriptions
To unshare a Capacity Reservation Group with all consumer subscriptions, remove all subscriptions from the sharing profile. 

#### [API](#tab/api-4)
To remove all consumer subscriptions from the sharing profile of an existing Capacity Reservation group, construct the following PUT request.  

The following example removes all consumer subscriptions from the sharing profile to an existing CRG called `myCapacityReservationGroup`.

```rest
PUT https://management.azure.com/subscriptions/{provider-subscription-id}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/capacityReservationGroups/myCapacityReservationGroup?api-version=2024-03-01 
``` 
    
In the request body, include the subscription ID to share the Capacity Reservation Group in the `sharing profile`.
    
```json
{
    "location": "westus",
    "tags": {
        "department": "finance"
    },
    "zones": [
        "1"
    ],
    "properties": {
        "sharingProfile": {
            "subscriptionIds": []
        }
    }
}
```

To learn more, see [Capacity Reservation Groups - Create Or Update](/rest/api/compute/capacity-reservation-groups/create-or-update).

#### [ARM Template](#tab/armtemplatesample-4)
An [ARM template](/azure/azure-resource-manager/templates/overview) is a JavaScript Object Notation (JSON) file that defines the infrastructure and configuration for your project. The template uses declarative syntax. In declarative syntax, you describe your intended deployment without writing the sequence of programming commands to create the deployment. 

ARM templates let you deploy groups of related resources. In a single template, you can share a Capacity Reservation Group. Deploy templates through the Azure portal, Azure CLI, Azure PowerShell, or from continuous integration/continuous delivery (CI/CD) pipelines. 

This ARM template unshares a CRG shared with all Consumer subscriptions. If your environment meets the prerequisites and you're familiar with using ARM templates, use the following template:

 ```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "resourcename": {
            "defaultValue": "SharedCRGResourceName",
            "type": "String"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Compute/capacityReservationGroups",
            "apiVersion": "2023-09-01",
            "name": "[parameters('resourcename')]",
            "location": "SharedCRGRegionName",
            "properties": {
				"sharingProfile" :
				{
					"subscriptionIds" : []
				}
			}
        }
    ]
}

```

#### [CLI](#tab/cli-4)
Remove all Consumer subscription IDs from the sharing profile of an existing Capacity Reservation Group using `az capacity reservation group update`.

The following example removes all Consumer subscription IDs from sharing profile of an existing Capacity Reservation Group that was previously shared with one or more Consumer subscription IDs:

 ```azurecli-interactive
 az capacity reservation group update
 -n reservationGroupName 
 -g myResourceGroup
 --sharing-profile 
 ```

To learn more, [AzCapacityReservationGroup](/cli/azure/capacity/reservation/group).

#### [PowerShell](#tab/powershell-4)
Remove all Consumer subscription IDs from the sharing profile of an existing CRG using `Update-AzCapacityReservationGroup`.

The following example removes all Consumer subscription IDs from the sharing profile of an existing CRG named `myCapacityReservationGroup` that was shared with one or more consumer subscription IDs.  

```powershell-interactive
Update-AzCapacityReservationGroup 
-ResourceGroupName myRG
-Name myCapacityReservationGroup
-SharingProfile "" 
```

To learn more, see [Update-AzCapacityReservation](/powershell/module/az.compute/update-azcapacityreservationgroup).

--- 
<!-- The three dashes above show that your section of tabbed content is complete. -->

## Delete a shared Capacity Reservation Group

See [Modify a Capacity Reservation](/azure/virtual-machines/capacity-reservation-modify) for deletion guidance.

- Users with sufficient rights can delete the shared Capacity Reservation Group.
- Azure allows a Capacity Reservation Group to be deleted when all the member capacity reservations are deleted.
- Azure allows a Capacity Reservation to be deleted when no VMs are associated to the Capacity Reservation.
- Unsharing of a CRG with a shared subscription happens as part of the shared Capacity Reservation Group deletion process.

## Using a shared Capacity Reservation Group

Once the Capacity Reservation Group is successfully shared, users with sufficient rights from a Consumer subscription can deploy a VM or Virtual Machine Scale Set in a shared Capacity Reservation Group.

> [!NOTE]
> The Provider subscription deploying the shared CRG will need to hold its own quota for deploying the CRG. The Consumer subscription making a deployment in the shared reservation group will need to hold its own quota.

### Availability zone mapping with shared zonal Capacity Reservation Groups 
The availability zones visible to each Azure subscription represent a logical mapping to underlying groups of physical data centers that constitute a physical zone. To promote efficient distribution of resources across availability zones, each Azure subscription gets a random logical to physical mapping of zones. For example, subscription A (Provider subscription) and subscription B (Consumer subscription) can have different logical mappings. 

Consider the following example: 

If subscription A deploys to AZ1, the deployment goes to physical zone 1. But if subscription B deploys to AZ1, the deployment goes to physical zone 2:
:::image type="content" source="./media/capacity-reservation-group-share/capacity-reservation-group-a-b-mapping.png" alt-text="A screenshot showing subscription A and subscription B having different physical to logical zone mapping.":::

Now consider a Capacity Reservation deployed by Subscription A to logical AZ1. The result is reserved capacity in physical zone 1.  
:::image type="content" source="./media/capacity-reservation-group-share/capacity-reservation-group-a-reservation.png" alt-text="A screenshot showing subscription A creating a capacity reservation in logical zone 1.":::

If subscription B deployed a VM to logical AZ1 using the shared Capacity Reservation Group, the deployment would fail because subscription B AZ1 resolves to physical zone 2, and there's no reserved capacity in physical zone 2. 
:::image type="content" source="./media/capacity-reservation-group-share/capacity-reservation-group-a-b-not-aligned.png" alt-text="A screenshot showing subscription A capacity reservation and subscription B VM are in two different logical zones resulting in failure.":::

The solution is to reconcile the logical to physical mappings for subscription A and subscription B. Subscription B should deploy to AZ2 to access reserved capacity in physical zone 1. 
:::image type="content" source="./media/capacity-reservation-group-share/capacity-reservation-group-a-b-aligned.png" alt-text="A screenshot showing subscription A capacity reservation and subscription B VM in same logical zone resulting in success.":::

When using a shared Capacity Reservation Group with zones, all subscriptions have the logical view of availability zones from the CRG subscription, which is likely different than the logical view of availability zones from the target subscription. When deploying a VM or scale set to a shared CRG, you must remap the zones. To ensure that the Azure resources are efficiently distributed across Availability zones in a Region, each subscription has independent logical zone mapping for Availability zones. This means that logical to physical zone mapping may or may not be the same across subscriptions.

To check the Physical Zone and Logical Zone mapping for your subscription, see [Subscriptions - List Locations - REST API (Azure Resource Management)](/rest/api/resources/subscriptions/list-locations). For more information, see [Physical and Logical availability zones](/azure/reliability/availability-zones-overview?tabs=azure-cli#physical-and-logical-availability-zones).

### Use of Reserved Instances with shared Capacity Reservation Groups 
Sharing a Capacity Reservation Group doesn't alter the scope of any Reserved Instances or Savings Plans. If either the CRG or the VM is deployed from a scope not covered by prepaid discounts, the pay-as-you-go price is charged. 

To share Reserved Instance discounts between a Capacity Reservation Group and VMs deployed from a Consumer subscription, the Provider subscription and the Consumer subscription must share the same Reserved Instance scope. If the two subscriptions share an enrollment or a management group, then Reserved Instances set to the corresponding scope works automatically. 

#### Associate or create a single Virtual Machine with shared Capacity Reservation Group
Single Virtual Machine can be deployed in shared Capacity Reservation Group using PowerShell, CLI, or REST API. See [Associate a virtual machine to a Capacity Reservation group](/azure/virtual-machines/capacity-reservation-associate-vm).

#### Remove a single Virtual Machine from Shared Capacity Reservation Group 
Single Virtual Machine can be removed from Shared Capacity Reservation Group using PowerShell, CLI, or REST API. See [Remove a virtual machine association from a Capacity Reservation group](/azure/virtual-machines/capacity-reservation-remove-vm).

#### Associate or create a Virtual Machine Scale Set with shared Capacity Reservation Group 
Virtual Machine Scale Set in Flexible and Uniform orchestration mode can be deployed in shared Capacity Reservation Group using PowerShell, CLI, or REST API. To learn more, see [Associate a scale set -Flexible](/azure/virtual-machines/capacity-reservation-associate-virtual-machine-scale-set-flex) and [Associate a scale set - Uniform](/azure/virtual-machines/capacity-reservation-associate-virtual-machine-scale-set).

#### Remove Virtual Machine Scale Set from Shared Capacity Reservation Group
Virtual Machine Scale Set in Flexible and Uniform orchestration mode can be removed from shared Capacity Reservation Group using PowerShell, CLI, or REST API. To learn more, see [Remove a scale set](/azure/virtual-machines/capacity-reservation-remove-virtual-machine-scale-set).

## View shared Capacity Reservation Group

Once a Capacity Reservation Group is shared successfully, the reservations are immediately available for use with single Virtual Machines and Virtual Machine Scale Sets. 

View the subscription IDs the Capacity Reservation Group are shared with from the sharing profile. 

To learn more, see [Create a Capacity Reservation](/azure/virtual-machines/capacity-reservation-create).

### [API](#tab/api-5)

```rest
GET https://management.azure.com/subscriptions/{provider-subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/capacityReservationGroups/{capacityReservationGroupName}?api-version=2023-07-01
``` 
    
```json
{
    "name": "SharedCRG",
    "id": "/subscriptions/{Provider-subscriptionID1}/resourceGroups/{MyRG}/providers/Microsoft.Compute/capacityReservationGroups/SharedCRG",
    "type": "Microsoft.Compute/capacityReservationGroups",
    "location": "eastus2",
    "properties": {
        "capacityReservations": [{
                "id": "/subscriptions/{Provider-subscriptionID1}/resourceGroups/{MyRG}/providers/Microsoft.Compute/capacityReservationGroups/{SharedCRG}/capacityReservations/{CR1}"
            },
        ],
        "sharingProfile": {
            "subscriptionIds": [{
                    "id": "/subscriptions/{consumer-subscriptionID2}"
                }
            ]
        },
        "provisioningState": "Succeeded"
    }
}
```

To learn more, see [Capacity Reservation Group-GET](/rest/api/compute/capacity-reservation-groups/get).
 
### [CLI](#tab/cli-5)

See [az capacity reservation group show](/cli/azure/capacity/reservation/group).

### [PowerShell](#tab/powershell-5)

See [Get-AzCapacityReservationGroup](/powershell/module/az.compute/get-azcapacityreservationgroup).

--- 
<!-- The three dashes above show that your section of tabbed content is complete. -->

## View the list of Capacity Reservation Groups for a subscription

The list of all Capacity Reservation Groups that are created locally or shared with by other subscriptions can be viewed for a given subscription. Extra parameter `resourceIdsonly` needs to be passed to view the shared Capacity Reservation Groups.

### Capacity Reservation Groups - List by Subscription ID
By default, obtaining a list of Capacity Reservation Groups returns only those owned by the subscription. To add the Capacity Reservation Groups shared to the subscription, the additional parameter of `resourceIdsOnly` must be set to `sharedwithsubscription`.

#### [API](#tab/api-6)
Enable fetching Resource IDs for all CRG resources shared with the subscription:  

```rest
GET https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Compute/capacityReservationGroups?api-version=2023-09-01&resourceIdsOnly=sharedwithsubscription 
``` 

Enable fetching Resource IDs for all CRG resources shared with the subscription and created in the subscription: 

```rest
GET https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Compute/capacityReservationGroups?api-version=2023-09-01&resourceIdsOnly=All 
``` 

Enable fetching Resource IDs for all CRG resources created in the subscription: 

```rest
GET https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Compute/capacityReservationGroups?api-version=2023-09-01&resourceIdsOnly=CreatedInSubscription 
``` 
    
```json
{
    "value": [
        {
            "id": "/subscriptions/{subscriptionId1}/resourceGroups/{resourceGroupName1} /providers/Microsoft.Compute/capacityReservationGroups/{CapacityReservationGroupName1} ",
            "type": "Microsoft.Compute/capacityReservationGroups",
            "location": "EastUS2"
        },
        {
            "id": "/subscriptions/{subscriptionId2}/resourceGroups/{resourceGroupName2} /providers/Microsoft.Compute/capacityReservationGroups/{CapacityReservationGroupName2} ",
            "type": "Microsoft.Compute/capacityReservationGroups",
            "location": "EastUS2"
        }
    ]
}
```

To learn more, see [Capacity Reservation Group - List by subscription](/rest/api/compute/capacity-reservation-groups/list-by-subscription).
 
#### [CLI](#tab/cli-6)
Enable fetching Resource IDs for all CRG resources shared with the subscription and created in the subscription:

 ```azurecli-interactive
 az capacity reservation group list
 --resource-ids-only all 
 ```

Enable fetching Resource IDs for all CRG resources created in the subscription:

 ```azurecli-interactive
 az capacity reservation group list
 --resource-ids-only CreatedInSubscription
 ```

Enable fetching Resource IDs for all CRG resources shared with the subscription:

 ```azurecli-interactive
 az capacity reservation group list
 --resource-ids-only SharedWithSubscription
 ```

To learn more, go to [AzCapacityReservationGroupList](/cli/azure/capacity/reservation/group).

#### [PowerShell](#tab/powershell-6)
Enable fetching Resource IDs for all CRG resources shared with the subscription and created in the subscription:

```powershell-interactive
Get-AzCapacityReservationGroup
-ResourceIdsOnly All 
```

Enable fetching Resource IDs for all CRG resources created in the subscription:

```powershell-interactive
Get-AzCapacityReservationGroup
-ResourceIdsOnly CreatedInSubscription 
```

Enable fetching Resource IDs for all CRG resources shared with the subscription:

```powershell-interactive
Get-AzCapacityReservationGroup
-ResourceIdsOnly SharedWithSubscription
```

To learn more, see Azure PowerShell command [Update-AzCapacityReservation](/powershell/module/az.compute/update-azcapacityreservationgroup).

--- 
<!-- The three dashes above show that your section of tabbed content is complete. -->

### Azure Resource Graph
Use the Azure Resource Graph to view the list of all Capacity Reservation Groups that are shared with created locally within a given subscription.

#### [Portal](#tab/portal-7)
To view the CRG list, go to [Azure Resource Graph Explorer](https://ms.portal.azure.com/#view/HubsExtension/ArgQueryBlade) and try this query.

Enable fetching Resource IDs for all CRG resources shared with subscription ID 1 and created within subscription ID 1: 

```kusto
resources
|where type == "microsoft.compute/capacityreservationgroups"
|where properties["sharingProfile"] contains "{subscriptionId1}" or subscriptionId == "{subscriptionId1}"
|project name, id
``` 

Enable fetching Resource IDs for all CRG resources shared with subscription ID 1: 

```kusto
resources
|where type == "microsoft.compute/capacityreservationgroups"
|where properties["sharingProfile"] contains "{subscriptionId}"
|project name, id
``` 
 
#### [CLI](#tab/cli-7)

Enable fetching Resource IDs for all CRG resources shared with subscription ID 1 and created in subscription ID 1:

 ```azurecli-interactive
az graph query -q "resources| where type == 'microsoft.compute/capacityreservationgroups'| where tostring(properties['sharingProfile']) contains '{subscriptionId1}' or subscriptionId == '{subscriptionId1}'| project name, id" 
 ```

Enable fetching Resource IDs for all CRG resources shared with subscription ID 1:

 ```azurecli-interactive
az graph query -q "resources| where type == 'microsoft.compute/capacityreservationgroups'| where tostring(properties['sharingProfile']) contains '{subscriptionId1}'| project name, id"
 ```

#### [PowerShell](#tab/powershell-7)

Enable fetching Resource IDs for all CRG resources shared with subscription ID 1 and created in subscription ID 1:

```powershell-interactive
$query = @"
resources
| where type == "microsoft.compute/capacityreservationgroups"
| where tostring(properties["sharingProfile"]) contains "{subscriptionId1}" or subscriptionId == "{subscriptionId1}"
| project name, id
"@
$result = Search-AzGraph -Query $query
$result
```

Enable fetching Resource IDs for all CRG resources shared with subscription ID 1:

```powershell-interactive
$query = @"
resources
| where type == "microsoft.compute/capacityreservationgroups"
| where tostring(properties["sharingProfile"]) contains "{subscriptionId1}"
| project name, id
"@

$result = Search-AzGraph -Query $query
$result
```

--- 
<!-- The three dashes above show that your section of tabbed content is complete. -->

## Next steps

> [!div class="nextstepaction"]
> [Learn how to associate a VM to a capacity reservation group](capacity-reservation-associate-vm.md)



