---
title: Troubleshooting guide for customers to investigate and analyze, using Service Fabric Explorer (SFX), why repair jobs are not being approved and determine the next steps 
description: Discover best practices for analyzing a stuck repair job in Service Fabric Explorer (SFX).
ms.topic: concept-article
ms.author: ashukumar
author: ashukumar
ms.service: azure-service-fabric
services: service-fabric
ms.date: 12/11/2025
# Customer intent: As a customer, I want to identify where the repair job is stuck using Service Fabric Explorer (SFX).
---

# Troubleshooting guide for customers to investigate and analyze, using Service Fabric Explorer (SFX), why repair jobs are not being approved and determine the next steps

## Repair Task overview

Any operation initiated from the Virtual Machine Scale Set (VMSS) that targets VMs is processed by Service Fabric (SF) as a repair task derived from the job it receives. The Infrastructure Service creates a repair task for each job and enriches it with details such as the update type, targeted update domain (UD), and document incarnation number.
These jobs begin with UD0 and progress sequentially through UD1, UD2, and so on within the Service Fabric cluster. If an Update Domain walk is required, separate repair tasks are generated for each UD. For example, in a cluster with five UDs, five distinct repair tasks will be created. These tasks execute one after another, UD by UD, and their progress can be tracked in Service Fabric Explorer (SFX).
RepairManager – Repair Manager (RM) defines and implements a safe workflow for performing repairs by coordinating between the Repair Requestor, Repair Executor, and itself to ensure safe and consistent repair actions.
Infrastructure Service – Infrastructure Service (IS) is responsible for managing and orchestrating infrastructure-level operations, such as updates and repairs, ensuring the health and stability of the Service Fabric cluster.

### Repair Task vs. Repair Job Conversion
A repair job is the GUID provided by the Policy Engine to Service Fabric. Service Fabric then creates a repair task by combining details such as:

Repair type
Target update domain
Document incarnation number

These elements are combined in the following format:
Azure/<repair type>/<repair job>/<update domain>/<document incarnation number>
Example:
Azure/TenantUpdate/addfb79e-1e8c-42c8-a967-b0e2e0afd6b4/0/110

The resulting entity is the repair task, which is used within the Service Fabric context. In contrast, the repair job is recognized outside Service Fabric components.

## Repair task states and their ownership

Created
In the Created state, the Repair Manager (RM) accepts and stores the repair request. At this point, the task is waiting for a Repair Executor (RE) to claim it. The requestor can cancel the task during this stage without any restrictions. Repair manager has ownership in this state.

Claimed 
Once the task is Claimed, the Repair Executor (RE) has taken ownership but has not yet specified the repair’s impact. The requestor still retains the ability to cancel the task at this stage. Repair executor has ownership in this state.

Preparing (RM)
In the Preparing state, the Repair Executor specifies the impact, and the Repair Manager prepares the environment, such as deactivating nodes. If the task is cancelled now, it skips execution and moves directly to restoring. Operators also have the option to force approval, bypassing certain safety checks. Repair Manager has ownership in this state.

Approved (RE)
When the task reaches Approved, the Repair Manager has completed all preparations and approved execution. The Repair Executor should move the task to Executing before starting the repair. Cancellation at this point requires cooperation from the executor, which should support cancellation if possible. Repair Executor has ownership in this state.

Executing (RE)
During Executing, the Repair Executor is actively performing the repair. The executor must finish all disruptive actions before reporting completion. Cancellation now requires executor cooperation and should only be acknowledged when it is safe to do so. Repair executor has ownership in this state.

Restoring (RM)
Once the repair is complete, the task enters Restoring, where the Repair Manager restores the environment, such as reactivating nodes. At this stage, the task cannot be cancelled. Repair Manager has ownership in this state.

Completed
Finally, in the Completed state, the task is finished and no further state changes occur. The final status may be Succeeded, Cancelled, interrupted (with details), or Failed (with details).

## Analysis from Service Fabric Explorer For stuck repair task

### Infrastructure Jobs view

To view jobs that have been submitted to Service Fabric for approval, navigate to the Infrastructure Jobs tab under cluster view. Each entry includes a Job ID, which remains consistent across Service Fabric as well as outside service fabric. The Acknowledgement Status indicates whether the job has been approved by Service Fabric:
•	WaitingForAcknowledgement means the job is still pending approval.
•	Acknowledged confirms that the job has been approved by Service Fabric.
This view represents perspective of the job. Jobs will only appear here when they are active states in the Service Fabric (SF) side.

