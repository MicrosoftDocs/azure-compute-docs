---
title: VM Snapshot extension using VM restore points
description: VM Snapshot extension using VM restore points
author: aarthiv
ms.author: aarthiv
ms.service: azure-virtual-machines
ms.subservice: recovery
ms.topic: how-to
ms.date: 11/2/2023
ms.custom: template-how-to
# Customer intent: As a cloud administrator, I want to utilize VM snapshot extensions for application-consistent restore points, so that I can ensure the integrity of my applications during backup operations.
---

# VMSnapshot extension

---

Application-consistent restore points capture VM data in a state that guarantees application integrity at the time of the snapshot. To achieve this, Azure coordinates with applications running on the VM before taking the snapshot:

- **Windows**: The **VMSnapshot Windows** extension triggers the Volume Shadow Copy Service (VSS) to quiesce application writes.
- **Linux**: The **VMSnapshot Linux** extension runs pre- and post-scripts to flush and resume application I/O.

When you request an application-consistent restore point, Azure automatically installs the VMSnapshot extension on the VM if it isn't already present. The extension updates automatically — no manual management is required.

> [!IMPORTANT]
> Azure begins creating a restore point only after **all extensions** on the VM, including VMSnapshot, have successfully provisioned. Monitor extension provisioning state before expecting restore point creation to succeed.

To confirm the extension is installed and in a **Provisioned** state, check **VM > Extensions + applications** in the Azure portal, or run:

```azurecli
az vm extension show \
  --resource-group <resourceGroupName> \
  --vm-name <vmName> \
  --name VMSnapshot
```

## Extension logs

Use the following paths to access VMSnapshot extension logs directly on the VM:

| OS | Log path |
|---|---|
| Windows | `C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.RecoveryServices.VMSnapshot` |
| Linux | `/var/log/azure/Microsoft.Azure.RecoveryServices.VMSnapshotLinux/extension.log` |

## Troubleshooting

Most restore point failures are caused by communication issues between the VM agent and the VMSnapshot extension. Start troubleshooting with [Troubleshoot restore point failures](restore-point-troubleshooting.md).

### VSS writer failures (Windows)

When a VSS writer fails, Azure cannot take an application-consistent snapshot and falls back to a file system-consistent restore point. This fallback applies for the **next three attempts**, regardless of the configured restore point schedule. From the **fourth attempt onward**, Azure resumes attempting application-consistent restore points.

To resolve VSS writer failures, see [Troubleshoot VSS writer issues](https://learn.microsoft.com/en-us/azure/backup/backup-azure-vms-troubleshoot#extensionfailedvsswriterinbadstate---snapshot-operation-failed-because-vss-writers-were-in-a-bad-state).

> [!WARNING]
> Do not manually delete the VMSnapshot extension. Deleting it breaks subsequent application-consistent restore point creation until the extension is reinstalled and provisioned again.

## Next steps

[Create a VM restore point](create-restore-points.md)