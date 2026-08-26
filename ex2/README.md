# Exercise 2: DMN Integration, Hit Policies & Dynamic Workflow Routing

Welcome to **Exercise 2 (ex2)** of the **Camunda Repository**. This directory contains standard BPMN 2.0 XML process diagrams, DMN 1.3 decision tables, test cases, and technical documentation for Question 1 and Question 2.

---

## 📂 Exercise 2 Directory Structure

```
ex2/
├── README.md                                   # Master Exercise 2 Overview (This document)
├── q1_loan_risk_routing/
│   ├── evaluate_loan_risk.dmn                  # Q1 DMN Decision Table (Hit Policy: Unique)
│   ├── q1_loan_risk_routing.bpmn               # Q1 BPMN 2.0 Process Diagram
│   └── q1_README.md                            # Q1 Detailed Specifications & FEEL Documentation
└── q2_order_discount_fulfillment/
    ├── calculate_discounts.dmn                 # Q2 DMN Decision Table (Hit Policy: Collect Sum C+)
    ├── q2_order_discount_fulfillment.bpmn      # Q2 BPMN 2.0 Process Diagram
    └── q2_README.md                            # Q2 Detailed Specifications & Test Verification
```

---

## 📑 Summary of Questions

### 1. Question 1: Loan Application Risk Routing
- **Directory**: [`q1_loan_risk_routing/`](q1_loan_risk_routing/)
- **BPMN Diagram**: [`q1_loan_risk_routing.bpmn`](q1_loan_risk_routing/q1_loan_risk_routing.bpmn)
- **DMN Table**: [`evaluate_loan_risk.dmn`](q1_loan_risk_routing/evaluate_loan_risk.dmn)
- **Documentation**: [`q1_README.md`](q1_loan_risk_routing/q1_README.md)
- **Key Concepts**:
  - Business Rule Task evaluating DMN table with `Single Result` result mapping.
  - Unique Hit Policy (`U`) in DMN table.
  - Multi-branch Exclusive Gateway (XOR) using FEEL condition expressions on decision output fields (`riskTier`, `requiresManualReview`).
  - Automated approval/disbursement vs. underwriter review vs. auto-rejection notification.

---

### 2. Question 2: Multi-Item Order Discount & Fulfillment Orchestration
- **Directory**: [`q2_order_discount_fulfillment/`](q2_order_discount_fulfillment/)
- **BPMN Diagram**: [`q2_order_discount_fulfillment.bpmn`](q2_order_discount_fulfillment/q2_order_discount_fulfillment.bpmn)
- **DMN Table**: [`calculate_discounts.dmn`](q2_order_discount_fulfillment/calculate_discounts.dmn)
- **Documentation**: [`q2_README.md`](q2_order_discount_fulfillment/q2_README.md)
- **Key Concepts**:
  - Multi-hit DMN evaluation using **Collect Sum (`C+`)** hit policy.
  - Mapping decision result via `Single Entry` (`totalDiscount`) because `C+` collapses matched output entries to a single scalar sum.
  - Script task calculating final invoice amount: `finalAmount = cartValue * (1 - (totalDiscount / 100))`.
  - Dynamic routing via XOR gateway (`finalAmount >= 1000` requires Manager Sign-off; `< 1000` proceeds directly to Warehouse).
  - Verified sample payload (`PREMIUM`, `600`, `FESTIVE10`) resulting in a `25%` total discount.

---

## 🛠️ How to Open & Test Files in Camunda Modeler

1. Launch **Camunda Modeler** (or execute `Camunda Modeler.exe`).
2. Click **File -> Open File...**.
3. Select any `.bpmn` or `.dmn` file inside `ex2/q1_loan_risk_routing/` or `ex2/q2_order_discount_fulfillment/`.
4. All diagram files contain clean visual layout coordinates (BPMNDI / DMNDI) ready for editing, deployment, or execution.
