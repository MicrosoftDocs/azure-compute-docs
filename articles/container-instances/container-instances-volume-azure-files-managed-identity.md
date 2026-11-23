---
title: Mount an Azure file share using managed identity in Azure Container Instances
description: Learn how to mount an Azure Files SMB share in Azure Container Instances without using a storage account access key.
ms.topic: how-to
ms.service: azure-container-instances
ms.date: 11/23/2026
ai-usage: ai-assisted
---

# Mount an Azure file share using managed identity and Kerberos

Azure Container Instances can mount an Azure Files SMB share by using a user-assigned managed identity. This method is a security enhancement to using a storage account access key. Internally, ACI uses the managed identity's OAuth token to obtain a Kerberos ticket for the storage account, enabling read/write access to Azure Files based on the identity's RBAC permissions.
Once mounted, the file share supports the same features as a share mounted using key-based authentication. For more information about key-based authentication and supported volume mount features, see [Mount an Azure file share in Azure Container Instances](container-instances-volume-azure-files.md).

## Limitations

* Only Linux containers are supported.
* System-assigned managed identities are not supported. Use a user-assigned managed identity.
* The storage account must use the SMB protocol for this feature. NFS isn't supported.
* Azure file share mounts using managed identity in a confidential container group aren't yet directly supported with ACI. However, you can deploy confidential pods through AKS using confidential VN2.

## API version

Use API version `2026-07-01` to mount Azure Files shares using managed identity.

> [!NOTE]
> Confirm the API version and feature availability for the target Azure region before publishing or deploying the workload.

## Prerequisites

Before you deploy the container group, complete the following steps.

### 1. Create the storage account and file share

Before mounting a file share using a user-assigned managed identity, create the storage account and file share with the required settings. Run the following commands in Bash. Use an existing resource group and replace `samplestorageaccount` with a globally unique storage account name. Keep these variables available for the later steps.

```azurecli-interactive
RESOURCE_GROUP=myResourceGroup
LOCATION=eastus
STORAGE_ACCOUNT=samplestorageaccount

# Create storage account
az storage account create \
  --name "$STORAGE_ACCOUNT" \
  --resource-group "$RESOURCE_GROUP" \
  --location "$LOCATION" \
  --sku Standard_LRS \
  --enable-smb-oauth true \
  --allow-blob-public-access false \
  --allow-shared-key-access false

# Create Azure Files share
az storage share-rm create \
  --resource-group "$RESOURCE_GROUP" \
  --storage-account "$STORAGE_ACCOUNT" \
  --name acishare \
  --quota 100 \
  --enabled-protocols SMB
```

Once these resources are created, confirm the settings on the storage account's **Configuration** page. Set **Managed Identity for SMB** to **Enabled**, **Allow Blob anonymous access** to **Disabled**, and **Allow storage account key access** to **Disabled**, as shown in the following screenshot.

:::image type="content" source="./media/container-instances-volume-azure-files-managed-identity/storage-account-configuration.png" alt-text="Screenshot of the storage account Configuration page with Managed Identity for SMB enabled.":::

### 2. Create the managed identity and assign permissions

Assign the `Storage File Data SMB MI Admin` RBAC role to the user-assigned managed identity that you intend to use to mount the Azure file share. On the storage account's **Access control (IAM)** page, select **Add role assignment** and assign this role to the identity.

:::image type="content" source="./media/container-instances-volume-azure-files-managed-identity/storage-account-access-control.png" alt-text="Screenshot of the storage account Access control (IAM) page with the Add role assignment button.":::

Alternatively, use the following commands to create a user-assigned managed identity and assign the RBAC role to it. Use the resource group, location, and storage account variables from the previous step. If you already have a user-assigned managed identity in this resource group, set `IDENTITY_NAME` to its name, skip `az identity create`, and run the remaining commands to retrieve its IDs and assign the role.

