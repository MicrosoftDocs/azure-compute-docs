---
author: cynthnan
ms.service: azure-virtual-machines
ms.topic: include
ms.date: 10/26/2018
ms.author: cynthn
# Customer intent: "As a cloud administrator, I want to migrate my virtual machines during scheduled maintenance, so that I can ensure minimal disruption and properly manage any changes, including IP address updates and agent requirements."
---

* The migration will restart the VM, so schedule the migration of your VMs during a pre-existing maintenance window. 

* The migration isn't reversible. 

* Be sure to test the migration. Migrate a test virtual machine before you perform the migration in production.

* During the migration, you deallocate the VM. The VM receives a new IP address when it's started after the migration. If needed, you can [assign a static IP address](/azure/virtual-network/ip-services/public-ip-addresses) to the VM.

* Review the minimum version of the Azure VM agent required to support the migration process. For information on how to check and update your agent version, see [Minimum version support for VM agents in Azure](https://support.microsoft.com/help/4049215/extensions-and-virtual-machine-agent-minimum-version-support)
