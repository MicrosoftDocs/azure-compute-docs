``
### **In this article**
- [Overview](https://msazure.visualstudio.com/AzureWiki/_wiki/wikis/AzureWiki.wiki?wikiVersion=GBwikiMaster&pagePath=/Azure%20Container%20Instances/NGroups%20%252D%20Customer%20Onboarding&pageId=709513&_a=edit&anchor=**overview**)
- [Concepts](https://msazure.visualstudio.com/AzureWiki/_wiki/wikis/AzureWiki.wiki?wikiVersion=GBwikiMaster&pagePath=/Azure%20Container%20Instances/NGroups%20%252D%20Customer%20Onboarding&pageId=709513&_a=edit&anchor=**concepts**)

1. [Container Group Profile](https://msazure.visualstudio.com/AzureWiki/_wiki/wikis/AzureWiki.wiki?wikiVersion=GBwikiMaster&pagePath=/Azure%20Container%20Instances/NGroups%20%252D%20Customer%20Onboarding&pageId=709513&_a=edit&anchor=container-group-profile-(cg-profile))
2. [NGroupsAPI](https://msazure.visualstudio.com/AzureWiki/_wiki/wikis/AzureWiki.wiki?wikiVersion=GBwikiMaster&pagePath=/Azure%20Container%20Instances/NGroups%20%252D%20Customer%20Onboarding&pageId=709513&_a=edit&anchor=ngroups-api)
3. [Updating a NGroup](https://msazure.visualstudio.com/AzureWiki/_wiki/wikis/AzureWiki.wiki?wikiVersion=GBwikiMaster&pagePath=/Azure%20Container%20Instances/NGroups%20%252D%20Customer%20Onboarding&pageId=709513&_a=edit&anchor=updating-a-ngroup)
4. [NGroups' billing model](https://msazure.visualstudio.com/AzureWiki/_wiki/wikis/AzureWiki.wiki?wikiVersion=GBwikiMaster&pagePath=/Azure%20Container%20Instances/NGroups%20%252D%20Customer%20Onboarding&pageId=709513&_a=edit&anchor=ngroups%27-billing-model)

- [Try NGroups](https://msazure.visualstudio.com/AzureWiki/_wiki/wikis/AzureWiki.wiki?wikiVersion=GBwikiMaster&pagePath=/Azure%20Container%20Instances/NGroups%20%252D%20Customer%20Onboarding&pageId=709513&_a=edit&anchor=**try-ngroups%3A**)

1. [Prerequisites for working on NGroups](https://msazure.visualstudio.com/AzureWiki/_wiki/wikis/AzureWiki.wiki?wikiVersion=GBwikiMaster&pagePath=/Azure%20Container%20Instances/NGroups%20%252D%20Customer%20Onboarding&pageId=709513&_a=edit&anchor=prerequisites-for-working-on-ngroups%3A)
2. [ARM template samples](https://msazure.visualstudio.com/AzureWiki/_wiki/wikis/AzureWiki.wiki?wikiVersion=GBwikiMaster&pagePath=/Azure%20Container%20Instances/NGroups%20%252D%20Customer%20Onboarding&pageId=709513&_a=edit&anchor=arm-template-samples%3A)

- [How-to-Guide](https://msazure.visualstudio.com/AzureWiki/_wiki/wikis/AzureWiki.wiki?wikiVersion=GBwikiMaster&pagePath=/Azure%20Container%20Instances/NGroups%20%252D%20Customer%20Onboarding&pageId=709513&_a=edit&anchor=**how-to-guide%3A**)

- [Swagger spec](https://github.com/saprakas/azure-rest-api-specs/blob/main/specification/containerinstance/resource-manager/Microsoft.ContainerInstance/preview/2024-09-01-preview/containerInstance.json)

1. [How to create a regional (zonal/non-zonal) NGroups?](https://msazure.visualstudio.com/AzureWiki/_wiki/wikis/AzureWiki.wiki?wikiVersion=GBwikiMaster&pagePath=/Azure%20Container%20Instances/NGroups%20%252D%20Customer%20Onboarding&pageId=709513&_a=edit&anchor=how-to-create-a-regional-(zonal/non-zonal)-ngroups%3F)
2. [Create NGroup CGs with a prefix:](https://msazure.visualstudio.com/AzureWiki/_wiki/wikis/AzureWiki.wiki?wikiVersion=GBwikiMaster&pagePath=/Azure%20Container%20Instances/NGroups%20%252D%20Customer%20Onboarding&pageId=709513&_a=edit&anchor=**create-ngroup-cgs-with-a-prefix%3A**)
3. [Create NGroup with both SystemAssigned and UserAssigned managed identity:](https://msazure.visualstudio.com/AzureWiki/_wiki/wikis/AzureWiki.wiki?wikiVersion=GBwikiMaster&pagePath=/Azure%20Container%20Instances/NGroups%20%252D%20Customer%20Onboarding&pageId=709513&_a=edit&anchor=**create-ngroup-with-both-systemassigned-and-userassigned-managed-identity%3A**)
4. [If I delete some CGs from the NGroup, can the NGroup rebuild itself with new CGs to maintain its desired count?](https://msazure.visualstudio.com/AzureWiki/_wiki/wikis/AzureWiki.wiki?wikiVersion=GBwikiMaster&pagePath=/Azure%20Container%20Instances/NGroups%20%252D%20Customer%20Onboarding&pageId=709513&_a=edit&anchor=**if-i-delete-some-cgs-from-the-ngroup%2C-can-the-ngroup-rebuild-itself-with-new-cgs-to-maintain-its-desired-count%3F**)
5. [How do I get the CG name, NGroup Id and other metadata propagated down into the container](https://msazure.visualstudio.com/AzureWiki/_wiki/wikis/AzureWiki.wiki?wikiVersion=GBwikiMaster&pagePath=/Azure%20Container%20Instances/NGroups%20%252D%20Customer%20Onboarding&pageId=709513&_a=edit&anchor=**how-do-i-get-the-cg-name%2C-ngroup-id-and-other-metadata-propagated-down-into-the-container%3F**)



## **Overview** 

When deploying containerized applications to the cloud, customers face a decision of managing their own infrastructure and orchestrator or choosing to deploy to a prescriptive serverless platform. Running large-scale applications on the cloud with containerized workloads is difficult due to a customer requiring expertise in infrastructure management and container grouping and management platforms. 

NGroups is an easy to use and manage at scale container offering that reduces customer’s infrastructure management responsibilities while still providing the flexibility to configure scaling rules, networking, and security policies. NGroups provides customers with an environment that is configured to run containerized applications at scale with promises such as Maintaining N instances, Rolling Upgrades, High Availability with Availability Zones (AZs), auto-scaling (coming soon), Confidentiality, Load Balancing setup, AZ Rebalancing (Zone Any) etc. Customers need not deploy VMs or orchestrate complex infrastructure to get their application up and running. 

To provide customers with the flexibility to configure NGroups to meet their organizational needs, NGroups integrates well with many Azure resources such as virtual networks, load balancers, Azure Managed Identity, storage accounts etc. This empowers customers to focus on their application and business logic, while Azure takes care of the rest of infrastructure management. 

For 1P customers using ACI, NGroups is the Azure Core Containers standards platform that offers capabilities to keep containers secure, highly available, and compliant with Azure standards. Onboarding to NGroups helps eliminate recurring bookkeeping of compliance checklists for these customers, with the platform natively applying all defined standards on NGroups resources. 

Examples of great workloads to run on top of NGroups include batch applications, CI/CD pipelines, ETL pipelines, and custom orchestrators that use Azure Container Instances (ACI) Container Groups as a unit of compute. 

 

ACI provides a single instance of a Hyper-V isolated version container group on demand. ACI does not provide concepts such as scaling, load balancing, rolling upgrades, spreading instances across availability zones out of the box. This is the best option for customers that simply want to deploy a container quickly, and do not need built-in support for these more advanced concepts. 

 

NGroups API are extension to ACI, which allows customers to quickly deploy a group of CGs and provides support for more complex scenarios. Customers can choose to use ACI if they have a single task, they want to run quickly without needing to consider scale. As their application needs grow, they can easily move their instance to a NGroups. ACI and NGroups will coexist analogous to the needs of customers using VMs with VMSS. 

## **Concepts** 

### Container Group Profile (CG Profile) 

When customers create ACI CG resources at scale, they usually end up copy-pasting the relevant properties of the CG (like the container images, the restart policy etc.) in each of their CGs. This copy-paste results in duplication and management overhead during the lifetime of these CGs.  

To alleviate this, we have a CG Profile. This serves as a “template” for building out similar looking CGs.  

The CG profile is another ARM resource. It is of type containerGroupProfiles. It contains the properties common to all CGs.  

Below are some properties that can be specified in a CG profile: 

- osType. E.g., Linux, Windows 

- containers. Image name, memory, CPU etc. 

- restartPolicy 

- ipAddress protocol and internal port 

- shutdownGracePeriod 

- timeToLive 

And below is a sample CG profile:

```
{ 

    "location": "{{location}}", 

    "properties": { 

        "sku": "Standard", 

        "containers": [ 

            { 

                "name": "container1", 

                "properties": { 

                    "image": "nginx", 

                    "ports": [ 

                        { 

                            "protocol": "TCP", 

                            "port": 80 

                        } 

                    ], 

                    "resources": { 

                        "requests": { 

                            "memoryInGB": 2.0, 

                            "cpu": 1.0 

                        } 

                    } 

                } 

            } 

        ], 

        "restartPolicy": "Always", 

        "shutdownGracePeriod": "PT1H", 

        "ipAddress": { 

            "ports": [ 

                { 

                    "protocol": "TCP", 

                    "port": 80 

                } 

            ], 

            "type": "Public", 

            "autoGeneratedDomainNameLabelScope": "Unsecure" 

        }, 

        "timeToLive": "PT1H", 

        "osType": "Linux" 

    }     

}
```
A NGroups will simply reference this CG profile ARM resource and use that to create N instances of similar looking CGs.  

Below is an example of how a NGroups references a CG profile:
 
```
{ 

    "location": "{{location}}", 

    "properties": { 

        "elasticProfile": { 

            "desiredCount": 100 // specifies how many CGs to stamp out 

        }, 

        "containerGroupProfiles": [ 

            { 

                "resource": { 

                    "id": "/subscriptions/{{subId}}/resourceGroups/{{rgName}}/providers/Microsoft.ContainerInstance/containerGroupProfiles/{{cgProfile1}}" 

 

                } 

            } 

        ] 

    } 

}
```
 The benefit of this approach is: 

1. Multiple NGroups related to your application can refer to the same CG profile.  This avoids duplication of a similar “profile” across NGroups.
E.g., in VMSS, each VMSS must define its “VM profile” inline in the VMSS schema. Admins/operators can update the VM profile of one VMSS, but forget to update the other VMSS, thus introducing subtle bugs. In NGroups, since all the NGroups can share the same CG profile, it guarantees consistency across NGroups for your application.  

 

2. In future, a single ACI CG could also be created from a CG profile. This allows us to rapidly move from prototype to production mode. E.g., a customer creates a single CG using the CG profile. They work on this CG to get their container images working. Then they simply create a NGroups using the same CG profile to massively scale out.

Since the CG profile is an ARM resource, it has its own ARM APIs. A CG profile needs to be created **before** creating a NGroups resource.

### NGroups API
This references a CG profile and adds additional orchestration related properties and capabilities. E.g.,  

- The desired count of CGs to scale out by 

- The VNET to connect all the CGs to while scaling out 

- The Load Balancer or Application Gateway to connect the CGs

This in turn invokes the ACI ARM APIs to create (and subsequently manage) each CG. Since it uses the same ARM APIs, there is no difference between the CGs created by NGroups and the CGs created directly by the customer. They have the exact same API experience.

### Updating a NGroup
As requirements change, we would need to keep updating our NGroups and its CGs. There are two update modes with which we can update a NGroups – **Manual** (default option) and **Rolling**. 

Consider a simple example of updating a CG profile reference from _cgprofile1_ to _cgprofile2_:

- In Manual mode, we update the reference to cgprofile2 and issue an UPDATE NGroups command:

NGroups stores this new CG profile reference. But it won’t update existing CGs with this reference. Existing CGs are currently running and there is no impact on them. However, when NGroups is scaled out, then new CGs will be created with cgprofile2. 

- How do we update existing CGs with cgprofile2?  

For this, we issue a ‘manual update’ command with an explicit list of CGs we want to update. This command only updates the CGs specified in its list. Updating the CG involves calling the ACI’s PUT CG API. The CGs not specified in this list will continue to run with cgprofile1. 

This mode gives us flexibility to update CGs selectively and provides full control over impact on production workloads.  

In **Rolling** mode, when we update the reference to cgprofile2 and issue an UPDATE NGroups command, existing CGs are updated with cgprofile2. The update to existing CGs happen in small batches (and not all at once). This ensures that there is a minimal impact on our workload since only a small percentage of CGs may be unavailable during the update.  

We can configure the batch size and other related rolling update mode settings with the NGroups API.

### NGroups' Billing model  

There is no cost associated with just NGroups or its capabilities. 

Customers only need to pay for the underlying resources NGroups creates or manages. E.g., ACI and other optional paid services (Like App Gateway, Public IP, etc.). Please refer to the Azure public documentation for the [cost of ACI](https://azure.microsoft.com/en-us/pricing/details/container-instances/) and each of these paid services.

## **Try NGroups:**

### Prerequisites for working on NGroups:

NGroups is currently under private preview and is only available on Canary regions. Currently supported API version is **2024-09-01-preview**.

1. Send request to NGroups Dev team to register your subscriptions under given AFECs: (corecontainersorg@microsoft.com or raise it under Partner channel) 
-    Microsoft.ContainerInstace/NGroupsPreview 
- Microsoft.ContainerInstance/NGroupsPreviewCanary 
2. Once the afec flags are applied to the subscription, run the following command to register the resourceProvider on your environment:
`az provider register -n Microsoft.ContainerInstance` 

3. **HIGHLY_IMP:** 
Since NGroups is currently under private preview in Canary, hence **the customer’s dev environment needs to target Canary regions**. Customers can follow below guideline to change environment target: [Targeting ARM Region using PowerShell | ARM Wiki (eng.ms)](https://eng.ms/docs/products/arm/troubleshooting/dri_tips/targeting_arm_region_pshell).


|**IMP**: Please use the api-version - 2024-09-01-preview for canary preview|
|--|


[Swagger Spec](https://github.com/saprakas/azure-rest-api-specs/blob/main/specification/containerinstance/resource-manager/Microsoft.ContainerInstance/preview/2024-09-01-preview/containerInstance.json)


If these prerequisites are not met: requests will fail, and the nGroups resource type will not be recognized. 

### ARM template samples:

Create CG Profile: [ContainerGroupProfile-Sample.json](https://microsoft.sharepoint-df.com/:u:/t/CoreContainersOrg/ESNVt_BUgyhImgal7jUb6AwB6oTKDmjMVxFFs3xZ0JvPjg?e=LIeFnO)

Create Zonal NGroups with CGProfile: [NGroups-Zonal-Sample.json](https://microsoft.sharepoint-df.com/:u:/t/CoreContainersOrg/EbLBbNYGg1FEt0KQjr4Hw-0B8OVm5LRCAMuwFBOELOJEHw?e=noHok4)

NGroups are not visible on the portal as of today, but you can check CGs being created on the portal under the resourceGroup. Customer can also check NGroups resourceId under CG JSON view in orchestrationProfile property: 
![portalngroups.png](/.attachments/portalngroups-1fd4008a-b203-45e3-8e91-8f1a0ef86f21.png)

## **How-to-guide:**

### **How to create a regional (zonal/non-zonal) NGroups?**
First create a CG profile. A sample CG profile is shown below. Currently supported API version is 2024-09-01-preview

```
{ 

    "properties": { 

        "sku": "Standard", 

        "containers": [ 

            { 

                "name": "container1", 

                "properties": { 

                    "image": "nginx", 

                    "ports": [ 

                    { 

                        "protocol": "TCP", 

                        "port": 80 

                    }], 

                    "resources": { 

                        "requests": { 

                            "memoryInGB": 2.0, 

                            "cpu": 1.0 

                        } 

                    } 

                } 

            } 

        ], 

        "restartPolicy": "Always", 

        "shutdownGracePeriod": "PT1H", 

        "ipAddress": { 

            "ports": [ 

            { 

                "protocol": "TCP", 

                "port": 80 

            }], 

            "type": "Public", 

            "autoGeneratedDomainNameLabelScope": "Unsecure" 

        }, 

        "timeToLive": "PT1H", 

        "osType": "Linux" 

    }
```
 Next, you can create a Zonal/Non-Zonal NGroup by either adding zones outside the properties or can leave the zones array empty.

```
{ 

    "properties": { 

        "elasticProfile": { 

            "desiredCount": 5 

        }, 

        "containerGroupProfiles": [ 

            { 

                "resource": { 

                    "id": "[resourceId('Microsoft.ContainerInstance/containerGroupProfiles', parameters('cgProfileName'))]" 

                } 

            } 

        ] 

    }, 

    "zones": [ "1", "2", "3" ] 

}
```
 When the NGroups is scaled out by setting its desiredCount property, the CGs are distributed evenly across all the specified zones. If one zone goes down, the application will still be available since the remaining CGs of the NGroups are running in other zones.  

**Technical capabilities and constraints:** 

1. Once a NGroup is created with a set of zones (e.g., { “1”, “2” }), the zones cannot be removed. However, a new zone can be added to the list. E.g., { “1”, “2”, “3” } 

2. If a specified zone is down, then the overall NGroups operation to create the CGs fails. Retry the request once the zone is back up. Another option is to simply delete the failed CGs. 

3. During scale down, NGroups randomly deletes instances which might not always maintain AZ spread. However, subsequent scale-out operations always try to rebalance the AZ spread.  

4. AZ spread isn’t supported with Spot Container. If you have such a requirement, please reach out to the ACI team. NGroups can enable this scenario if ACI allows it. There is currently no technical limitation to support this, just COGs/business reasons in ACI. 

5. Please also see: impact of availability due to infrastructure/platform updates

### **Create NGroup CGs with a prefix:**

Customers can create NGroup CGs with a prefix instead of just GUID names:

```
"properties": { 

    "elasticProfile": { 

        "desiredCount": 2,             

        "containerGroupNamingPolicy": { 

            "guidNamingPolicy": { 

                "prefix": "cg-" 

            } 

        } 

    },
```
 This is useful when you have multiple NGroups in a single RG and want to differentiate CGs belonging to each NGroup (e.g., in the Azure portal view). You can also change this for each scale out operation to identify CGs that were scaled out together in one operation.

### **Create NGroup with both SystemAssigned and UserAssigned managed identity:**

```
“location”: “{{location}}” 

"identity": { 

    "type": "SystemAssigned, UserAssigned", 

    "userAssignedIdentities": { 

        "/subscriptions/{{subId}}/resourceGroups/{{rgName}}/providers/Microsoft.ManagedIdentity/userAssignedIdentities/{{userAssignedIdentity1}}": {},  

"/subscriptions/{{subId}}/resourceGroups/{{rgName}}/providers/Microsoft.ManagedIdentity/userAssignedIdentities/{{userAssignedIdentity2}}": {} 

    }
```
### **If I delete some CGs from the NGroup, can the NGroup rebuild itself with new CGs to maintain its desired count?**

Yes, you can set the properties.elasticProfile.maintainDesiredCount bool property to true.  

This creates a new CG for every CG that is being deleted/detached from the NGroup. It will try and maintain the desiredCount property of the NGroup to its set value. 

This is useful when you want to use the NGroup as a ‘pool’ which automatically gets replenished when you take away CGs from the pool for your workload scenarios.  

This is a nullable bool property. If you omit it for subsequent NGroup PUT/update calls, it will not be reset to false. To reset, you must explicitly set it to false. When this is null/false, and when a CG is deleted/detached from the NGroup, the NGroup’s desiredCount property reduces accordingly. 
 
### **How do I get the CG name, NGroup Id and other metadata propagated down into the container?**

Currently, we expose only the CG name and orchestrator Id (the ARM resource Id). In future, additional properties could be considered. These 2 properties show up as container environment variables. 

To get these environment variables on the container, specify these tags _at the NGroup level_ 

```
“tags: { 

    “metadata.container.environmentVariable.containerGroupName”: true, 

    “metadata.container.environmentVariable.orchestratorId”: true, 

    : 

    : // other NGroup tags you may have 

    : 

}
```
NGroup understands these tags as ‘special’ and propagates the required environment variables down to each container as shown below.
![tagngroup.png](/.attachments/tagngroup-31a8d9f9-796b-4b4e-9f03-0f2346a502aa.png)
 