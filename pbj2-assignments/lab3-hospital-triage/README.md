# LAB 3: End-to-End Hospital Appointment and Patient Triage Management

## 1. Architecture Overview
This workflow models a patient triage and appointment management system using Camunda 8.
- **Workflow Engine**: Zeebe 8.x
- **External Worker (Spring Boot)**:
  - `validate-patient-worker`: Validates synthetic patient identity and registration fields.
  - `confirm-booking-worker`: Commits the final appointment schedule and outputs booking reference.
- **REST Connector (`io.camunda:http-json:1`)**: Calls simulated hospital scheduling API (`/api/hospital-slots`) to query available doctor/clinic appointment slots.
- **DMN Decision Table (`decision_patient_triage`)**: Applies medical triage rules evaluating urgency indicators, symptoms summary, and age to classify patient into `Routine`, `Priority Review`, or `Urgent Assessment`.
- **Human Review**:
  - `Urgent Assessment`: Immediate Tasklist user task for emergency clinical response.
  - `Authorized Staff Review`: Tasklist user task invoked when a request requires manual override or when no appointment slots are available.

---

## 2. BPMN & DMN Artifacts
- **BPMN Model**: [`hospital_triage_process.bpmn`](hospital_triage_process.bpmn)
- **DMN Model**: [`patient_triage.dmn`](patient_triage.dmn)

---

## 3. Process Variables Dictionary

| Variable Name | Type | Direction | Purpose | Example |
|---|---|---|---|---|
| `patientRef` | String | Input | Synthetic patient ID | `"PAT-77312"` |
| `patientName` | String | Input | Full name of patient | `"Eleanor Vance"` |
| `patientAge` | Integer | Input | Age of the patient | `68` |
| `symptomsSummary` | String | Input | Clinical symptoms keyword | `"CHEST_PAIN"` |
| `urgencyIndicator` | String | Input | Triage indicator (`CRITICAL`, `MODERATE`, `LOW`) | `"CRITICAL"` |
| `appointmentPreference`| String | Input | Preferred time window (`MORNING`, `AFTERNOON`, `ANY`) | `"MORNING"` |
| `isRegistrationValid` | Boolean | Output (Worker) | Patient registry check result | `true` |
| `triageCategory` | String | Output (DMN) | `Routine`, `Priority Review`, or `Urgent Assessment` | `"Urgent Assessment"` |
| `maxWaitTimeHours` | Double | Output (DMN) | SLA maximum allowable wait time in hours | `0.25` |
| `recommendedDepartment`| String | Output (DMN) | Destination medical clinic | `"EMERGENCY_CARE"` |
| `hasAvailableSlot` | Boolean | Output (REST API) | Availability of slot matching criteria | `true` |
| `availableSlots` | List | Output (REST API) | Array of available appointment timeslots | `["10:00 AM", "02:30 PM"]` |
| `assignedSlot` | String | Output (REST API) | Assigned timeslot string | `"2026-11-05 10:00 AM"` |
| `staffResolution` | String | User Task Input | Staff decision (`BookSlot`, `Waitlist`, `Decline`) | `"BookSlot"` |
| `bookingStatus` | String | Output (Worker) | Final booking status (`CONFIRMED`, `WAITLISTED`) | `"CONFIRMED"` |
| `bookingReference` | String | Output (Worker) | Unique clinic appointment ticket | `"APT-2026-9912"` |

---

## 4. DMN Decision Table Evaluation Logic

| Rule | Urgency Indicator | Symptoms Summary | Patient Age | Triage Category | Max Wait Time | Recommended Dept |
|---|---|---|---|---|---|---|
| **1** | `"CRITICAL", "EMERGENCY"` | `-` | `-` | `"Urgent Assessment"` | `0.25 hrs` (15 min) | `"EMERGENCY_CARE"` |
| **2** | `"MODERATE", "HIGH"` | `"FEVER_HIGH", "DIZZINESS", "FRACTURE_SUSPECTED"` | `< 5, > 65` | `"Priority Review"` | `2.0 hrs` | `"URGENT_OUTPATIENT"` |
| **3** | `"MODERATE"` | `-` | `-` | `"Priority Review"` | `6.0 hrs` | `"SPECIALIST_CLINIC"` |
| **4** | `"LOW", "ROUTINE"` | `-` | `-` | `"Routine"` | `24.0 hrs` | `"GENERAL_MEDICINE"` |
| **5** | `-` | `-` | `-` (Fallback) | `"Routine"` | `48.0 hrs` | `"GENERAL_MEDICINE"` |

---

## 5. REST Connector Specification
- **Task Type**: `io.camunda:http-json:1`
- **Method**: `POST`
- **URL**: `http://localhost:8080/api/hospital-slots`
- **Request Payload**:
  ```json
  {
    "patientRef": "PAT-77312",
    "department": "GENERAL_MEDICINE",
    "appointmentPreference": "MORNING"
  }
  ```
- **Response Mapping (`resultExpression`)**:
  ```feel
  ={availableSlots: response.body.availableSlots, hasAvailableSlot: response.body.hasAvailableSlot, assignedSlot: response.body.assignedSlot}
  ```

---

## 6. Test-Case Matrix (Minimum Test Scenarios)

| Case ID | Scenario Description | Inputs | Expected Route & Outcome | Actual Status |
|---|---|---|---|---|
| **TC-L3-01** | **Routine Request with Available Slot** | Urgency: `"LOW"`, Age: 30, Preference: `"MORNING"` | DMN Routine -> REST API confirms slot -> Auto-booked -> Confirmed | Verified |
| **TC-L3-02** | **Priority Staff Review** | Urgency: `"MODERATE"`, Symptoms: `"FEVER_HIGH"`, Age: 72 | DMN Priority Review -> Tasklist Staff Review -> Staff books slot -> Booked | Verified |
| **TC-L3-03** | **Urgent Assessment Category** | Urgency: `"CRITICAL"`, Symptoms: `"CHEST_PAIN"` | Immediate human emergency assessment in Tasklist -> Escalated | Verified |
| **TC-L3-04** | **No Appointment Slots Available** | Preference: `"NONE_AVAILABLE"` (API returns `hasAvailableSlot=false`) | REST API returns 0 slots -> Routes to Authorized Staff Review -> Waitlisted | Verified |
| **TC-L3-05** | **Scheduling API Incident & Recovery** | Simulated HTTP 500 error on Scheduling API | Incident captured in Operate -> Service corrected -> Job retried -> Flow completes successfully | Verified |
