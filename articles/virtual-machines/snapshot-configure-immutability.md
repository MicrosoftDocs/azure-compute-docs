# Configure immutability for managed disk snapshots (preview)

Azure disk snapshots are inherently write-once, read-many (WORM). Once created, a snapshot's data can't be modified or overwritten. An immutability policy prevents deletion of the snapshot for a specified retention period. You can use immutability policies to meet regulatory compliance requirements (SEC 17a-4, CFTC 1.31, FINRA 4511) and for additional protection against ransomware.

Immutability is enforced through an `immutabilityPolicy` property on individual snapshots. When an immutability policy is active, the snapshot can't be deleted by any user, regardless of their role or permissions. Users with the required permissions can extend, reduce, or remove an unlocked policy. An active locked policy can only be extended.

## Limitations

The following limitations apply only during public preview:

- An `immutabilityPolicy` can only be set using PowerShell or the REST API. Azure portal and Azure CLI support is planned for general availability (GA).
- Only apply an `immutabilityPolicy` to a snapshot with `snapshotAccessState` set to `Available` or `AvailableWithInstantAccess`. During public preview, a request to apply an immutability policy to a snapshot in any other state can incorrectly report success without saving the policy. Requests made in unsupported states are planned to return an error at GA.
- Only use the dedicated `immutabilityPolicy` operations to set and update a policy. Snapshot creation or update operations can accept the `immutabilityPolicy` property updates even though these operations aren't supported for policy management. Immutability policy requests made during snapshot create or update operations will begin reporting errors at GA.

The following limitations will continue to apply during GA:

- Immutability policies can only be set after snapshot creation, not at creation time.
- An immutability policy can only be set on a snapshot in the following `SnapshotAccessState`:
  - `Available`: Premium SSD, Standard SSD, and Standard HDD snapshots; incremental snapshots of Ultra Disk and Premium SSD v2 disks after the background data copy completes; snapshots copied within the same region using shallow copy.
  - `AvailableWithInstantAccess`: Instant access snapshots of Ultra Disk and Premium SSD v2 disks when the instant access duration hasn't lapsed and the background data copy completes.
- An immutability policy can't be set on a snapshot in the following `SnapshotAccessState`:
  - `Pending`: Incremental snapshots of Ultra Disk and Premium SSD v2 disks during background data copy; snapshots being copied across regions.
  - `InstantAccess`: Instant access snapshots of Ultra Disk and Premium SSD v2 disks when the instant access duration hasn't lapsed and the background data copy is ongoing.
- The maximum retention period is 90 days.
- For long-term immutable storage, use [Azure Backup Immutable Vaults](/azure/backup/backup-azure-immutable-vault-concept), which provide cost-effective, long-term immutable retention.

## Immutability policy types

You can configure two types of immutability policies depending on your needs. The immutability policy must be in a locked state to be fully compliant in accordance with regulatory requirements. While the unlocked state provides immutability protection, we don't recommend using the unlocked state for anything other than short-term testing.

| Type | Allowed modifications | Use case |
| --- | --- | --- |
| Unlocked | Extend, reduce, or remove policy. | Data governance, testing, ransomware protection. |
| Locked | Extend retention only; can't be removed or changed to an unlocked policy. | Regulatory compliance for financial, legal, or medical records. |

> [!WARNING]
> An active locked policy can't be removed, shortened, or changed to unlocked by anyone, including Azure. You must wait for retention to expire to delete the snapshot. Verify the policy type and duration before applying or extending locked retention.

## Immutability policy properties

When you set an immutability policy, you specify a retention duration in days, along with the type of policy you intend to set. The system records the start time and calculates the expiry automatically.

| Property | Description |
| --- | --- |
| `immutabilityDurationDays` | Total retention period up to 90 days from when the policy was first set. User assigned. |
| `type` | `Unlocked` or `Locked`. User assigned. |
| `policyStartTime` | Timestamp assigned by the service when the policy is first set. Doesn't support user-assigned values. |
| `policyExpirationTime` | Equals `policyStartTime` + `immutabilityDurationDays`. Doesn't support user-assigned values. |
| `isPolicyExpired` | `true` when the current time exceeds `policyExpirationTime`. Doesn't support user-assigned values. |

