# Exercise 3 (ex3): BPMN 2.0 Process Models & Failure Path Registers

This repository folder contains executable standard BPMN 2.0 XML process diagrams and comprehensive documentation for **Assignment 1** and **Assignment 3** modeled for **Camunda Modeler** and **Camunda Platform 7 / Camunda 8**.

---

## 📂 Directory Structure

```
camunda/ex3/
├── README.md                                   # Documentation & Element Specifications (This file)
├── assignment1_student_project_approval.bpmn   # Assignment 1 BPMN 2.0 Diagram (Student Project Approval)
└── assignment3_loan_origination_approval.bpmn  # Assignment 3 BPMN 2.0 Diagram (Loan Origination System)
```

---

## 📑 Summary of Assignments

### 1. Assignment 1: Student Project Approval & Allocation System (`assignment1_student_project_approval.bpmn`)
- **Business Purpose**: Automated verification of student project proposals, screening by project coordinator, committee evaluation, faculty guide workload matching, and allocation dispatch.
- **Participant Pools & Lanes**:
  - **Student / Team**: Submits proposals, resubmissions, and revised topics.
  - **Project Coordinator**: Screens proposals and manually allocates guides when automatic rules return no match.
  - **Review Committee (incl. HoD)**: Evaluates project feasibility, scope, and decides approval/revision/rejection.
  - **Faculty Guide**: Accepts or declines assigned student teams.
  - **Project Management System**: Service tasks for form validation, plagiarism checks, business rule matching, and notification dispatch.

#### Happy Path Flow (10 Steps)
1. **Student Team**: Submits project proposal (`User Task`).
2. **System**: Validates form completeness and team composition rules (`Service Task`).
3. **Coordinator**: Screens proposal and verifies domain compatibility (`User Task`).
4. **System**: Runs automated similarity & plagiarism check (`Service Task`).
5. **Review Committee**: Evaluates proposal; Exclusive Gateway routes outcome (`User Task` + `Exclusive Gateway`).
6. **System**: Business rule task checks guide availability and workload capacity against domain match (`Business Rule Task`).
7. **System**: Allocates matched faculty guide and dispatches allocation request (`Service Task` + `Send Task`).
8. **Faculty Guide**: Formally accepts the allocation request (`User Task`).
9. **System**: Sends confirmation notification to student team and records allocation (`Send Task`).
10. **End Event**: Project & Guide Formally Allocated (`None End Event`).

#### Failure Path Register (F1 – F11)
| ID | Failure Point | Cause / Trigger | Solution & BPMN Construct |
|:---|:---|:---|:---|
| **F1** | Proposal validation | Missing required fields, no abstract, or invalid roll numbers. | **Exclusive Gateway** after validation service task. Returns proposal with error list (`Task_ResubmitProposal`); student resubmits. Retry counter limits attempts to 2 before escalation. |
| **F2** | Team rule check | Team size exceeds limit or duplicate member registration. | Reject at validation stage; process terminates with **Error End Event (`EndEvent_InvalidTeam`)**. |
| **F3** | Duplicate / plagiarised topic | Similarity score above cutoff threshold against previous projects. | Coordinator reviews similarity report; requests topic modification or rejects. |
| **F4** | Proposal deadline | Student team fails to submit proposal before deadline. | **Timer Boundary Event** on submission task; late submission request routed to coordinator. |
| **F5** | Committee asks for revision | Proposal scope too large, objectives vague, or low novelty. | Revision loop (`Task_ReviseProposal`) guarded by 7-day SLA timer; max 2 cycles allowed. |
| **F6** | Committee rejects | Proposal not feasible, out of syllabus scope, or lacks resources. | Send rejection reasons; terminates with **End Event (`EndEvent_ProposalRejected`)**. |
| **F7** | Review delayed | Review committee fails to issue decision within SLA. | **Non-interrupting Timer Boundary Event** sends reminder; second timer raises escalation to HoD. |
| **F8** | No guide available | Domain mismatch or all matching faculty guides at maximum load. | Business rule task returns `"none"`. Coordinator executes manual allocation (`Task_ManualAllocation`) / co-guide / waitlist. |
| **F9** | Guide declines or is silent | Workload capacity, conflict of interest, or no response within SLA. | Timer plus decline path. Rerun allocation rule (3x limit, then coordinator allocates manually). |
| **F10** | Notification / system failure | Email service down or database write error. | **Error Boundary Event** on service task with exponential backoff retry (3x) + manual fallback. |
| **F11** | Withdrawal after approval | Student team dissolves or allocated guide goes on sabbatical. | **Message Boundary Event** on allocation sub-process; **Compensation Event** releases guide slot. |

