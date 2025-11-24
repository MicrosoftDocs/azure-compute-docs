---
title: Deploy OpenShift to Azure Stack Hub
ms.reviewer: robess
description: Discover how to deploy OpenShift on Azure Stack Hub, configure CNS for GlusterFS, and leverage Red Hat-managed RHCOS images.
author: ronmiab
ms.author: robess
ms.service: azure-stack
ms.custom: linux-related-content
ms.collection: linux
ms.topic: concept-article
ms.date: 11/24/2025
# Customer intent: As a system administrator, I want to deploy OpenShift on Azure Stack Hub, so that I can leverage container orchestration in a hybrid cloud environment with the specific storage configurations needed for my organization.
---

# OpenShift Container Platform or OKD deployment to Azure Stack Hub

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Flexible scale sets 

[OpenShift](openshift-get-started.md) can be deployed in Azure Stack Hub. Some key differences exist between Azure and Azure Stack Hub, so deployment and capabilities differ slightly.

Currently, the Azure Cloud Provider doesn't work in Azure Stack Hub. You can't use disk attach for persistent storage in Azure Stack Hub. Instead, you can configure other storage options such as NFS, iSCSI, and GlusterFS. Or, you can enable CNS and use GlusterFS for persistent storage. If you enable CNS, the deployment adds three more nodes with storage for GlusterFS usage.

## Deploy OpenShift 4.x On Azure Stack Hub

Red Hat manages the Red Hat Enterprise Linux CoreOS (RHCOS) image for OpenShift 4.x. The deployment process gets the image from a Red Hat endpoint. As a result, you don't need to get an image from the Azure Stack hub Marketplace.

Follow the steps in the OpenShift documentation at [Installing a cluster on Azure Stack Hub using ARM templates](https://docs.openshift.com/container-platform/4.9/installing/installing_azure_stack_hub/installing-azure-stack-hub-user-infra.html).

> [!WARNING]
> If you have an issue with OpenShift, contact Red Hat for support.
## Next steps

[Installing a cluster on Azure Stack Hub using ARM templates](https://docs.openshift.com/container-platform/4.9/installing/installing_azure_stack_hub/installing-azure-stack-hub-user-infra.html).