```azurecli-interactive
IDENTITY_NAME=aci-fileshare-identity

az identity create \
  --resource-group "$RESOURCE_GROUP" \
  --name "$IDENTITY_NAME" \
  --location "$LOCATION"

IDENTITY_CLIENT_ID=$(az identity show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$IDENTITY_NAME" \
  --query clientId \
  --output tsv)

IDENTITY_RESOURCE_ID=$(az identity show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$IDENTITY_NAME" \
  --query id \
  --output tsv)

STORAGE_SCOPE=$(az storage account show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$STORAGE_ACCOUNT" \
  --query id \
  --output tsv)

az role assignment create \
  --assignee "$IDENTITY_CLIENT_ID" \
  --role "Storage File Data SMB MI Admin" \
  --scope "$STORAGE_SCOPE"
```

> [!NOTE]
> Role assignments can take several minutes to propagate.

## Deploy a container group

> [!NOTE]
> Don't specify `storageAccountKey` when using managed identity authentication. `userManagedIdentityClientId` and `storageAccountKey` represent mutually exclusive authentication methods. Specify only the property for the authentication method that you use.

### Sample payload without a virtual network

The following Resource Manager payload mounts the `acishare` share at `/mnt/fileshare`. It uses the user-assigned managed identity instead of a storage account key. The client ID of the user-assigned managed identity is used in the volume definition. The identity resource ID is used in the container group's `identity` property.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "identityResourceId": {
      "type": "string"
    },
    "identityClientId": {
      "type": "string"
    },
    "storageAccountName": {
      "type": "string"
    },
    "fileShareName": {
      "type": "string",
      "defaultValue": "acishare"
    }
  },
  "resources": [
    {
      "name": "fileshare-demo",
      "type": "Microsoft.ContainerInstance/containerGroups",
      "apiVersion": "2026-07-01",
      "location": "[resourceGroup().location]",
      "identity": {
        "type": "UserAssigned",
        "userAssignedIdentities": {
          "[parameters('identityResourceId')]": {}
        }
      },
      "properties": {
        "osType": "Linux",
        "restartPolicy": "Never",
        "containers": [
          {
            "name": "fileshare-client",
            "properties": {
              "image": "mcr.microsoft.com/azure-cli:latest",
              "resources": {
                "requests": {
                  "cpu": 1,
                  "memoryInGB": 1.5
                }
              },
              "command": [
                "/bin/sh",
                "-c",
                "ls -la /mnt/fileshare && sleep 3600"
              ],
              "volumeMounts": [
                {
                  "name": "fileshare-volume",
                  "mountPath": "/mnt/fileshare",
                  "readOnly": false
                }
              ]
            }
          }
        ],
        "volumes": [
          {
            "name": "fileshare-volume",
            "azureFile": {
              "shareName": "[parameters('fileShareName')]",
              "storageAccountName": "[parameters('storageAccountName')]",
              "userManagedIdentityClientId": "[parameters('identityClientId')]",
              "readOnly": false
            }
          }
        ]
      }
    }
  ]
}
```

Save the payload as `deploy-aci-fileshare-mi.json`, then deploy it with:

```azurecli-interactive
az deployment group create \
  --resource-group "$RESOURCE_GROUP" \
  --template-file deploy-aci-fileshare-mi.json \
  --parameters \
    identityResourceId="$IDENTITY_RESOURCE_ID" \
    identityClientId="$IDENTITY_CLIENT_ID" \
    storageAccountName="$STORAGE_ACCOUNT"