In addition to the Job ID and Acknowledgement Status, the Impact Type section displays the nature of the job’s impact—for example, Instance Removal Impact.
The Current Repair Task section shows which repair task is actively running for the job approval on the Service Fabric side.
By selecting All Repair Tasks, you can view the status of every repair task associated with the current job.


 ![Infrastructure Job view](./media/TSG-for-customers-to-investigate-and-analyze-updates-using-SFX/Infrastructure-job-view.png)


### Repair Jobs and health check view

To view individual and all repair tasks associated with a cluster, go to the Repair Jobs tab. This section displays both pending repair tasks and those that have been completed or cancelled.
For pending tasks, you can see their current state. A repair task is not yet approved by Service Fabric if its state is Created, Claimed, or Preparing.
Once a repair task transitions to the Approved state, it is considered approved by Service Fabric, and the approval is then forwarded to the Repair Executor for the corresponding job.

 ![Repair Task view ](./media/TSG-for-customers-to-investigate-and-analyze-updates-using-SFX/repair-task-view.png)

If a repair task is stuck in the Preparing state, there are two possible reasons:
It could be stuck in either a Health Check or a Safety Check. Unhealthy entity in the cluster , including customer applications as well as system applications can cause the health check to not be green.
To determine if it's stuck in a Health Check, first verify whether Preparing or Restoring Health Check is enabled—based on the state where the task is stuck.
In the Repair Task view, expanding the task will show the Health Check status, indicating whether it is enabled.



![Health check view](./media/TSG-for-customers-to-investigate-and-analyze-updates-using-SFX/Health-check.png)

If enabled, the Repair Task History will show that the Health Check started but did not complete, confirming that the task is stuck in the Health Check phase—as illustrated in the screenshot above.

### Safety check view

A repair task can get stuck in the Safety Check phase only if it has an impact on any node. This can be confirmed by checking the Impact section in the Repair Task view, as shown in the previous screenshot.
If node impact is present, you can identify which Safety Check is causing the delay by inspecting each impacted node individually. Click on the node from the Node List, and in the Safety Check section, you’ll see the specific check where the task is stuck.
The Repair Task ID is also displayed here, indicating which repair task is responsible for the node deactivation and safety check—this is illustrated in the screenshot below.
In the screenshot below, the repair task is stuck in the EnsureSeedNodeQuorum safety check.

![safety check view](./media/TSG-for-customers-to-investigate-and-analyze-updates-using-SFX/safety-check-view.png)

If there are no errors in Infrastructure Service (IS) related to a repair task and the task has entered the Executing state, it means the job’s acknowledgment status is 'Acknowledged' for Impact Start.
Similarly, if the repair task transitions to the Completed state, it indicates that the job’s acknowledgment status is 'Acknowledged' for Impact End.

![Repair task executing view](./media/TSG-for-customers-to-investigate-and-analyze-updates-using-SFX/Repair-task-executing.png)

All completed or cancelled repair tasks for the cluster can be viewed by clicking on the Completed Repair Tasks section. This provides a comprehensive list of repair tasks that have either successfully finished or were terminated.

![Repair task executing view](./media/TSG-for-customers-to-investigate-and-analyze-updates-using-SFX/completed-repair-task-view.png)

### Scenario and DRI Involvement

## Reference Table

| DRI | Scenario |
|-------------------|-------------|
| SF DRI | 
1.	Infrastructure Service Warning or Missing
If the Service Fabric Infrastructure Service is in a warning state or missing for the corresponding tenant, updates may fail to reach Service Fabric, resulting in pending approvals.
2.	Repair Task Stuck in Health or Safety Check
A repair task may remain stuck in the health check or safety check phase until the respective conditions are satisfied.
3.	Repair Manager in Error State
When the Repair Manager is in an error state, all repairs stored within it cannot progress because state changes are managed by the Repair Manager. Consequently, update statuses will not advance.
4.	Throttling Job Status
Verify the status of the throttling job. If another job is already in progress, incoming jobs may be throttled and delayed.|
| AZPE DRI | 1.	If the Infrastructure Service is healthy for the corresponding tenant, but no repair task appears in Service Fabric Explorer (SFX) when one is expected, and there is no throttled job for that tenant, contact the AZPE DRI to investigate why the job has not been initiated.|
| Blackbird DRI | 1.	If the repair task remains in the Executing state for an extended period while the Infrastructure Service is healthy, it indicates that the task has been approved by Service Fabric. In this case, the issue should be checked with the Blackbird DRI. |
