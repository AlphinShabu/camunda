# Lab 3: Hospital Appointment & Patient Triage (Camunda 8)

Automates hospital appointment booking: validate the patient, decide how urgent the case is, find a slot, involve staff when needed, then confirm the booking.

## How It Works

1. **Validate patient** – `validate-patient-worker` checks the patient's identity and registration details.
2. **Triage** – a DMN table classifies the patient as **Routine**, **Priority Review** or **Urgent Assessment**.
3. **Urgent cases** – go straight to a Tasklist user task for immediate emergency response.
4. **Find a slot** – a REST connector calls `POST /api/hospital-slots` to get available appointment times.
5. **Staff review** – priority cases, manual overrides, or "no slots available" go to a Tasklist user task.
6. **Confirm booking** – `confirm-booking-worker` finalizes the appointment and gives a booking reference.

## Tech Used

| Part | Tool |
|---|---|
| Workflow engine | Zeebe 8.x |
| Workers | Spring Boot (`validate-patient-worker`, `confirm-booking-worker`) |
| Slot lookup | REST Connector (`io.camunda:http-json:1`) |
| Decision logic | DMN (`decision_patient_triage`) |
| Manual steps | Camunda Tasklist |

## Files

- `hospital_triage_process.bpmn` – the process diagram
- `patient_triage.dmn` – the decision table

## Key Variables

| Variable | Meaning | Example |
|---|---|---|
| `patientRef` | Patient ID (synthetic) | `PAT-77312` |
| `patientAge` | Patient age | `68` |
| `symptomsSummary` | Symptom keyword | `CHEST_PAIN` |
| `urgencyIndicator` | CRITICAL, MODERATE or LOW | `CRITICAL` |
| `appointmentPreference` | MORNING, AFTERNOON or ANY | `MORNING` |
| `isRegistrationValid` | Patient found in registry? | `true` |
| `triageCategory` | DMN result | `Urgent Assessment` |
| `maxWaitTimeHours` | Max allowed wait (hours) | `0.25` |
| `recommendedDepartment` | Where to send the patient | `EMERGENCY_CARE` |
| `hasAvailableSlot` | Is a slot free? | `true` |
| `staffResolution` | BookSlot, Waitlist or Decline | `BookSlot` |
| `bookingStatus` | CONFIRMED or WAITLISTED | `CONFIRMED` |
| `bookingReference` | Appointment ticket | `APT-2026-9912` |

## Decision Rules (Simplified)

| Urgency | Symptoms / Age | Category | Max Wait | Department |
|---|---|---|---|---|
| CRITICAL / EMERGENCY | any | Urgent Assessment | 15 min | EMERGENCY_CARE |
| MODERATE / HIGH | FEVER_HIGH, DIZZINESS or FRACTURE_SUSPECTED, and age < 5 or > 65 | Priority Review | 2 hrs | URGENT_OUTPATIENT |
| MODERATE | any | Priority Review | 6 hrs | SPECIALIST_CLINIC |
| LOW / ROUTINE | any | Routine | 24 hrs | GENERAL_MEDICINE |
| anything else | – | Routine | 48 hrs | GENERAL_MEDICINE |

## Slot API Call

**Request**
```json
{ "patientRef": "PAT-77312", "department": "GENERAL_MEDICINE", "appointmentPreference": "MORNING" }
```

**Response mapping** – the connector saves these from the reply:
`availableSlots`, `hasAvailableSlot`, `assignedSlot`

## Test Cases (All Verified ✅)

| ID | Scenario | Result |
|---|---|---|
| TC-L3-01 | Low urgency, age 30, morning | Auto-booked and confirmed |
| TC-L3-02 | Moderate, `FEVER_HIGH`, age 72 | Staff review, slot booked |
| TC-L3-03 | Critical, `CHEST_PAIN` | Emergency task in Tasklist, escalated |
| TC-L3-04 | No slots available | Staff review, waitlisted |
| TC-L3-05 | Scheduling API returns HTTP 500 | Incident in Operate, fixed, retried, completed |


