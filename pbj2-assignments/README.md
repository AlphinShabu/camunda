# PBJ-2 Practical Assignment: Camunda 8 End-to-End Workflow Automation
**Author**: Alphin Shabu  
**Submission Repository**: `https://github.com/AlphinShabu/camunda`  
**Target Platform**: Camunda 8 (Zeebe 8.x, Camunda Modeler, Connectors Runtime, Tasklist, Operate)  
**Implementation Modules**: Java 21, Spring Boot 3.2.5, Zeebe Client 8.5.5  

---

## 📑 Project Structure & Deliverables

```
pbj2-assignments/
├── README.md                                    # Master submission report and execution manual
├── common-worker/                               # Unified Spring Boot Worker & Mock REST Services application
│   ├── pom.xml
│   ├── mvnw / mvnw.cmd
│   └── src/main/java/com/camunda/assignment/
│       ├── AssignmentApplication.java           # Spring Boot Bootstrap
│       ├── mockapi/
│       │   └── MockServicesController.java      # Simulated REST APIs (Credit, Payment, Hospital Slots)
│       └── worker/
│           └── AssignmentWorkers.java           # 6 Zeebe Job Workers across Labs 1, 2, and 3
├── lab1-loan-approval/
│   ├── loan_approval_process.bpmn               # Camunda 8 BPMN diagram
│   ├── loan_classification.dmn                  # DMN Decision Table
│   └── README.md                                # Lab 1 Architecture, variable dictionary & test cases
├── lab2-ecommerce-fulfillment/
│   ├── order_processing_process.bpmn            # Camunda 8 BPMN diagram
│   ├── order_priority.dmn                       # DMN Decision Table
│   └── README.md                                # Lab 2 Architecture, variable dictionary & test cases
└── lab3-hospital-triage/
    ├── hospital_triage_process.bpmn             # Camunda 8 BPMN diagram
    ├── patient_triage.dmn                       # DMN Decision Table
    └── README.md                                # Lab 3 Architecture, variable dictionary & test cases
```

---

## 🚀 Quick Start Instructions

### 1. Start Camunda 8 Run (Local Engine & Webapps)
In the root directory, start the bundled Camunda 8 distribution:
```powershell
.\start-camunda.bat
```
Default Webapp URLs:
- **Operate**: `http://localhost:8081/operate` (Default credentials: `demo` / `demo`)
- **Tasklist**: `http://localhost:8082/tasklist` (Default credentials: `demo` / `demo`)
- **Zeebe Gateway**: `127.0.0.1:26500`
- **Connectors Runtime**: `http://localhost:8086/`

### 2. Deploy BPMN and DMN Models
Open **Camunda Modeler** (`Camunda Modeler.exe`), open each pair of models, and click **Deploy current diagram** to your local cluster (`http://localhost:26500` with authentication `None`):
1. **Lab 1**: `loan_classification.dmn` then `loan_approval_process.bpmn`
2. **Lab 2**: `order_priority.dmn` then `order_processing_process.bpmn`
3. **Lab 3**: `patient_triage.dmn` then `hospital_triage_process.bpmn`

### 3. Start the Spring Boot Worker & Mock API Service
Open a terminal in `pbj2-assignments/common-worker/` and run:
```powershell
.\mvnw.cmd spring-boot:run
```
The application will start on port `8080`, exposing all mock REST endpoints and connecting the 6 Zeebe workers to `127.0.0.1:26500`.

---

## 🧪 Minimum Test Scenarios Summary

### Lab 1: End-to-End Loan Approval Automation
1. **TC-L1-01 (Approval Met)**: `{"applicantId": "APP-101", "applicantAge": 30, "monthlyIncome": 7000.0, "requestedAmount": 30000.0}` → Auto-approved by DMN Rule 1 and auto-disbursed.
2. **TC-L1-02 (Fails Eligibility)**: `{"applicantId": "APP-102", "applicantAge": 22, "monthlyIncome": 1800.0, "requestedAmount": 20000.0}` → Auto-rejected by DMN Rule 5 (Low Income).
3. **TC-L1-03 (Manual Review)**: `{"applicantId": "APP-103", "applicantAge": 38, "monthlyIncome": 8500.0, "requestedAmount": 80000.0}` → DMN Rule 2 triggers Manual Review → Officer approves in Tasklist.
4. **TC-L1-04 (Credit API Error)**: `{"applicantId": "APP-104", "simulateError": true}` → REST Connector Boundary Error catches `CREDIT_API_ERROR` → Falls back to Tasklist Loan Review.
5. **TC-L1-05 (Worker Failure & Incident Recovery)**: `{"applicantId": "APP-105", "simulateWorkerFailure": true}` → Worker throws runtime exception → Incident logged in Operate → Set variable `simulateWorkerFailure=false` and retry job in Operate.

