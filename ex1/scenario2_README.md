# Scenario 2: Online Purchase Order Processing

## 📋 Overview
This BPMN 2.0 process models an **E-Commerce Online Purchase Order Workflow**. The system validates item inventory, processes customer payment securely, handles failure branches (out of stock, payment declined), prepares order fulfillment, ships items, and notifies the buyer.

---

## ⚙️ Process Flow & Step-by-Step Requirements

1. **Start Event**: A customer places an order via the online storefront (`StartEvent_OrderPlaced`).
2. **Product Availability Check**: The order service checks stock availability in warehouse inventory (`Task_CheckAvailability`).
3. **Exclusive Gateway - Inventory Check (`Gateway_AvailabilityCheck`)**:
   - **Path A (Out of Stock)**: If the item is unavailable, an out-of-stock notification is dispatched to the customer (`Task_SendOutOfStockNotif`), and the process terminates at `EndEvent_OutOfStock`.
   - **Path B (Available)**: If product is available, the system initiates payment processing (`Task_ProcessPayment`).
4. **Exclusive Gateway - Payment Check (`Gateway_PaymentCheck`)**:
   - **Path B1 (Payment Failed)**: If credit card/gateway payment fails, a failure notification is sent to the customer (`Task_SendPaymentFailedNotif`), ending at `EndEvent_PaymentFailed`.
   - **Path B2 (Payment Successful)**: If payment succeeds:
     - The order is confirmed and sent to fulfillment/warehouse (`Task_PrepareShipment`).
     - The warehouse packs and ships the order (`Task_ShipOrder`).
     - A shipping tracking confirmation email is sent (`Task_SendShippingConf`).
     - The process completes successfully at `EndEvent_OrderCompleted`.

---

## 🛠️ BPMN Elements Mapping

| BPMN Element Type | Element ID | Label / Name | Type Details |
| :--- | :--- | :--- | :--- |
| **Start Event** | `StartEvent_OrderPlaced` | Customer Places Order | None (Start Event) |
| **Service Task** | `Task_CheckAvailability` | Check Product Availability | Camunda External Service Task |
| **Exclusive Gateway** | `Gateway_AvailabilityCheck` | Is Product Available? | XOR Gateway |
| **Send Task** | `Task_SendOutOfStockNotif` | Notify Customer (Out of Stock) | Camunda External Send Task |
| **End Event** | `EndEvent_OutOfStock` | Order Cancelled (Out of Stock) | None End Event |
| **Service Task** | `Task_ProcessPayment` | Process Payment | Camunda External Service Task |
| **Exclusive Gateway** | `Gateway_PaymentCheck` | Is Payment Successful? | XOR Gateway |
| **Send Task** | `Task_SendPaymentFailedNotif` | Notify Customer (Payment Failure) | Camunda External Send Task |
| **End Event** | `EndEvent_PaymentFailed` | Order Cancelled (Payment Failed) | None End Event |
| **Service Task** | `Task_PrepareShipment` | Confirm Order & Prepare Product for Shipment | Camunda External Service Task |
| **Service Task** | `Task_ShipOrder` | Ship Order | Camunda External Service Task |
| **Send Task** | `Task_SendShippingConf` | Send Shipping Confirmation | Camunda External Send Task |
| **End Event** | `EndEvent_OrderCompleted` | Order Completed & Shipped | None End Event |

---

## 📐 Visual Process Diagram Architecture

```
[Start: Order Placed] ──> [Check Product Availability] ──> <Is Product Available?>
                                                               │
                                         ┌─────────────────────┴─────────────────────┐
                                  (No / Out of Stock)                         (Yes / Available)
                                         │                                           │
                                         ▼                                           ▼
                           [Notify Customer (Out of Stock)]                   [Process Payment]
                                         │                                           │
                                         ▼                                           ▼
                             (End: Order Cancelled)                       <Is Payment Successful?>
                                                                                     │
                                                             ┌───────────────────────┴───────────────────────┐
                                                        (No / Failed)                                  (Yes / Success)
                                                             │                                               │
                                                             ▼                                               ▼
                                            [Notify Customer (Payment Failure)]             [Confirm Order & Prepare Shipment]
                                                             │                                               │
                                                             ▼                                               ▼
                                                 (End: Order Cancelled)                                 [Ship Order]
                                                                                                             │
                                                                                                             ▼
                                                                                                [Send Shipping Confirmation]
                                                                                                             │
                                                                                                             ▼
                                                                                                  (End: Order Completed)
```

---

## 🚀 How to Import & Run in Camunda Modeler
1. Launch **Camunda Modeler**.
2. Open `scenario2_purchase_order.bpmn`.
3. Test execution paths in Camunda Engine or Camunda Desktop Application.
