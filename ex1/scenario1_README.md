# Scenario 1: Employee Leave Approval

##  Overview
This BPMN 2.0 process models an automated and manager-mediated **Employee Leave Approval System**. The process ensures that an employee's leave balance is verified prior to requesting managerial review and handles all subsequent balance updates and employee notifications.

---

##  Process Flow & Step-by-Step Requirements

1. **Start Event**: The process begins when an employee submits a leave request through the company's HR system (`StartEvent_LeaveRequested`).
2. **Leave Balance Verification**: The HR system executes a service task (`Task_CheckBalance`) to evaluate the employee's remaining leave entitlement against the requested days.
3. **Exclusive Gateway - Balance Evaluation (`Gateway_BalanceCheck`)**:
   - **Path A (Insufficient Balance)**: If the leave balance is less than the requested amount, the process routes to `Task_SendInsufficientNotif`, sending an automated notification to the employee. The process terminates at `EndEvent_InsufficientBalance`.
   - **Path B (Sufficient Balance)**: If sufficient leave balance exists, the process routes the leave request to the manager's task queue (`Task_ManagerApproval`).
4. **Manager Approval & Exclusive Gateway (`Gateway_ManagerDecision`)**:
   - **Path B1 (Approved)**: If the manager approves the leave request:
     - The system updates the employee's leave balance record in the HR database (`Task_UpdateBalance`).
     - An approval notification email/message is dispatched to the employee (`Task_SendApprovalNotif`).
     - The process ends successfully at `EndEvent_Approved`.
   - **Path B2 (Rejected)**: If the manager rejects the leave request:
     - A rejection notification with optional comments is sent to the employee (`Task_SendRejectionNotif`).
     - The process terminates at `EndEvent_Rejected`.


## Visual Process Diagram Architecture

```
[Start: Leave Requested] ──> [Check Leave Balance] ──> <Sufficient Balance?>
                                                            │
                                        ┌───────────────────┴───────────────────┐
                                  (No / Insufficient)                       (Yes / Sufficient)
                                        │                                       │
                                        ▼                                       ▼
                       [Send Insufficient Notification]            [Manager Review & Approval]
                                        │                                       │
                                        ▼                                       ▼
                            (End: Insufficient Balance)                <Manager Decision?>
                                                                                │
                                                            ┌───────────────────┴───────────────────┐
                                                       (Approved)                               (Rejected)
                                                            │                                       │
                                                            ▼                                       ▼
                                               [Update Leave Balance]                    [Send Rejection Notification]
                                                            │                                       │
                                                            ▼                                       ▼
                                               [Send Approval Notification]               (End: Leave Rejected)
                                                            │
                                                            ▼
                                                (End: Leave Approved)


