# Camunda BPMN 2.0 Process Modeling Repository

[![BPMN 2.0](https://img.shields.io/badge/BPMN-2.0-blue.svg)](https://www.omg.org/spec/BPMN/2.0/)
[![Camunda Modeler](https://img.shields.io/badge/Camunda%20Modeler-5.20+-orange.svg)](https://camunda.com/download/modeler/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

Repository created for **AlphinShabu** ([github.com/AlphinShabu](https://github.com/AlphinShabu)) containing enterprise process workflows, BPMN 2.0 XML diagrams, and architectural documentation created for Camunda Modeler and Camunda Platform.

---

## 📁 Repository Structure

```text
camunda/
├── README.md                           # Main Repository Documentation
├── .gitignore                          # Git Ignore File
└── ex1/                                # Exercise 1: Core BPMN Scenarios
    ├── README.md                       # Comprehensive Exercise 1 Guide
    ├── scenario1_leave_approval.bpmn   # Scenario 1 BPMN 2.0 XML File
    ├── scenario1_README.md             # Scenario 1 Full Specification & Readme
    ├── scenario2_purchase_order.bpmn   # Scenario 2 BPMN 2.0 XML File
    ├── scenario2_README.md             # Scenario 2 Full Specification & Readme
    ├── scenario3_it_service_request.bpmn # Scenario 3 BPMN 2.0 XML File
    ├── scenario3_README.md             # Scenario 3 Full Specification & Readme
    ├── scenario1/                      # Dedicated Scenario 1 Subfolder
    │   ├── README.md
    │   └── scenario1_leave_approval.bpmn
    ├── scenario2/                      # Dedicated Scenario 2 Subfolder
    │   ├── README.md
    │   └── scenario2_purchase_order.bpmn
    └── scenario3/                      # Dedicated Scenario 3 Subfolder
        ├── README.md
        └── scenario3_it_service_request.bpmn
```

---

## 🎯 Scenarios Included in Exercise 1 (`ex1`)

### 1. Scenario 1: Employee Leave Approval Process
- **Description**: An automated leave management process that checks an employee's remaining leave balance prior to managerial review and executes appropriate updates/notifications.
- **Key Elements**: Start Event, Service Task (`Check Leave Balance`), Exclusive Gateways (`Balance Check`, `Manager Decision`), User Task (`Manager Review & Approval`), Service Task (`Update Balance`), Send Tasks (Notifications), and 3 End Events.
- **Details**: [`ex1/scenario1_README.md`](ex1/scenario1_README.md)

### 2. Scenario 2: Online Purchase Order Processing
- **Description**: An end-to-end e-commerce order processing pipeline covering product availability verification, payment processing, fulfillment, and shipment dispatch.
- **Key Elements**: Start Event (`Customer Places Order`), Service Task (`Check Product Availability`), Exclusive Gateways (`Availability Check`, `Payment Check`), Service Tasks (`Process Payment`, `Prepare Shipment`, `Ship Order`), Send Tasks (`Out of Stock`, `Payment Failure`, `Shipping Confirmation`), and 3 End Events.
- **Details**: [`ex1/scenario2_README.md`](ex1/scenario2_README.md)

### 3. Scenario 3: IT Service Request Management
- **Description**: Enterprise IT support workflow handling user incident logging, severity-based technician assignment (Low -> Support Tech, High -> Senior Tech), internal vs. external provider escalation, and status notifications.
- **Key Elements**: Start Event, User Tasks (`Submit Request`, `Investigate`), Service Tasks (`Register Request`, `Check Severity`, `Update Status`), Exclusive Gateways (`Severity Triage`, `Resolution Check`), Merge Gateways, Send Task, and End Event.
- **Details**: [`ex1/scenario3_README.md`](ex1/scenario3_README.md)

---

## 💻 How to Open and Execute with Camunda Modeler

1. Download **Camunda Modeler** from [camunda.com](https://camunda.com/download/modeler/) or run `Camunda Modeler.exe`.
2. Open Camunda Modeler and go to `File -> Open File...`.
3. Select any `.bpmn` diagram from `ex1/`.
4. Deploy the diagram directly to your running **Camunda Platform 7 / Camunda 8 Engine**.

---

## 📤 Pushing to GitHub Profile (`https://github.com/AlphinShabu/camunda`)

To push this repository to your GitHub profile:

```bash
# Navigate to the repo folder
cd camunda

# Initialize git repository
git init -b main

# Add files and commit
git add .
git commit -m "Add Exercise 1 BPMN scenarios and README documentation"

# Link remote repository
git remote add origin https://github.com/AlphinShabu/camunda.git

# Push to GitHub main branch
git push -u origin main
```
