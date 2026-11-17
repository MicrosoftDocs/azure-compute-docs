---
title: Serverless sandboxes prerequisites
description: Prerequisites for serverless sandboxes in Azure Container Instances
ms.author: dpeshwani
author: drishtipeshwani09
ms.service: azure-container-instances
services: container-instances
ms.topic: include
ms.date: 11/04/2026
---

* An Azure subscription. If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/) before you begin.
* The [Azure CLI](/cli/azure/install-azure-cli), version 2.60 or later, signed in with `az login`, or another tool to send authenticated REST requests to Azure Resource Manager.
* The `Microsoft.ContainerInstance` resource provider registered in your subscription.

    ```azurecli
    az provider register --namespace Microsoft.ContainerInstance --wait
    ```

* The **Contributor**, **Azure Deployment Stack Owner**, and **Storage Blob Data Contributor** roles assigned to the **Compute Sandbox Groups** service principal on your subscription, so that the service can provision the managed resource group and snapshot storage account in your subscription. To assign the roles, you need permission to create role assignments on the subscription, such as the **Owner** or **User Access Administrator** role:

    ```azurecli
    subscriptionId=$(az account show --query id --output tsv)
    spObjectId=$(az ad sp show --id 1eeacc2b-ec4a-4b1e-977a-2b5fef2131c6 --query id --output tsv 2>/dev/null || \
      az ad sp create --id 1eeacc2b-ec4a-4b1e-977a-2b5fef2131c6 --query id --output tsv)

    for role in "Contributor" "Azure Deployment Stack Owner" "Storage Blob Data Contributor"; do
      az role assignment create --assignee-object-id $spObjectId --assignee-principal-type ServicePrincipal \
        --role "$role" --scope /subscriptions/$subscriptionId
    done
    ```

    Role assignments can take up to 15 minutes to take effect.
* Quota provisioned for serverless sandboxes in Azure Container Instances on your subscription:
    1. Open [New support request in the Azure portal](https://portal.azure.com/#view/Microsoft_Azure_Support/NewSupportRequestV4Blade/topicId/a11041c8-bf9f-820b-f1ef-c859995c6040/productId/06bfd9d3-516b-d5c6-5802-169c800dec89/issueType/quota) (with **Issue type** set to **Service and subscription limits (quotas)** and **Quota type** set to **Container Instances**), select your subscription, and go to the **Additional details** tab.
    1. Select **Enter details** to open the **Quota details** pane, choose your **Location**, select **AIAgentsGroups - StandardCores (Preview)** for **Quota type**, and fill in **Networking type**, **Container Type**, **Total AIAgentsGroups Group Limit**, and **Total AIAgentsGroups Core Limit**:

        :::image type="content" source="../media/ai-sandboxes/ai-sandboxes-quota-request.png" alt-text="Screenshot of the New support request page in the Azure portal showing the Quota details pane with AIAgentsGroups - StandardCores (Preview) selected." lightbox="../media/ai-sandboxes/ai-sandboxes-quota-request.png":::

    1. Submit the support request. After your quota request is approved, re-register the `Microsoft.ContainerInstance` resource provider (`az provider register --namespace Microsoft.ContainerInstance --wait`) so that your subscription picks up the newly enabled capabilities.
* To connect to a running sandbox after you create an AI agents group and a sandbox (see [Quickstart: Create your first serverless sandbox](../container-instances-ai-sandboxes-quickstart.md)), a gRPC client generated from the sandbox gateway protobuf file, [`sandboxgateway.proto`](../container-instances-ai-sandboxes-gateway-proto.md). A gRPC client is required because the sandbox gateway exposes a gRPC service (`aci.sandboxgateway.SandboxGatewayService`) for in-sandbox operations such as waiting for readiness, running commands, transferring files, and reading logs. To use the cross-platform sample C# CLI in this documentation (supported on Linux, macOS, and Windows), install the [.NET 8 SDK](https://dotnet.microsoft.com/download/dotnet/8.0) and [build the CLI](../container-instances-ai-sandboxes-gateway-proto.md#build-a-c-cli-from-sandboxgatewayproto).

    > [!NOTE]
    > A gRPC client isn't required if you schedule and manage serverless sandboxes through Azure Kubernetes Service (AKS).
