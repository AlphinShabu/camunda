# Lab 1: Loan Approval Automation (Camunda 8)

Automates a loan application from start to finish: validate the applicant, check credit, decide, optionally get a human review, then disburse.

## How It Works

1. **Validate applicant** – a Spring Boot worker checks name, age (18+), residency and employment.
2. **Check credit** – a REST connector calls `POST /api/credit-assessment` and gets a credit score.
3. **Decide** – a DMN table returns **Approved**, **Manual Review** or **Rejected**.
4. **Human review** – borderline cases (or API errors) go to a Tasklist user task for an officer.
5. **Disburse** – approved loans are paid out by a Spring Boot worker.

## Tech Used

| Part | Tool |
|---|---|
| Workflow engine | Zeebe 8.x |
| Worker | Spring Boot |
| Credit API call | REST Connector (`io.camunda:http-json:1`) |
| Decision logic | DMN (`decision_loan_classification`) |
| Manual step | Camunda Tasklist |

## Files

- `loan_approval_process.bpmn` – the process diagram
- `loan_classification.dmn` – the decision table

## Key Variables

| Variable | Meaning | Example |
|---|---|---|
| `applicantId` | Applicant ID | `APP-8012` |
| `applicantAge` | Age (must be 18+) | `32` |
| `monthlyIncome` | Monthly salary (USD) | `6500.0` |
| `requestedAmount` | Loan amount asked | `35000.0` |
| `creditScore` | Score from credit API | `780` |
| `loanClassification` | DMN result | `Approved` |
| `interestRate` | Suggested rate (%) | `5.5` |
| `disbursementStatus` | Final status | `DISBURSED` |

## Decision Rules (Simplified)

| Credit Score | Income | Amount | Result | Rate |
|---|---|---|---|---|
| 750+ | ≥ 5000 | ≤ 50000 | Approved | 5.5% |
| 700+ | ≥ 7000 | > 50000 | Manual Review | 6.8% |
| 650–749 | ≥ 4000 | ≤ 25000 | Approved | 7.9% |
| 600–649 | ≥ 3000 | ≤ 30000 | Manual Review | 9.5% |
| any | < 2500 | any | Rejected | 0% |
| < 600 | any | any | Rejected | 0% |
| anything else | – | – | Manual Review | 10% |

## Credit API Call

**Request**
```json
{ "applicantId": "APP-8012", "monthlyIncome": 6500.0, "requestedAmount": 35000.0 }
```

**Response**
```json
{ "applicantId": "APP-8012", "creditScore": 780, "creditTier": "PRIME", "creditReportId": "CR-9921", "status": "SUCCESS" }
```

## Test Cases (All Verified ✅)

| ID | Scenario | Result |
|---|---|---|
| TC-L1-01 | Good applicant (score 780) | Auto-approved and disbursed |
| TC-L1-02 | Low income and score (540) | Rejected |
| TC-L1-03 | Large loan, score 720 | Manual review, then disbursed |
| TC-L1-04 | Credit API down | Routed to manual review |
| TC-L1-05 | Worker fails (`simulateFailure=true`) | Incident in Operate, fixed, retried, completed |

