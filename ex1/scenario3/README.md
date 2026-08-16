# Scenario 3: IT Service Request Management

## 📋 Overview
This BPMN 2.0 process models an **Enterprise IT Support Request & Help Desk Management System**. The workflow captures IT ticket logging, dynamic technician assignment based on problem severity, internal vs. vendor escalation paths, status tracking, and resolution notification.

---

## ⚙️ Process Flow & Step-by-Step Requirements

1. **Start Event**: An employee encounters an IT issue (`StartEvent_ITProblemReported`).
2. **Submit IT Support Request**: The employee submits details via the self-service IT portal (`Task_SubmitITRequest`).
3. **Register Support Request**: The IT Help Desk system logs the ticket and generates a unique request ID (`Task_RegisterRequest`).
4. **Check Problem Severity**: System/Help desk evaluates ticket parameters to assess severity (`Task_CheckSeverity`).
5. **Exclusive Gateway - Severity Triage (`Gateway_SeverityCheck`)**:
   - **Path A (Low Severity)**: Assigned to a Tier-1 **Support Technician** (`Task_AssignSupportTech`) who conducts the investigation (`Task_InvestigateLowTech`).
   - **Path B (High Severity)**: Assigned directly to a **Senior Technician** (`Task_AssignSeniorTech`) who conducts emergency investigation (`Task_InvestigateSeniorTech`).
6. **Investigation Consolidation & Exclusive Gateway - Resolution Check (`Gateway_ResolutionCheck`)**:
   - **Path A1 (Internal Fix)**: If the issue can be resolved internally, the technician fixes the problem (`Task_FixInternally`).
   - **Path A2 (External Escalation)**: If the problem requires specialized third-party intervention, the ticket is escalated to the **External Service Provider** (`Task_EscalateExternal`).
7. **Resolution Merge & Status Update**:
   - Both resolution paths merge (`Gateway_MergeResolution`).
   - Help desk system updates the request status to "Resolved" in the ITSM portal (`Task_UpdateStatus`).
8. **Employee Notification**: The employee receives a resolution notification with resolution details (`Task_SendResolutionNotif`).
9. **Process Completion**: Process finishes cleanly at `EndEvent_ITRequestResolved`.

---

## 🛠️ BPMN Elements Mapping

| BPMN Element Type | Element ID | Label / Name | Type Details |
| :--- | :--- | :--- | :--- |
| **Start Event** | `StartEvent_ITProblemReported` | Employee Reports IT Problem | None Start Event |
| **User Task** | `Task_SubmitITRequest` | Submit IT Support Request | Employee Portal Task |
| **Service Task** | `Task_RegisterRequest` | Register Support Request | Camunda External Service Task |
| **Service Task** | `Task_CheckSeverity` | Check Severity of Problem | Camunda External Service Task |
| **Exclusive Gateway** | `Gateway_SeverityCheck` | Problem Severity? | XOR Gateway (Low / High) |
| **User Task** | `Task_AssignSupportTech` | Assign to Support Technician | Support Technician Task |
| **User Task** | `Task_AssignSeniorTech` | Assign to Senior Technician | Senior Technician Task |
| **User Task** | `Task_InvestigateLowTech` | Investigate Problem (Support Tech) | Technician Investigation |
| **User Task** | `Task_InvestigateSeniorTech` | Investigate Problem (Senior Tech) | Senior Tech Investigation |
| **Exclusive Gateway** | `Gateway_MergeInvestigation` | Merge Investigation | Merge Gateway |
| **Exclusive Gateway** | `Gateway_ResolutionCheck` | Can Problem Be Resolved Internally? | XOR Gateway (Yes / No) |
| **User Task** | `Task_FixInternally` | Fix Problem Internally | Internal Technician Task |
| **Service Task** | `Task_EscalateExternal` | Escalate Problem to External Service Provider | External Escalation Service |
| **Exclusive Gateway** | `Gateway_MergeResolution` | Merge Resolution | Merge Gateway |
| **Service Task** | `Task_UpdateStatus` | Update Request Status | Service Task |
| **Send Task** | `Task_SendResolutionNotif` | Send Resolution Notification to Employee | Send Task |
| **End Event** | `EndEvent_ITRequestResolved` | IT Request Resolved & Closed | None End Event |
