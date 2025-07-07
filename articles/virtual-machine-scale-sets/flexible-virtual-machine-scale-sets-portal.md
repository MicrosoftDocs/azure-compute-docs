---
title: Create virtual machines in a Flexible scale set using Azure portal
description: Learn how to create a Virtual Machine Scale Set in Flexible orchestration mode in the Azure portal.
author: fitzgeraldsteele
ms.author: fisteele
ms.topic: how-to
ms.service: azure-virtual-machine-scale-sets
ms.date: 04/24/2025
ms.reviewer: jushiman
ms.custom: mimckitt, vmss-flex
# Customer intent: "As a cloud administrator, I want to create a Virtual Machine Scale Set using the portal, so that I can efficiently manage and scale virtual machine instances according to my application's needs."
---

# Create virtual machines in a scale set using Azure portal

This article steps through using Azure portal to create a Virtual Machine Scale Set.
## Sign in to Azure
Sign in to the [Azure portal](https://portal.azure.com).


## Create a Virtual Machine Scale Set

You can deploy a scale set with a Windows Server image or Linux image such as RHEL, Ubuntu, or SLES.

1. In the Azure portal search bar, search for and select **Virtual Machine Scale Sets**.
   
2. Select **Create** on the **Virtual Machine Scale Sets** page.
   
3. In the **Basics** tab, under **Project details**, make sure the correct subscription is selected and create a new resource group called *myVMSSResourceGroup*.
   
4. Under **Scale set details**, set *myScaleSet* for your scale set name and select a **Region** that is close to your area.
   
5. Under **Orchestration**, select *Flexible* Orchestration Mode.
    
6. Under **Security type**, select *Trusted launch virtual machines*.
    
7. Under **Scaling**, keep it the Scaling mode as *Manual*.
    
8. Under **Instance count**, set the **initial instance count** field to *5*. You can set this number up to 1000
    
9. Under **Instance details**, select a marketplace image for **Image**. Select any of the Supported Distros.
    
10. Under **Administrator account** configure the admin username and set up an associated password or SSH public key.
      - A **Password** must be at least 12 characters long and meet three out of the four following complexity requirements: one lower case character, one upper case character, one number, and one special character. For more information, see [username and password requirements](../virtual-machines/windows/faq.yml#what-are-the-password-requirements-when-creating-a-vm-).
      - If you select a Linux OS disk image, you can instead choose **SSH public key**. You can use an existing key or create a new one. In this example, we will have Azure generate a new key pair for us. For more information on generating key pairs, see [create and use SSH keys](../virtual-machines/linux/mac-create-ssh-keys.md).


:::image type="content" source="media/quickstart-guides/quick-start-portal-1.png" alt-text="A screenshot of the Basics tab in the Azure portal during the Virtual Machine Scale Set creation process.":::

11. Select **Next: Spot** to move to the Spot virtual machine configuration options. For this quickstart, leave the default Spot configurations.
    
12. Select **Next: Disks** to move the disk configuration options. For this quickstart, leave the default disk configurations.
    
13. Select **Next: Networking** to move the networking configuration options.
    
14. On the **Networking** page, under **Load balancing**, select **Azure load balancer**.
    
15. In **Select a load balancer**, select a load balancer or create a new one.

:::image type="content" source="media/quickstart-guides/quick-start-portal-2.png" alt-text="A screenshot of the Networking tab in the Azure portal during the Virtual Machine Scale Set creation process.":::

16. When you're done, select **Review + create**.
    
17. After it passes validation, select **Create** to deploy the scale set.


## Clean up resources
When no longer needed, delete the resource group, scale set, and all related resources. To do so, select the resource group for the scale set and then select **Delete**.


## Next steps
> [!div class="nextstepaction"]
> [Learn how to create a Flexible scale with Azure CLI.](flexible-virtual-machine-scale-sets-cli.md)