Key behaviors:

- Setting `immutabilityDurationDays` to 0 on an unlocked policy expires it immediately. After expiry, the snapshot returns to its normal state. You can delete it or apply a new policy, including on snapshots that previously had an immutability policy.
- Updating an active policy, including changing it from Unlocked to Locked, retains its original `policyStartTime`. The duration you specify is the total retention in days from that original start, not additional days from the update, and `policyExpirationTime` is recalculated accordingly. A new start is assigned only when you are applying an immutability policy on a snapshot for the first time or replacing an expired policy.

## Allowed and blocked operations

While an immutability policy is active:

| Operation | Result |
| --- | --- |
| Create a disk from the snapshot | Allowed. Standard restore workflow is unaffected. |
| Copy the snapshot | Allowed. The copy doesn't inherit the immutability policy. |
| Read snapshot properties | Allowed. |
| Delete the snapshot | Blocked. |

> [!IMPORTANT]
> Copying a snapshot doesn't change the source snapshot's immutability policy. The copy doesn't inherit that policy. Apply a separate policy to the copy if it needs protection.

## Access control

Use the dedicated built-in roles when available, or assign the equivalent permissions to conduct the action in a custom role. Privileged roles such as Contributor and Owner have these permissions built in by default.

| Built-in RBAC role | Corresponding action | Permissions |
| --- | --- | --- |
| Disk Snapshot Immutability Operator | `Microsoft.Compute/snapshots/updateImmutabilityPolicy/action` | Create, extend, shorten, or remove an unlocked policy. |
| Disk Snapshot Immutability Admin | `Microsoft.Compute/snapshots/updateImmutabilityPolicyLock/action` | Create and extend Locked retention. Perform the documented explicit removal of an active Unlocked policy. |

## Regulatory compliance

Microsoft retained a leading independent assessment firm that specializes in records management and information governance, Cohasset Associates, to evaluate immutable disk snapshots and their compliance with requirements specific to the financial services industry. Cohasset validated that an active locked immutable policy on a snapshot meets the relevant storage requirements of CFTC Rule 1.31(c)-(d), FINRA Rule 4511, and SEC Rule 17a-4(f). Microsoft targeted this set of rules because they represent the most prescriptive guidance globally for records retention for financial institutions.

