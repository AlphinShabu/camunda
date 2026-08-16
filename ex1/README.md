# Exercise 1: BPMN 2.0 Business Process Modeling Scenarios

Welcome to **Exercise 1 (ex1)** of the **Camunda BPMN 2.0 Repository**. This folder contains executable standard BPMN 2.0 XML process diagrams and comprehensive documentation for three core enterprise business process scenarios modeled for **Camunda Modeler** and **Camunda Platform 7 / Camunda 8**.

---

## 📂 Exercise 1 Directory Structure

```
ex1/
├── README.md                           # Exercise 1 Overview (This document)
├── scenario1_leave_approval.bpmn       # Scenario 1 BPMN 2.0 XML Diagram
├── scenario1_README.md                 # Scenario 1 Documentation & Element Specifications
├── scenario2_purchase_order.bpmn       # Scenario 2 BPMN 2.0 XML Diagram
├── scenario2_README.md                 # Scenario 2 Documentation & Element Specifications
├── scenario3_it_service_request.bpmn   # Scenario 3 BPMN 2.0 XML Diagram
└── scenario3_README.md                 # Scenario 3 Documentation & Element Specifications
```

---

## 📑 Summary of Scenarios

### 1. Scenario 1: Employee Leave Approval (`scenario1_leave_approval.bpmn`)
- **Business Purpose**: Automated verification of leave entitlement followed by managerial review and automated notification dispatch.
- **Key Features**:
  - Balance check prior to manager review.
  - Multi-path XOR branching (Insufficient Balance vs. Manager Approval vs. Manager Rejection).
  - Automated balance deduction on approval.
- **Expected BPMN Elements**: Start Event, Service Tasks, User Task, Send Tasks, Exclusive Gateways, 3 End Events.
- **Documentation**: See [`scenario1_README.md`](scenario1_README.md).

---

### 2. Scenario 2: Online Purchase Order Processing (`scenario2_purchase_order.bpmn`)
- **Business Purpose**: E-commerce customer order fulfillment pipeline with stock checking, payment authorization, and shipment dispatch.
- **Key Features**:
  - Product stock checking with early termination if out-of-stock.
  - Payment processing with payment failure handling.
  - Sequential fulfillment stages (Preparation -> Shipment -> Customer Confirmation).
- **Expected BPMN Elements**: Start Event, Service Tasks, Send Tasks, Exclusive Gateways, 3 End Events, Multiple process paths.
- **Documentation**: See [`scenario2_README.md`](scenario2_README.md).

---

### 3. Scenario 3: IT Service Request Management (`scenario3_it_service_request.bpmn`)
- **Business Purpose**: Internal IT help desk problem reporting, severity-based technician routing, internal vs. vendor escalation, and ticket closure.
- **Key Features**:
  - Dynamic severity assessment (Low Severity -> Support Tech, High Severity -> Senior Tech).
  - Alternative resolution routes (Internal fix vs. External Service Provider escalation).
  - Path merging prior to final status update and notification.
- **Expected BPMN Elements**: Start Event, User Tasks, Service Tasks, Send Tasks, Exclusive Gateways, Alternative Paths, End Event.
- **Documentation**: See [`scenario3_README.md`](scenario3_README.md).

---

## 🛠️ How to View & Deploy BPMN Files in Camunda Modeler

1. Download and install **Camunda Modeler** (or launch `Camunda Modeler.exe`).
2. Click **File -> Open File...** and select any `.bpmn` file inside `ex1/`.
3. Modify or execute the diagrams in Camunda 7 (Engine 7.x) or Camunda 8 (Zeebe).
4. All XML files contain valid BPMN Diagram Interchange (BPMNDI) tags for clean visual layout without node overlaps.