### Lab 2: End-to-End E-Commerce Order Processing and Fulfilment
1. **TC-L2-01 (Stock Available & Payment Succeeds)**: `{"orderId": "ORD-201", "productId": "PROD-STANDARD", "quantity": 2, "customerCategory": "REGULAR", "deliveryPreference": "STANDARD", "paymentToken": "tok_visa"}` → Stock confirmed → DMN standard routing → Payment SUCCESS → Fulfilled.
2. **TC-L2-02 (Stock Insufficient)**: `{"orderId": "ORD-202", "productId": "PROD-OUT-OF-STOCK", "quantity": 5}` → Stock check returns `false` → Order cancelled.
3. **TC-L2-03 (Payment Declined)**: `{"orderId": "ORD-203", "productId": "PROD-STANDARD", "quantity": 1, "paymentToken": "tok_declined"}` → Mock payment declines transaction → Order terminated.
4. **TC-L2-04 (High-Value Order Manual Verification)**: `{"orderId": "ORD-204", "productId": "PROD-LAPTOP-PRO", "quantity": 2}` (Total = $1500) → DMN Rule 1 triggers `requiresManualVerification=true` → Auditor verifies in Tasklist → Payment & Fulfillment.
5. **TC-L2-05 (Inventory Incident & Recovery)**: `{"orderId": "ORD-205", "simulateWorkerFailure": true}` → Incident raised in Operate → Fix issue & retry failed job.

### Lab 3: End-to-End Hospital Appointment and Patient Triage Management
1. **TC-L3-01 (Routine Request with Available Slot)**: `{"patientRef": "PAT-301", "patientAge": 28, "urgencyIndicator": "ROUTINE", "appointmentPreference": "MORNING"}` → DMN Routine → REST API finds slot → Booked & Confirmed.
2. **TC-L3-02 (Priority Review)**: `{"patientRef": "PAT-302", "patientAge": 70, "urgencyIndicator": "MODERATE", "symptomsSummary": "FEVER_HIGH"}` → DMN Priority Review → Staff Review in Tasklist → Staff books slot.
3. **TC-L3-03 (Urgent Assessment)**: `{"patientRef": "PAT-303", "urgencyIndicator": "CRITICAL", "symptomsSummary": "CHEST_PAIN"}` → Emergency human assessment in Tasklist → Escalated immediately.
4. **TC-L3-04 (No Appointment Slots)**: `{"patientRef": "PAT-304", "urgencyIndicator": "LOW", "appointmentPreference": "NONE_AVAILABLE"}` → REST API returns 0 slots → Tasklist Staff Review → Patient waitlisted.
5. **TC-L3-05 (Scheduling API Incident & Recovery)**: `{"patientRef": "PAT-305", "simulateError": true}` → REST Connector incident in Operate → Retry in Operate after backend restore.

---

## 📋 Common Submission Checklist Compliance

- [x] **BPMN model files and process design**: Included for all 3 labs (`.bpmn` files with complete DI layout coordinates).
- [x] **DMN model files, rule descriptions, and sample results**: Complete DMN 1.3 XML tables with FIRST/UNIQUE hit policies.
- [x] **External job worker source code and instructions**: Full Spring Boot worker source code, Maven wrapper, and execution guide.
- [x] **REST Connector configuration and API examples**: `io.camunda:http-json:1` configured with input/output FEEL mappings and mock Spring Boot endpoints.
- [x] **Process-variable dictionary**: Exhaustive dictionary for every variable name, data type, purpose, and producer.
- [x] **Test cases covering normal, rejected, manual-review, and incident paths**: Formatted test matrices provided in each lab's documentation.
- [x] **Operate & Tasklist incident investigation and recovery**: Incident reproduction and resolution documented step-by-step.
- [x] **Concise report describing architecture, execution flow, results, and limitations**: Included in master and lab-level README files.
