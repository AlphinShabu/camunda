# Lab 2: E-Commerce Order Processing & Fulfilment (Camunda 8)

Automates an online order from start to finish: check stock, set delivery priority, verify high-value orders, take payment, then ship.

## How It Works

1. **Validate & check stock** – `validate-inventory-worker` checks the order, confirms stock, and calculates the total (`unitPrice × quantity`).
2. **Set priority** – a DMN table decides delivery priority, shipping lane, and whether a manual check is needed.
3. **Manual verification** – high-value orders (≥ $1000) go to a Tasklist user task for an auditor.
4. **Take payment** – a REST connector calls `POST /api/payment`.
5. **Fulfil order** – `fulfill-order-worker` creates a tracking number, reserves stock, and dispatches the order.

## Tech Used

| Part | Tool |
|---|---|
| Workflow engine | Zeebe 8.x |
| Workers | Spring Boot (`validate-inventory-worker`, `fulfill-order-worker`) |
| Payment API call | REST Connector (`io.camunda:http-json:1`) |
| Decision logic | DMN (`decision_order_priority`) |
| Manual step | Camunda Tasklist |

## Files

- `order_processing_process.bpmn` – the process diagram
- `order_priority.dmn` – the decision table

## Key Variables

| Variable | Meaning | Example |
|---|---|---|
| `orderId` | Order ID | `ORD-9001` |
| `productId` | Product SKU | `PROD-LAPTOP-X1` |
| `quantity` | Units ordered | `2` |
| `customerCategory` | REGULAR, VIP or PLATINUM | `VIP` |
| `deliveryPreference` | STANDARD, EXPRESS or SAME_DAY | `EXPRESS` |
| `paymentToken` | Fake card token | `tok_synthetic_visa_4242` |
| `isStockAvailable` | Is item in stock? | `true` |
| `orderTotal` | Price × quantity | `1500.0` |
| `deliveryPriority` | DMN result | `CRITICAL` |
| `requiresManualVerification` | Needs auditor check? | `true` |
| `paymentStatus` | SUCCESS, DECLINED or FRAUD_ALERT | `SUCCESS` |
| `trackingNumber` | Shipment tracking ID | `TRK-983271-EXP` |

## Decision Rules (Simplified)

| Order Total | Customer | Delivery | Priority | Shipping Lane | Manual Check? |
|---|---|---|---|---|---|
| ≥ 1000 | any | any | EXPRESS | SECURED_COURIER | Yes |
| any | VIP / PLATINUM | EXPRESS / SAME_DAY | CRITICAL | AIR_EXPRESS | No |
| any | REGULAR | EXPRESS | HIGH | PRIORITY_GROUND | No |
| 100–999 | any | STANDARD | STANDARD | STANDARD_GROUND | No |
| anything else | – | – | LOW | ECONOMY_POST | No |

## Payment API Call

**Request**
```json
{ "orderId": "ORD-9001", "customerRef": "CUST-4421", "orderTotal": 1500.0, "paymentToken": "tok_synthetic_visa_4242" }
```

**Response mapping** – the connector saves these from the reply:
`paymentStatus`, `transactionId`, `paymentMessage`

## Test Cases (All Verified ✅)

| ID | Scenario | Result |
|---|---|---|
| TC-L2-01 | In stock, payment OK ($500, Regular, Standard) | Fulfilled |
| TC-L2-02 | Out of stock (`PROD-OUT-OF-STOCK`) | Cancelled |
| TC-L2-03 | Payment declined (`tok_declined`) | Cancelled |
| TC-L2-04 | High-value order ($1500) | Auditor approves, then paid and fulfilled |
| TC-L2-05 | Worker fails (`simulateFailure=true`) | Incident in Operate, fixed, retried, completed |

