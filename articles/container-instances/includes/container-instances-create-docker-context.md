---
services: container-instances
author: tomvcassidy
ms.service: azure-container-instances
ms.topic: include
ms.date: 11/17/2025
ms.author: tomcassidy
# Customer intent: As a developer utilizing containerization, I want to create an Azure context with Docker, so that I can manage and deploy container instances seamlessly within my Azure subscription and resource group.
---

## Create Azure context

To use Docker commands to run containers in Azure Container Instances, first log into Azure:

```bash
docker login azure --tenant-id "[tenant ID]"
```
To find your tenant ID, browse to the Microsoft Entra ID properties.

When prompted, enter or select your Azure credentials.

Create an ACI context by running `docker context create aci`. This context associates Docker with an Azure subscription and resource group so you can create and manage container instances. For example, to create a context called *myacicontext*:

```
docker context create aci myacicontext
```

When prompted, select your Azure subscription ID, then select an existing resource group or **create a new resource group**. If you choose a new resource group, it has a system-generated name on creation. Azure container instances, like all Azure resources, must be deployed into a resource group. Resource groups allow you to organize and manage related Azure resources.

Run `docker context ls` to confirm that you added the ACI context to your Docker contexts:

```
docker context ls
```