```

### Sample virtual network payload

This example deploys a container group into an existing virtual network and mounts the same Azure file share by using a user-assigned managed identity.

Before you deploy:

* Use a subnet delegated to `Microsoft.ContainerInstance/containerGroups` that contains only container groups. For setup instructions, see [Deploy to an existing virtual network](container-instances-vnet.md#deploy-to-existing-virtual-network).
* Use a subnet in the same Azure region as the container group. This template deploys the container group in the resource group's region.
* Configure a [NAT gateway for outbound connectivity](container-instances-nat-gateway.md) on the subnet.
* If the storage account restricts network access, configure an allowed network path from the container group to the Azure Files endpoint. For networking options, see [Azure Files networking considerations](/azure/storage/files/storage-files-networking-overview).
* Ensure that the network path used to mount the Azure file share permits SMB traffic over TCP port 445.
* If accessing Azure Files through a private endpoint, ensure that DNS resolution directs the storage account's file endpoint to the private endpoint IP address.

> [!IMPORTANT]
> The container group and the storage account must have network connectivity. Deploying into a virtual network doesn't automatically grant access through the storage account firewall.

The `subnetResourceId` parameter identifies the delegated subnet. The `subnetIds` property places the container group in that subnet. This example doesn't expose inbound ports.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "identityResourceId": {
      "type": "string"
    },
    "identityClientId": {
      "type": "string"
    },
    "storageAccountName": {
      "type": "string"
    },
    "fileShareName": {
      "type": "string",
      "defaultValue": "acishare"
    },
    "subnetResourceId": {
      "type": "string"
    }
  },
  "resources": [
    {
      "name": "fileshare-vnet-demo",
      "type": "Microsoft.ContainerInstance/containerGroups",
      "apiVersion": "2026-07-01",
      "location": "[resourceGroup().location]",
      "identity": {
        "type": "UserAssigned",
        "userAssignedIdentities": {
          "[parameters('identityResourceId')]": {}
        }
      },
      "properties": {
        "osType": "Linux",
        "restartPolicy": "Never",
        "subnetIds": [
          {
            "id": "[parameters('subnetResourceId')]"
          }
        ],
        "containers": [
          {
            "name": "fileshare-client",
            "properties": {
              "image": "mcr.microsoft.com/azure-cli:latest",
              "resources": {
                "requests": {
                  "cpu": 1,
                  "memoryInGB": 1.5
                }
              },
              "command": [
                "/bin/sh",
                "-c",
                "ls -la /mnt/fileshare && sleep 3600"
              ],
              "volumeMounts": [
                {
                  "name": "fileshare-volume",
                  "mountPath": "/mnt/fileshare",
                  "readOnly": false
                }
              ]
            }
          }
        ],
        "volumes": [
          {
            "name": "fileshare-volume",
            "azureFile": {
              "shareName": "[parameters('fileShareName')]",
              "storageAccountName": "[parameters('storageAccountName')]",
              "userManagedIdentityClientId": "[parameters('identityClientId')]",
              "readOnly": false
            }
          }
        ]
      }
    }
  ]
}
```

Save the payload as `deploy-aci-fileshare-mi-vnet.json`. Use the identity and storage variables from the earlier steps. Replace the network resource group, virtual network name, and subnet name with your existing network's values, then deploy:

```azurecli-interactive
VNET_RESOURCE_GROUP=myResourceGroup
VNET_NAME=myVNet
SUBNET_NAME=myAciSubnet

SUBNET_RESOURCE_ID=$(az network vnet subnet show \
  --resource-group "$VNET_RESOURCE_GROUP" \
  --vnet-name "$VNET_NAME" \
  --name "$SUBNET_NAME" \
  --query id \
  --output tsv)

az deployment group create \
  --resource-group "$RESOURCE_GROUP" \
  --template-file deploy-aci-fileshare-mi-vnet.json \
  --parameters \
    identityResourceId="$IDENTITY_RESOURCE_ID" \
    identityClientId="$IDENTITY_CLIENT_ID" \
    storageAccountName="$STORAGE_ACCOUNT" \
    subnetResourceId="$SUBNET_RESOURCE_ID"
```

After deployment, check the container logs for the directory listing from the mounted share:

```azurecli-interactive
az container logs \
  --resource-group "$RESOURCE_GROUP" \
  --name fileshare-vnet-demo
```

## Next steps

Learn how to mount other volume types in Azure Container Instances:

* [Mount an emptyDir volume in Azure Container Instances](container-instances-volume-emptydir.md).
* [Mount a gitRepo volume in Azure Container Instances](container-instances-volume-gitrepo.md).
* [Mount a secret volume in Azure Container Instances](container-instances-volume-secret.md).