See the full [Cohasset report](https://aka.ms/AzureImmutableSnapshotCertification).

### How immutability policies satisfy regulatory requirements

| Regulatory requirement | How Azure satisfies it |
| --- | --- |
| Non-rewriteable storage | Managed disk snapshots are inherently read-only and can't be modified after creation. |
| Non-erasable during retention | An active immutability policy on a snapshot prevents deletion by all users, including administrators. |
| Retention enforcement | Locked policy retention can't be reduced, nor can a locked policy be removed by any user, including Azure. |
| Record verification | Storage-layer integrity checks and redundancy. |
| Retrieval and portability | Restore to disk, copy cross-region or subscription, or export via SAS URI. |
| Audit trail | Azure Activity Log tracks all policy and access operations and retains logs by default for 90 days. |

## Billing

There's no extra charge for immutability policies. Azure bills snapshots at standard managed disk snapshot rates. Active locked protection prevents deletion until retention expires. Expiry or policy removal doesn't delete the snapshot; normal snapshot charges continue until deletion.

For details, see [Managed Disks pricing](https://azure.microsoft.com/pricing/details/managed-disks/).

## Configure immutability policies for disk snapshots

Use Azure PowerShell or the REST API to set and update immutability policies on existing snapshots.

### Set an immutability policy on a snapshot

You can set an immutability policy only on an existing snapshot, not at snapshot creation time. Choose a policy type and specify the retention duration in days (up to 90). An unlocked policy prevents deletion while allowing you to modify or remove the policy later; use it for data governance or to test your configuration. A locked policy provides compliance-grade protection: while active, it can't be removed, reduced, or changed to unlocked.

> [!WARNING]
> A locked policy can't be removed, shortened, or changed to unlocked while it is active. The snapshot can't be deleted until retention expires, and charges continue until you delete the snapshot. Verify the policy type and retention duration before applying.

# [Azure PowerShell](#tab/azure-powershell)

```powershell
# Unlocked policy
$snapshot = Set-AzSnapshotImmutabilityPolicy `
    -ResourceGroupName "myResourceGroup" `
    -Name "mySnapshot" `
    -PolicyType Unlocked `
    -DurationInDays 30
$snapshot.ImmutabilityPolicy | Format-List

# Locked policy
$snapshot = Set-AzSnapshotImmutabilityPolicy `
    -ResourceGroupName "myResourceGroup" `
    -Name "mySnapshot" `
    -PolicyType Locked `
    -DurationInDays 30
$snapshot.ImmutabilityPolicy | Format-List
```

# [REST API](#tab/rest)

```http
# Unlocked policy
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/snapshots/{snapshotName}/updateImmutabilityPolicy?api-version=2026-03-02
Content-Type: application/json

{
  "type": "Unlocked",
  "immutabilityDurationDays": 30
}

# Locked policy
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/snapshots/{snapshotName}/updateImmutabilityPolicyLock?api-version=2026-03-02
Content-Type: application/json

{
  "type": "Locked",
  "immutabilityDurationDays": 30
}
```

---

### Update an existing immutability policy on a snapshot

Update an active policy by running the same operation with new values. An update keeps the original `policyStartTime`, and the duration you specify is the total retention in days from that start, not additional days. `policyExpirationTime` is recalculated accordingly. You can:

- Extend an unlocked or locked policy. The new duration must be greater than the current value and no more than 90 days.
- Reduce or remove an unlocked policy only. The new value must be less than the current duration. Choosing 0 or a duration that has already elapsed expires protection immediately.
- Change an unlocked policy to locked by specifying the Locked type and a total duration.

> [!NOTE]
> Locking an active unlocked policy doesn't restart retention. For example, a policy that starts on November 1 and is locked on November 4 with a 30-day duration expires on December 1, not December 4. Check the returned `policyExpirationTime` to confirm the remaining protection.

# [Azure PowerShell](#tab/azure-powershell)

```powershell
# Extend or reduce an unlocked policy (for example, 60 or 14 days)
$snapshot = Set-AzSnapshotImmutabilityPolicy `
    -ResourceGroupName "myResourceGroup" `
    -Name "mySnapshot" `
    -PolicyType Unlocked `
    -DurationInDays 60
$snapshot.ImmutabilityPolicy | Format-List

# Extend a locked policy
$snapshot = Set-AzSnapshotImmutabilityPolicy `
    -ResourceGroupName "myResourceGroup" `
    -Name "mySnapshot" `
    -PolicyType Locked `
    -DurationInDays 60
$snapshot.ImmutabilityPolicy | Format-List

# Change an unlocked policy to locked
$snapshot = Set-AzSnapshotImmutabilityPolicy `
    -ResourceGroupName "myResourceGroup" `
    -Name "mySnapshot" `
    -PolicyType Locked `
    -DurationInDays 30
$snapshot.ImmutabilityPolicy | Format-List

# Remove an unlocked policy
$snapshot = Remove-AzSnapshotImmutabilityPolicy `
    -ResourceGroupName "myResourceGroup" `
    -Name "mySnapshot"
```

# [REST API](#tab/rest)

```http
# Extend or reduce an unlocked policy (for example, 60 or 14 days)
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/snapshots/{snapshotName}/updateImmutabilityPolicy?api-version=2026-03-02
Content-Type: application/json

{
  "type": "Unlocked",
  "immutabilityDurationDays": 60
}

# Extend a locked policy, or change an unlocked policy to locked
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/snapshots/{snapshotName}/updateImmutabilityPolicyLock?api-version=2026-03-02
Content-Type: application/json

{
  "type": "Locked",
  "immutabilityDurationDays": 60
}

# Remove an unlocked policy
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/snapshots/{snapshotName}/updateImmutabilityPolicyLock?api-version=2026-03-02
Content-Type: application/json

{
  "type": "Unlocked",
  "immutabilityDurationDays": 0
}
```

---

## Next steps

- [Create an incremental snapshot for managed disks](disks-incremental-snapshots.md).
- [Create a snapshot of a virtual hard disk](snapshot-copy-managed-disk.md).
