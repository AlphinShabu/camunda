# Question 1: Loan Application Risk Routing Specification

## 📌 Executive Summary
This document contains the complete technical specification, BPMN 2.0 flow design, DMN decision table configuration, and FEEL condition expressions for **Exercise 1 / Question 1: Loan Application Risk Routing**.

---

## 📂 File Artifacts
- **BPMN 2.0 Process Diagram**: [`q1_loan_risk_routing.bpmn`](q1_loan_risk_routing.bpmn)
- **DMN 1.3 Decision Table**: [`evaluate_loan_risk.dmn`](evaluate_loan_risk.dmn)

---

## ⚙️ 1. BPMN Process Workflow

### Process Key & Name
- **Process ID**: `Process_LoanApplicationRiskRouting`
- **Name**: `Loan Application Risk Routing Process`

### Process Steps & Data Flow
1. **Start Event (`StartEvent_LoanSubmitted`)**:
   - **Label**: `Loan Application Submitted`
   - **Incoming Payload**: `applicantAge` (integer), `creditScore` (integer), `loanAmount` (double)

2. **Business Rule Task (`BusinessRuleTask_EvaluateLoanRisk`)**:
   - **Label**: `Evaluate Loan Risk`
   - **Decision Reference**: `evaluate-loan-risk`
   - **Map Decision Result**: `Single Result` (`singleResult`)
   - **Result Variable**: `riskResult`
   - **Description**: Invokes the DMN decision table `evaluate-loan-risk` with parameters `creditScore` and `loanAmount`, returning a single object containing `riskTier` and `requiresManualReview`.

3. **Exclusive Gateway (XOR) (`Gateway_RiskRouting`)**:
   - **Label**: `Risk Tier & Review Evaluation`
   - **Routing Branches**:
     - **Branch 1 (Auto-Approve)**:
       - **Flow Name**: `riskTier == 'LOW'`
       - **FEEL / EL Expression**: `${riskResult.riskTier == 'LOW'}` (or `= riskResult.riskTier = "LOW"`)
       - **Target Task**: Service Task `Auto-Approve and Disburse`
       - **Target End Event**: `Loan Approved and Disbursed`
     - **Branch 2 (Underwriter Review)**:
       - **Flow Name**: `riskTier == 'MEDIUM' or requiresManualReview == true`
       - **FEEL / EL Expression**: `${riskResult.riskTier == 'MEDIUM' || riskResult.requiresManualReview == true}`
       - **Target Task**: User Task `Underwriter Review`
       - **Target End Event**: `Underwriter Review Completed`
     - **Branch 3 (Auto-Reject)**:
       - **Flow Name**: `riskTier == 'HIGH' and requiresManualReview == false`
       - **FEEL / EL Expression**: `${riskResult.riskTier == 'HIGH' && riskResult.requiresManualReview == false}`
       - **Target Task**: Service Task `Auto-Reject Notification`
       - **Target End Event**: `Loan Auto-Rejected`

---

## 📊 2. DMN Decision Table Specifications (`evaluate-loan-risk`)

### Decision Properties
- **Decision ID**: `evaluate-loan-risk`
- **Hit Policy**: `Unique` (`U`) - Rules are mutually exclusive; exactly one rule matches for any given input combination.

### Inputs & Outputs Table

| Type | Label | Identifier | Data Type | FEEL Unary Test / Expression |
|---|---|---|---|---|
| **Input 1** | Credit Score | `creditScore` | `integer` | e.g., `>= 750`, `[600..749]`, `< 600` |
| **Input 2** | Loan Amount | `loanAmount` | `double` | e.g., `<= 50000`, `> 50000`, `-` |
| **Output 1** | Risk Tier | `riskTier` | `string` | `"LOW"`, `"MEDIUM"`, `"HIGH"` |
| **Output 2** | Requires Manual Review | `requiresManualReview` | `boolean` | `true`, `false` |

### Decision Rules Table

| Rule # | Credit Score (`creditScore`) | Loan Amount (`loanAmount`) | Risk Tier (`riskTier`) | Requires Manual Review (`requiresManualReview`) | Business Rationale |
|---|---|---|---|---|---|
| **1** | `>= 750` | `<= 50000` | `"LOW"` | `false` | High credit score and modest loan amount $\rightarrow$ Instant auto-approval. |
| **2** | `>= 750` | `> 50000` | `"MEDIUM"` | `true` | High credit score but high loan value $\rightarrow$ Underwriter review. |
| **3** | `[600..749]` | `-` (Any) | `"MEDIUM"` | `true` | Moderate credit score regardless of loan amount $\rightarrow$ Underwriter review. |
| **4** | `< 600` | `-` (Any) | `"HIGH"` | `false` | Low credit score $\rightarrow$ Automated instant rejection. |

---

## 🧪 3. Verification Scenarios & Expected Routing

| Test Scenario | Payload Inputs | Matched DMN Rule | Decision Output | XOR Gateway Branch | Final Outcome |
|---|---|---|---|---|---|
| **Scenario 1A** | `creditScore: 780`, `loanAmount: 35000` | Rule 1 | `riskTier="LOW"`, `requiresManualReview=false` | Flow 1 (`LOW`) | Service Task: `Auto-Approve and Disburse` |
| **Scenario 1B** | `creditScore: 800`, `loanAmount: 75000` | Rule 2 | `riskTier="MEDIUM"`, `requiresManualReview=true` | Flow 2 (`MEDIUM` / `review=true`) | User Task: `Underwriter Review` |
| **Scenario 1C** | `creditScore: 680`, `loanAmount: 20000` | Rule 3 | `riskTier="MEDIUM"`, `requiresManualReview=true` | Flow 2 (`MEDIUM` / `review=true`) | User Task: `Underwriter Review` |
| **Scenario 1D** | `creditScore: 550`, `loanAmount: 10000` | Rule 4 | `riskTier="HIGH"`, `requiresManualReview=false` | Flow 3 (`HIGH` & `review=false`) | Service Task: `Auto-Reject Notification` |
