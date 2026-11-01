# LAB 2: End-to-End E-Commerce Order Processing and Fulfilment

## 1. Architecture Overview
This workflow automates order validation, stock checking, order priority classification via DMN, payment processing using REST Connector, manual verification on exceptions, and order fulfillment.
- **Workflow Engine**: Zeebe 8.x
- **External Worker (Spring Boot)**:
  - `validate-inventory-worker`: Validates order fields, verifies mock warehouse inventory, and computes order total (`unitPrice * quantity`).
  - `fulfill-order-worker`: Generates tracking number, reserves stock, and triggers warehouse dispatch.
- **REST Connector (`io.camunda:http-json:1`)**: Invokes simulated Payment Gateway API (`/api/payment`).
- **DMN Decision Table (`decision_order_priority`)**: Evaluates `orderTotal`, `customerCategory`, and `deliveryPreference` to set delivery priority, shipping lane, and flag high-value orders for manual audit.
- **Human Verification**: Interactive Tasklist User Task for manual verification of high-value or exception orders.

---

## 2. BPMN & DMN Artifacts
- **BPMN Model**: [`order_processing_process.bpmn`](order_processing_process.bpmn)
- **DMN Model**: [`order_priority.dmn`](order_priority.dmn)

---

## 3. Process Variables Dictionary

| Variable Name | Type | Direction | Purpose | Example |
|---|---|---|---|---|
| `orderId` | String | Input | Unique order reference | `"ORD-9001"` |
| `customerRef` | String | Input | Customer identifier | `"CUST-4421"` |
| `productId` | String | Input | Product SKU requested | `"PROD-LAPTOP-X1"` |
| `quantity` | Integer | Input | Number of units ordered | `2` |
| `customerCategory` | String | Input | Customer status (`REGULAR`, `VIP`, `PLATINUM`) | `"VIP"` |
| `deliveryPreference`| String | Input | Delivery service level (`STANDARD`, `EXPRESS`, `SAME_DAY`) | `"EXPRESS"` |
| `paymentToken` | String | Input | Synthetic token representing customer payment card | `"tok_synthetic_visa_4242"` |
| `isStockAvailable` | Boolean | Output (Worker) | Stock check boolean result from inventory system | `true` |
| `unitPrice` | Double | Output (Worker) | Product base price | `750.0` |
| `orderTotal` | Double | Output (Worker) | Calculated order total (`unitPrice * quantity`) | `1500.0` |
| `deliveryPriority` | String | Output (DMN) | Order priority level (`LOW`, `STANDARD`, `HIGH`, `CRITICAL`) | `"CRITICAL"` |
| `shippingLane` | String | Output (DMN) | Warehouse fulfillment shipping route | `"AIR_EXPRESS"` |
| `requiresManualVerification` | Boolean | Output (DMN) | Flag indicating high-value threshold trigger | `true` |
| `isAuditorApproved` | Boolean | User Task Input | Auditor confirmation from Tasklist | `true` |
| `paymentStatus` | String | Output (REST API) | Payment outcome (`SUCCESS`, `DECLINED`, `FRAUD_ALERT`) | `"SUCCESS"` |
| `transactionId` | String | Output (REST API) | Payment gateway transaction reference | `"TXN-88231"` |
| `trackingNumber` | String | Output (Worker) | Carrier logistics tracking number | `"TRK-983271-EXP"` |

---

## 4. DMN Decision Table Evaluation Logic

| Rule | Order Total | Customer Category | Delivery Preference | Delivery Priority | Shipping Lane | Requires Manual Verification |
|---|---|---|---|---|---|---|
| **1** | `>= 1000` | `-` | `-` | `"EXPRESS"` | `"SECURED_COURIER"` | `true` (High Value) |
| **2** | `-` | `"VIP", "PLATINUM"` | `"EXPRESS", "SAME_DAY"` | `"CRITICAL"` | `"AIR_EXPRESS"` | `false` |
| **3** | `-` | `"REGULAR"` | `"EXPRESS"` | `"HIGH"` | `"PRIORITY_GROUND"` | `false` |
| **4** | `[100..999]` | `-` | `"STANDARD"` | `"STANDARD"` | `"STANDARD_GROUND"` | `false` |
| **5** | `-` | `-` | `-` (Fallback) | `"LOW"` | `"ECONOMY_POST"` | `false` |

---

## 5. REST Connector Specification
- **Task Type**: `io.camunda:http-json:1`
- **Method**: `POST`
- **URL**: `http://localhost:8080/api/payment`
- **Request Payload**:
  ```json
  {
    "orderId": "ORD-9001",
    "customerRef": "CUST-4421",
    "orderTotal": 1500.0,
    "paymentToken": "tok_synthetic_visa_4242"
  }
  ```
- **Response Mapping (`resultExpression`)**:
  ```feel
  ={paymentStatus: response.body.paymentStatus, transactionId: response.body.transactionId, paymentMessage: response.body.message}
  ```

---

## 6. Test-Case Matrix (Minimum Test Scenarios)

| Case ID | Scenario Description | Inputs | Expected Route & Outcome | Actual Status |
|---|---|---|---|---|
| **TC-L2-01** | **Stock Available & Payment Succeeds** | Qty: 2, Total: \$500, Regular, Standard | Stock confirmed -> DMN Standard -> Payment SUCCESS -> Fulfilled | Verified |
| **TC-L2-02** | **Stock Insufficient** | Product: `"PROD-OUT-OF-STOCK"`, Qty: 10 | Stock check returns `false` -> Cancelled at Gateway -> End | Verified |
| **TC-L2-03** | **Payment Declined** | Token: `"tok_declined"`, Total: \$250 | Payment returns `DECLINED` -> Cancelled at Payment Gateway -> End | Verified |
| **TC-L2-04** | **High-Value Order Routed to Manual Review** | Qty: 2, Price: \$750 (Total: \$1500) | DMN triggers `requiresManualVerification=true` -> Tasklist user task -> Auditor approves -> Payment & Fulfilled | Verified |
| **TC-L2-05** | **Inventory Worker Incident & Recovery** | Simulated Worker Exception (`simulateFailure=true`) | Incident captured in Operate -> Service bug fixed -> Job retried in Operate -> Execution resumes smoothly | Verified |
