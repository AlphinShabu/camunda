# LAB 1: End-to-End Loan Approval Automation

## 1. Architecture Overview
This workflow orchestrates an end-to-end loan application approval cycle on Camunda 8.
- **Workflow Engine**: Zeebe 8.x
- **External Worker (Spring Boot)**: Validates applicant metadata (name, age, residency, employment status).
- **REST Connector (`io.camunda:http-json:1`)**: Invokes simulated credit-assessment REST API (`/api/credit-assessment`).
- **DMN Decision Table (`decision_loan_classification`)**: Classifies applicants into `Approved`, `Manual Review`, or `Rejected` based on credit score, monthly income, and requested loan amount.
- **Human Review**: Interactive Tasklist User Task for manual officer assessment on borderline or API error conditions.
- **Disbursement Service Task**: Auto-disburses approved loans through Spring Boot worker.

---

## 2. BPMN & DMN Artifacts
- **BPMN Model**: [`loan_approval_process.bpmn`](loan_approval_process.bpmn)
- **DMN Model**: [`loan_classification.dmn`](loan_classification.dmn)

---

## 3. Process Variables Dictionary

| Variable Name | Type | Direction | Purpose | Example |
|---|---|---|---|---|
| `applicantId` | String | Input | Unique applicant reference | `"APP-8012"` |
| `applicantName` | String | Input | Full name of applicant | `"Alice Johnson"` |
| `applicantAge` | Integer | Input | Age of applicant (eligibility >= 18) | `32` |
| `monthlyIncome` | Double | Input | Verified gross monthly salary in USD | `6500.0` |
| `requestedAmount` | Double | Input | Loan principal requested | `35000.0` |
| `employmentStatus` | String | Input | Employment state (`EMPLOYED`, `SELF_EMPLOYED`, `UNEMPLOYED`) | `"EMPLOYED"` |
| `isValidApplicant` | Boolean | Output (Worker) | Validation flag set by Spring Boot worker | `true` |
| `validationMessage`| String | Output (Worker) | Diagnostic details if validation failed | `"Applicant passed criteria"` |
| `creditScore` | Integer | Output (REST API) | Credit score returned from credit bureau | `780` |
| `creditTier` | String | Output (REST API) | Credit tier (`PRIME`, `STANDARD`, `SUBPRIME`) | `"PRIME"` |
| `loanClassification` | String | Output (DMN) | `Approved`, `Manual Review`, or `Rejected` | `"Approved"` |
| `riskRating` | String | Output (DMN) | Risk evaluation (`LOW`, `MEDIUM`, `HIGH`) | `"LOW"` |
| `interestRate` | Double | Output (DMN) | Recommended interest rate percentage | `5.5` |
| `officerDecision` | String | User Task Input | Decision submitted by credit officer | `"Approve"` |
| `officerComments` | String | User Task Input | Justification notes from human reviewer | `"Solid collateral"` |
| `disbursementStatus`| String | Output (Worker) | Final settlement confirmation | `"DISBURSED"` |

---

## 4. DMN Decision Table Evaluation Logic

| Rule | Credit Score | Monthly Income | Requested Amount | Classification | Risk Rating | Interest Rate |
|---|---|---|---|---|---|---|
| **1** | `>= 750` | `>= 5000` | `<= 50000` | `"Approved"` | `"LOW"` | `5.5%` |
| **2** | `>= 700` | `>= 7000` | `> 50000` | `"Manual Review"` | `"MEDIUM"` | `6.8%` |
| **3** | `[650..749]` | `>= 4000` | `<= 25000` | `"Approved"` | `"MEDIUM"` | `7.9%` |
| **4** | `[600..649]` | `>= 3000` | `<= 30000` | `"Manual Review"` | `"MEDIUM_HIGH"` | `9.5%` |
| **5** | `-` | `< 2500` | `-` | `"Rejected"` | `"HIGH"` | `0.0%` |
| **6** | `< 600` | `-` | `-` | `"Rejected"` | `"HIGH"` | `0.0%` |
| **7** | `-` | `-` | `-` (Fallback) | `"Manual Review"` | `"HIGH"` | `10.0%` |

---

## 5. REST Connector Specification
- **Task Type**: `io.camunda:http-json:1`
- **Method**: `POST`
- **URL**: `http://localhost:8080/api/credit-assessment`
- **Request Payload**:
  ```json
  {
    "applicantId": "APP-8012",
    "monthlyIncome": 6500.0,
    "requestedAmount": 35000.0
  }
  ```
- **Response Payload (Success)**:
  ```json
  {
    "applicantId": "APP-8012",
    "creditScore": 780,
    "creditTier": "PRIME",
    "creditReportId": "CR-9921",
    "status": "SUCCESS"
  }
  ```
- **Response Mapping (`resultExpression`)**:
  ```feel
  ={creditScore: response.body.creditScore, creditTier: response.body.creditTier, creditReportId: response.body.creditReportId}
  ```

---

## 6. Test-Case Matrix (Minimum Test Scenarios)

| Case ID | Scenario Description | Inputs (`applicantAge`, `monthlyIncome`, `requestedAmount`) | Expected Route & Outcome | Actual Status |
|---|---|---|---|---|
| **TC-L1-01** | **Approval Criteria Met** | Age: 32, Income: \$6500, Amount: \$35000, Credit: 780 | Auto-approved -> Auto Disburse Loan -> Approved | Verified |
| **TC-L1-02** | **Failing Eligibility** | Age: 26, Income: \$1800, Amount: \$15000, Credit: 540 | DMN Rule 5 & 6 match -> Rejected -> End | Verified |
| **TC-L1-03** | **Manual Review Required** | Age: 41, Income: \$7500, Amount: \$75000, Credit: 720 | DMN Rule 2 match -> Tasklist User Task -> Officer Approval -> Disbursed | Verified |
| **TC-L1-04** | **Credit API Error / Timeout** | Simulated downtime on Credit Assessment endpoint | Boundary Error Event triggers -> Routes to Manual Loan Review in Tasklist | Verified |
| **TC-L1-05** | **Worker Incident & Recovery** | Validation worker simulated exception (`simulateFailure=true`) | Incident raised in Operate -> Worker issue resolved / restarted -> Job retried in Operate -> Completed successfully | Verified |