---

### 2. Assignment 3: Loan Origination & Approval System (`assignment3_loan_origination_approval.bpmn`)
- **Business Purpose**: Comprehensive processing of personal and secured loan applications through identity verification, credit bureau scoring, risk underwriting, offer generation, e-signature, and funds disbursement.
- **Participant Pools & Lanes**:
  - **Applicant**: Submits applications, accepts offers, and e-signs agreements.
  - **Loan Officer**: Conducts manual document checks, identity verification, and income review.
  - **Underwriter / Credit Committee**: Reviews high-value, borderline, or complex cases.
  - **Fraud & Compliance Team**: Investigates AML, sanction list, and forgery alerts.
  - **Credit Bureau**: External service provider returning credit scores.
  - **Core Banking System**: Performs automated rules, risk scoring, and account creation.
  - **Operations / Disbursement**: Manages final funds transfer and account posting.

#### Happy Path Flow (10 Steps)
1. **Applicant**: Submits loan application and required supporting documents (`Start Event` / `User Task`).
2. **System**: Checks document completeness against minimum checklist (`Business Rule Task`).
3. **Parallel Gateway**: Runs KYC verification (`Service Task`), credit score retrieval (`Service Task` via external pool), and income verification (`User Task`) concurrently, then joins (`Parallel Gateway`).
4. **System**: Risk scoring and eligibility evaluation (`Business Rule Task`).
5. **Underwriter**: Evaluates risk profile and approves loan case (`User Task`).
6. **System**: Generates formal loan offer document (`Script Task`).
7. **Applicant**: Formally accepts loan offer (`Receive Task`).
8. **Applicant**: Digitally signs (e-signs) loan agreement (`User Task`).
9. **Operations / System**: Disburses funds to verified bank account, creates loan account, and sends confirmation (`Service Task`).
10. **End Event**: Loan Disbursed & Active (`None End Event`).

#### Failure Path Register (L1 – L12)
| ID | Failure Point | Cause / Trigger | Solution & BPMN Construct |
|:---|:---|:---|:---|
| **L1** | Document check | Incomplete or illegible document uploads. | Return application with checklist & 7-day timer; loop on resubmission; terminate if lapsed. |
| **L2** | KYC verification | Identity details do not match official government databases. | **Exclusive Gateway** routes to manual verification by Loan Officer; reject if unresolved. |
| **L3** | Fraud / AML screening | Forged documents, AML hit, or global sanctions list match. | **Escalation Event** to Fraud pool; investigation sub-process; **Terminate End Event**. |
| **L4** | Credit bureau call | API timeout or external credit bureau system downtime. | **Error Boundary Event**: retry 3 times at exponential backoff intervals; fallback to manual credit check. |
| **L5** | Credit score / risk | Credit score below minimum threshold. | **Exclusive Gateway**: reject or issue counter-offer requiring guarantor or collateral. |
| **L6** | Income / DTI | Insufficient verified income or DTI ratio exceeds ceiling. | Offer reduced loan amount or longer tenure & loop back to re-scoring (max 1 attempt). |
| **L7** | High-value / borderline case | Loan amount exceeds underwriter authority level or borderline score. | Escalate case to Senior Credit Committee with SLA timer; **Inclusive Gateway** adds extra checks. |
| **L8** | Collateral valuation | Appraisal valuation lower than requested loan amount or title defects. | Revalue asset, reduce LTV ratio, and re-underwrite; reject if title defects remain. |
| **L9** | Offer acceptance | Applicant fails to respond within offer validity period (15 days). | **Timer Boundary Event** on offer receive task; offer expires (`EndEvent_OfferExpired`). |
| **L10** | Withdrawal at any stage | Applicant cancels application prior to disbursement. | **Message Boundary Event** on main process; **Compensation Event** reverses fees and releases holds. |
| **L11** | E-signature | Agreement not signed or digital signature validation failure. | Resend agreement link & dispatch reminder notifications (max 2 attempts); cancel offer if failed. |
| **L12** | Disbursement | Invalid recipient bank account details or payment gateway error. | Account verification step, then retry disbursement; hold funds if persistent failure; compensation reverses account creation. |

---

## 🛠️ How to View in Camunda Modeler

1. Launch **Camunda Modeler** (`Camunda Modeler.exe`).
2. Select **File -> Open File...** and choose either:
   - `assignment1_student_project_approval.bpmn`
   - `assignment3_loan_origination_approval.bpmn`
3. Diagrams contain complete BPMNDI layout XML tags with 5+ participant pools and swimlanes for clear visual presentation.
