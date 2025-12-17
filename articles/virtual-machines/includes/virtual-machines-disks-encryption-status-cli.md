---
 title: include file
 description: include file
 services: virtual-machines
 author: roygara
 ms.service: azure-virtual-machines
 ms.topic: include
 ms.date: 06/23/2020
 ms.author: rogarana
 ms.custom: include file
# Customer intent: "As a cloud engineer, I want to check the encryption type of a specific disk in my resource group, so that I can ensure compliance with security policies."
---
```azurecli
az disk show -g yourResourceGroupName -n yourDiskName --query [encryption.type] -o tsv
```