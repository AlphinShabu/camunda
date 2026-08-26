# Exercise 2: DMN Integration, Hit Policies & Dynamic Workflow Routing


##  Summary of Questions

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




