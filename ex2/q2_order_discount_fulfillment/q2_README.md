# Question 2: Multi-Item Order Discount & Fulfillment Orchestration Specification

## 📌 Executive Summary
This document contains the complete technical specification, BPMN 2.0 flow design, DMN decision table configuration with **Collect Sum (`C+`)** hit policy, script task invoice formula calculations, and test payload verification for **Exercise 2 / Question 2**.

---

## 📂 File Artifacts
- **BPMN 2.0 Process Diagram**: [`q2_order_discount_fulfillment.bpmn`](q2_order_discount_fulfillment.bpmn)
- **DMN 1.3 Decision Table**: [`calculate_discounts.dmn`](calculate_discounts.dmn)

---

## ⚙️ 1. BPMN Process Workflow

### Process Key & Name
- **Process ID**: `Process_OrderDiscountFulfillment`
- **Name**: `Multi-Item Order Discount & Fulfillment Process`

### Process Steps & Data Flow
1. **Start Event (`StartEvent_OrderReceived`)**:
   - **Label**: `Order Received`
   - **Incoming Payload**: `customerTier` ("REGULAR", "PREMIUM"), `cartValue` (double), `promoCode` (string)

2. **Business Rule Task (`BusinessRuleTask_CalculateDiscount`)**:
   - **Label**: `Calculate Total Discount`
   - **Decision Reference**: `calculate-discounts`
   - **Map Decision Result**: `Single Entry` (`singleEntry`)
   - **Result Variable**: `totalDiscount`
   - **Rationale**: Hit policy `C+` (Collect Sum) automatically aggregates all matching rule output percentages into a single scalar numeric sum (e.g. `25`). Thus `singleEntry` extracts this sum directly into `totalDiscount`.

3. **Script / Service Task (`ScriptTask_CalculateFinalAmount`)**:
   - **Label**: `Calculate Final Invoice Amount`
   - **Result Variable**: `finalAmount`
   - **Formula**: `finalAmount = cartValue * (1 - (totalDiscount / 100))`
   - **Script (JavaScript)**:
     ```javascript
     var calculatedFinal = cartValue * (1.0 - (totalDiscount / 100.0));
     execution.setVariable("finalAmount", calculatedFinal);
     ```

4. **Exclusive Gateway (XOR) (`Gateway_InvoiceCheck`)**:
   - **Label**: `Check Invoice Amount`
   - **Routing Branches**:
     - **Branch 1 (Manager Sign-off)**:
       - **Condition**: `${finalAmount >= 1000}`
       - **Target Task**: User Task `Manager Sign-off` $\rightarrow$ Service Task `Send Order to Warehouse`
     - **Branch 2 (Proceed Directly)**:
       - **Condition**: `${finalAmount < 1000}`
       - **Target Task**: Service Task `Send Order to Warehouse`

5. **Service Task & End Event**:
   - **Service Task (`ServiceTask_SendToWarehouse`)**: `Send Order to Warehouse`
   - **End Event (`EndEvent_OrderFulfilled`)**: `Order Completed`

---

## 📊 2. DMN Decision Table Specifications (`calculate-discounts`)

### Decision Properties
- **Decision ID**: `calculate-discounts`
- **Hit Policy**: `Collect Sum` (`C+` / `hitPolicy="COLLECT" aggregation="SUM"`) - Allows multiple rules to match concurrently and sums up their `discountPercent` values.

### Inputs & Outputs Table

| Type | Label | Identifier | Data Type | FEEL Unary Test / Allowed Values |
|---|---|---|---|---|
| **Input 1** | Customer Tier | `customerTier` | `string` | `"PREMIUM"`, `"REGULAR"`, `-` |
| **Input 2** | Cart Value | `cartValue` | `double` | `>= 500`, `< 500`, `-` |
| **Input 3** | Promo Code | `promoCode` | `string` | `"FESTIVE10"`, `"SPECIAL10"`, `-` |
| **Output 1** | Discount Percent | `discountPercent` | `integer` | Additional discount percentage (e.g., `10`, `5`, `0`) |

### Decision Rules Table

| Rule # | Customer Tier (`customerTier`) | Cart Value (`cartValue`) | Promo Code (`promoCode`) | Discount Added (`discountPercent`) | Rule Description |
|---|---|---|---|---|---|
| **1** | `"PREMIUM"` | `-` (Any) | `-` (Any) | `+10` | 10% discount for Premium customers |
| **2** | `-` (Any) | `>= 500` | `-` (Any) | `+5` | 5% discount for high-value carts (>= $500) |
| **3** | `-` (Any) | `-` (Any) | `"FESTIVE10", "SPECIAL10"` | `+10` | 10% discount for valid promo code |
| **4** | `-` (Any) | `-` (Any) | `-` (Any) | `+0` | Default rule (matches all cases) |

---

## 🧪 3. Student Task 3: Test Payload & Step-by-Step Verification

### Sample Input Payload
```json
{
  "customerTier": "PREMIUM",
  "cartValue": 600.0,
  "promoCode": "FESTIVE10"
}
```

### Execution Steps & Results
1. **DMN Rule Evaluation (`calculate-discounts`)**:
   - **Rule 1**: `customerTier == "PREMIUM"` $\rightarrow$ **MATCHED** $\rightarrow$ Adds `10%`
   - **Rule 2**: `cartValue (600.0) >= 500` $\rightarrow$ **MATCHED** $\rightarrow$ Adds `5%`
   - **Rule 3**: `promoCode ("FESTIVE10")` in `("FESTIVE10", "SPECIAL10")` $\rightarrow$ **MATCHED** $\rightarrow$ Adds `10%`
   - **Rule 4**: Default rule $\rightarrow$ **MATCHED** $\rightarrow$ Adds `0%`
2. **Collect Sum Aggregation (`C+`)**:
   $$\text{totalDiscount} = 10 + 5 + 10 + 0 = 25\%$$
   - Result mapped via `singleEntry` into process variable `totalDiscount = 25`.
3. **Script Task Calculation**:
   $$\text{finalAmount} = 600 \times \left(1 - \frac{25}{100}\right) = 600 \times 0.75 = 450.0$$
4. **XOR Gateway Evaluation**:
   - Condition `${finalAmount >= 1000}` ($450.0 \ge 1000$) $\rightarrow$ `false`
   - Condition `${finalAmount < 1000}` ($450.0 < 1000$) $\rightarrow$ `true`
5. **Flow Routing**:
   - Bypasses `Manager Sign-off` user task.
   - Proceeds directly to `Send Order to Warehouse` service task and completes at `Order Completed` end event.
