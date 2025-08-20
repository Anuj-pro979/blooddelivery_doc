**detailed page lists** + a realistic **JSON `pageState` example** for every page in each application: **User (Hospital/Patient)**, **Delivery Agent**, **Blood Bank**, **Admin**, and **Logistics Admin**.

You can paste these JSON objects into your front-end state, use them as mock API responses for development, or convert them into Postman/OpenAPI mocks. I keep sensitive fields masked (Aadhaar masked, no raw PHI).

---

# User Application (Hospital / Patient / Caregiver)

Notes: different roles share the app; UI adapts by `role`. Pages below include `pageState` JSON examples for what the UI needs.

## Pages

1. Onboarding / Registration (Aadhaar verification)
2. Home / Dashboard (quick actions + active reservations)
3. Create Reservation (form + attach hospital report)
4. Reservation Status / Tracking (order + delivery telemetry)
5. Hospital Reports (upload / history)
6. Donor Search & Volunteering (optional)
7. Notifications & Alerts
8. Profile & Documents
9. Help / Emergency Hotline

---

### Page: Onboarding / Registration

**Purpose:** register user/hospital admin; start Aadhaar OTP flow; upload hospital authorizations.
**APIs:** `POST /api/register/*`, `POST /api/upload/document`, `POST /api/verify/aadhaar`
**pageState (example):**

```json
{
  "page": "onboarding",
  "role": "HOSPITAL_USER",
  "form": {
    "name": "",
    "phone": "",
    "email": "",
    "organization_id": null,
    "aadhaar_masked": "XXXX-XXXX-1234",
    "aadhaar_txn": null,
    "documents": []
  },
  "ui": {
    "aadhaarOtpSent": false,
    "validationErrors": [],
    "nextSteps": [
      "Upload hospital authorization (PDF)",
      "Admin review expected within 24-48 hours"
    ]
  }
}
```

---

### Page: Home / Dashboard

**Purpose:** quick overview: active reservations, matched deliveries, nearby blood banks, alerts.
**APIs:** `GET /api/dashboard/hospital/{id}`, `GET /api/reservations?status=active`
**pageState (example):**

```json
{
  "page": "dashboard",
  "role": "HOSPITAL_USER",
  "hospital": {
    "id": "org-uuid-hosp1",
    "name": "Apollo Hospital - Hyderabad",
    "location": { "lat": 17.3850, "lng": 78.4867 }
  },
  "quickStats": {
    "activeReservations": 2,
    "awaitingDeliveries": 1,
    "nearbyBloodBanks": 3
  },
  "activeReservations": [
    {
      "reservation_id": "res-uuid-1001",
      "blood_group": "O-",
      "component": "PRBC",
      "units_required": 2,
      "status": "PENDING_MATCH",
      "created_at": "2025-08-20T07:10:00Z"
    }
  ],
  "alerts": [
    { "id":"note-1","level":"HIGH","text":"Temperature breach in last 24 hours at Bank: Red Cross (ID: bank-201)"}
  ]
}
```

---

### Page: Create Reservation

**Purpose:** create new blood request; attach hospital report for emergency; show live matching progress.
**APIs:** `POST /api/hospital/report/upload`, `POST /api/reservations`, `GET /api/matching/{reservation_id}`
**pageState (example):**

```json
{
  "page": "create_reservation",
  "form": {
    "hospital_id": "org-uuid-hosp1",
    "requester_user_id": "user-uuid-100",
    "patient_ref": "ABHA-xyz-0001",
    "blood_group": "A+",
    "component": "PRBC",
    "units_required": 2,
    "urgency": "EMERGENCY",
    "hospital_report_id": "rpt-uuid-5001"
  },
  "upload": { "file_token": "file_tok_abc123", "uploadStatus": "DONE" },
  "matching": {
    "status": "NOT_STARTED",
    "candidates": []
  },
  "validation": {
    "requiresReport": true,
    "reportConfirmed": false
  }
}
```

---

### Page: Reservation Status / Tracking

**Purpose:** realtime tracking of matched bank and delivery; telemetry graph (temp/GPS).
**APIs:** `GET /api/reservations/{id}`, `GET /api/delivery/{id}/telemetry`
**pageState (example):**

```json
{
  "page": "reservation_tracking",
  "reservation": {
    "reservation_id": "res-uuid-1001",
    "status": "CONFIRMED",
    "matched_bank": {
      "bank_id": "bank-uuid-201",
      "name": "Red Cross Blood Bank",
      "distance_km": 6.2
    },
    "delivery": {
      "delivery_id": "del-uuid-7001",
      "status": "IN_TRANSIT",
      "eta_drop": "2025-08-20T07:55:00Z",
      "tracking_link": "https://track.example/del-7001"
    }
  },
  "telemetry_preview": [
    {"ts":"2025-08-20T07:20:00Z","temp_c":4.1,"lat":17.4100,"lng":78.4200},
    {"ts":"2025-08-20T07:30:00Z","temp_c":3.9,"lat":17.3950,"lng":78.4350}
  ],
  "actions": ["Cancel Reservation", "Contact Courier", "View Audit Log"]
}
```

---

### Hospital Reports / Uploads

**pageState (example):**

```json
{
  "page": "hospital_reports",
  "hospital_id": "org-uuid-hosp1",
  "reports": [
    {
      "report_id": "rpt-uuid-5001",
      "reservation_id": "res-uuid-1001",
      "file_token": "file_tok_abc123",
      "summary": "Emergency C-section, 2 units PRBC",
      "authorizing_physician": { "name": "Dr. Y", "reg_no": "MED123" },
      "status": "CONFIRMED",
      "uploaded_at": "2025-08-20T07:05:00Z"
    }
  ]
}
```

---

# Delivery Agent Application

Notes: minimal-distraction app focused on jobs, scanning, telemetry, and incident reporting.

## Pages

1. Onboarding / Registration (Aadhaar masked, vehicle)
2. Jobs List (Assigned, Available)
3. Job Detail (pickup/drop points, units list)
4. Pickup (scan units, seal, start telemetry)
5. In-Transit (live telemetry + alerts)
6. Delivery Confirmation / POD (scan hospital, OTP, signature)
7. History & Earnings
8. Profile & Training Certificates
9. Incidents / Emergency Report

---

### Page: Onboarding / Registration (Agent)

**pageState (example):**

```json
{
  "page": "agent_onboarding",
  "agent_profile": {
    "agent_id": "agent-uuid-900",
    "name": "Raju",
    "phone": "+91-9XXXXXXXX",
    "aadhaar_masked": "XXXX-XXXX-2345",
    "vehicle": {"type":"Bike","reg_no":"TS09AB1234"},
    "status": "PENDING_APPROVAL",
    "documents": [
      { "type":"insurance","file_token":"file_ins_001","status":"PENDING" },
      { "type":"training_cert","file_token":"file_tr_001","status":"PENDING" }
    ]
  },
  "nextSteps": ["Aadhaar OTP verification", "Admin approval expected 24–48 hrs"]
}
```

---

### Page: Jobs List

**pageState (example):**

```json
{
  "page": "jobs",
  "agent_id": "agent-uuid-900",
  "assignedJobs": [
    {
      "delivery_id": "del-uuid-7001",
      "reservation_id": "res-uuid-1001",
      "pickup_bank": { "id":"bank-uuid-201","name":"Red Cross", "address":"Secunderabad" },
      "drop_hospital": { "id":"org-uuid-hosp1","name":"Apollo Hospital" },
      "status": "ASSIGNED",
      "eta_pickup": "2025-08-20T07:40:00Z",
      "eta_drop": "2025-08-20T07:55:00Z"
    }
  ],
  "availableJobs": []
}
```

---

### Page: Job Detail / Pickup

**pageState (example):**

```json
{
  "page": "job_detail",
  "delivery_job": {
    "delivery_id": "del-uuid-7001",
    "reservation_id": "res-uuid-1001",
    "pickup_bank": {"id":"bank-uuid-201","name":"Red Cross","address":"Secunderabad"},
    "drop_hospital": {"id":"org-uuid-hosp1","name":"Apollo Hospital"},
    "units": [
      { "blood_unit_id":"unit-uuid-401","isbt128_id":"ISBT-0001","blood_group":"O-","component":"PRBC","expiry_date":"2025-09-30" }
    ],
    "seal_id": "seal-ABC-001",
    "chain_token": "chain-xyz-001",
    "status": "ASSIGNED"
  },
  "pickupActions": {
    "scan_units": true,
    "verify_seal": true,
    "start_telemetry": true
  },
  "safetyPrompts": ["Check temperature logger", "Confirm seal intact"]
}
```

---

### Page: In-Transit (live)

**pageState (example):**

```json
{
  "page": "in_transit",
  "delivery_id": "del-uuid-7001",
  "telemetry": {
    "current_temp_c": 4.0,
    "location": { "lat": 17.3950, "lng": 78.4350 },
    "last_point_ts": "2025-08-20T07:30:00Z"
  },
  "alerts": [],
  "actions": ["Report incident", "Call hospital", "Reroute suggestions"]
}
```

---

### Page: Delivery Confirmation / POD

**pageState (example):**

```json
{
  "page": "delivery_confirm",
  "delivery_id": "del-uuid-7001",
  "scans": {
    "hospital_scan_ok": true,
    "otp": "987654",
    "recipient": { "name":"Ward Nurse", "id":"user-uuid-nurse-1" }
  },
  "temperature_trace_summary": { "min_c":3.8, "max_c":4.4 },
  "actions": ["Complete Delivery","Report Discrepancy"]
}
```

---

# Blood Bank Application

Notes: full operations portal for lab, inventory, stock entry, and dispatch.

## Pages

1. Onboarding / Organization Profile (license upload)
2. Dashboard (stocks, expiries, reservations)
3. Stock Entry (single & bulk CSV import)
4. Picklist & Fulfilment (prepare units for reservations)
5. Lab / Testing (mark tests complete, upload certificates)
6. Dispatch / Handover (seal generation, courier pickup)
7. Inventory Management (search, FEFO lists)
8. Compliance & License (renewal, test audit)
9. Integrations (e-RaktKosh, ABDM, ISBT scanner)
10. Reports & Exports (wastage, issuance, HvPI triggers)
11. Users & Roles (stock operators, lab technicians)

---

### Page: Dashboard

**pageState (example):**

```json
{
  "page": "bank_dashboard",
  "bank": {
    "id": "bank-uuid-201",
    "name": "Red Cross Blood Bank",
    "license_no": "LIC-12345",
    "license_valid_till": "2026-05-31"
  },
  "inventorySummary": {
    "total_units": 420,
    "available": 390,
    "reserved": 20,
    "near_expiry": 12,
    "wastage_30d": 5
  },
  "recentReservations": [
    { "reservation_id":"res-uuid-1001","units":2,"urgency":"EMERGENCY","status":"HELD","requested_at":"2025-08-20T07:10:00Z" }
  ],
  "alerts": [
    {"id":"alert-1","type":"EXPIRY","text":"12 units expire in next 72 hrs"}
  ]
}
```

---

### Page: Stock Entry (bulk CSV)

**pageState (example):**

```json
{
  "page": "stock_entry",
  "uploader": { "user_id":"user-uuid-op1","name":"Stock Operator" },
  "bulkUpload": {
    "file_token": "csv_tok_777",
    "rows_total": 150,
    "rows_success": 145,
    "rows_failed": 5,
    "errors": [
      { "row": 12, "reason": "duplicate ISBT id" },
      { "row": 48, "reason": "expiry date < collection date" }
    ]
  },
  "quickActions": ["Mark testing PASSED", "Auto-assign FEFO for pending reservations"]
}
```

---

### Page: Picklist & Fulfilment

**pageState (example):**

```json
{
  "page": "picklist",
  "reservation_id": "res-uuid-1001",
  "pickList": [
    { "blood_unit_id":"unit-uuid-401","isbt128_id":"ISBT-0001","blood_group":"O-","component":"PRBC","expiry_date":"2025-09-30","testing_status":"PASSED" }
  ],
  "packaging": { "seal_id":"seal-ABC-001","temp_logger_id":"dev-9001" },
  "actions": ["Lock units","Generate dispatch job"]
}
```

---

### Page: Lab / Testing

**pageState (example):**

```json
{
  "page": "lab_testing",
  "pendingTests": [
    { "unit_id":"unit-uuid-501","isbt":"ISBT-00501","required_tests":["HIV","HBsAg","HCV","NAT"], "collected_at":"2025-08-19T09:00:00Z" }
  ],
  "testHistory": [
    { "unit_id":"unit-uuid-401","isbt":"ISBT-0001","status":"PASSED","tested_at":"2025-08-19T11:00:00Z","cert_file":"file_test_cert_01" }
  ],
  "actions": ["Mark as PASSED","Raise QUARANTINE"]
}
```

---

# Admin Application (Super-admin / Regulatory UI)

Notes: very detailed — admin manages registrations, approvals, license enforcement, audits, KPIs, and HvPI submissions.

## Pages

1. Login / Admin Dashboard (global KPIs)
2. User & Actor Management (all users, search, filters)
3. Organization Management (blood banks, hospitals, couriers) — full data + actions
4. Registration Approvals (pending list with document viewer)
5. Inventory Oversight (view/flag banks with high wastage or expired stock)
6. Reservation & Delivery Oversight (live incidents)
7. Haemovigilance (HvPI submissions + follow-ups)
8. Compliance & Licensing (issue/revoke licenses, auto reminders)
9. Audit Log & Forensics (immutable logs, export)
10. System Config & Roles (RBAC, thresholds, SLA targets)
11. Reporting & Exports (CSV/PDF, dashboards)
12. Notifications & Escalations (SMS/Email templates)
13. Integration Monitor (e-RaktKosh / ABDM sync status)
14. Admin Help / SOP Library

---

### Page: Admin Dashboard (pageState)

**pageState (example):**

```json
{
  "page": "admin_dashboard",
  "globalKPIs": {
    "total_banks": 1289,
    "active_banks": 1170,
    "total_reservations_today": 842,
    "avg_booking_to_dispatch_min": 28,
    "wastage_rate_pct_30d": 1.9
  },
  "alerts": [
    { "id":"inc-120","severity":"CRITICAL","text":"Temperature breach >30 mins - Delivery del-uuid-7010" }
  ],
  "pendingApprovals": 48,
  "recentHvpiReports": 6
}
```

---

### Page: User & Actor Management

**Purpose:** search, filter, approve/reject, freeze accounts.
**pageState (example):**

```json
{
  "page": "actor_management",
  "filters": { "status":"PENDING,ACTIVE,SUSPENDED", "role":"BLOOD_BANK,COURIER,HOSPITAL_USER" },
  "results": [
    {
      "id": "bank-uuid-201",
      "type": "BLOOD_BANK",
      "name": "Red Cross Blood Bank",
      "license_no": "LIC-12345",
      "status": "ACTIVE",
      "last_audit": "2025-04-10"
    },
    {
      "id": "agent-uuid-900",
      "type": "COURIER_AGENT",
      "name": "Raju",
      "status": "PENDING_APPROVAL",
      "documents": ["insurance","training_cert"]
    }
  ],
  "actions": ["Approve","Reject","Suspend","RequestMoreInfo"]
}
```

---

### Page: Organization Management (detail view)

**pageState (example):**

```json
{
  "page": "org_detail",
  "organization": {
    "id":"bank-uuid-201",
    "type":"BLOOD_BANK",
    "name":"Red Cross Blood Bank",
    "address":"Secunderabad",
    "license_no":"LIC-12345",
    "license_valid_till":"2026-05-31",
    "contact_person":{"name":"Ms. X","phone":"91-9XXXX"},
    "status":"ACTIVE",
    "integrations":[{"system":"ERAKTKOSH","status":"ACTIVE","last_sync":"2025-08-19T02:00:00Z"}]
  },
  "tabs": {
    "inventory": { "total_units":420,"near_expiry":12 },
    "reservations": [{ "res_id":"res-1001","status":"HELD","units":2 }],
    "audits": [{ "id":"audit-9001","actor":"user-uuid-1","action":"LICENSE_UPLOAD","ts":"2025-05-01T09:00:00Z" }]
  },
  "adminActions": ["RevokeLicense","ForceInventorySync","ScheduleInspection"]
}
```

---

### Page: Compliance & Licensing

**pageState (example):**

```json
{
  "page": "compliance",
  "licensesExpiringSoon": [
    {"org_id":"bank-uuid-201","name":"Red Cross","expires":"2026-05-31","days_left":284}
  ],
  "complianceIncidents": [
    {"id":"ci-501","org_id":"bank-uuid-305","type":"TEMP_BREACH","severity":"HIGH","status":"OPEN"}
  ],
  "actions": ["SendNotice","SuspendUse","EscalateToHealthDept"]
}
```

---

### Page: Audit Log & Forensics

**pageState (example):**

```json
{
  "page": "audit_log",
  "query": { "entity_type":"reservation","from":"2025-08-01","to":"2025-08-20" },
  "results": [
    {"id":1,"actor_user_id":"user-uuid-1","action":"CREATE_RESERVATION","entity_type":"reservation","entity_id":"res-uuid-1001","diff":{},"created_at":"2025-08-20T07:10:05Z"}
  ],
  "exportOptions": ["CSV","JSON","PDF"]
}
```

---

# Logistics Admin (Distribution & Network Management)

Notes: focused on routing, hubs, fleet, telemetry aggregation, and distribution planning.

## Pages

1. Network Map / Hubs View (map with stock heatmap)
2. Distribution Planner (demand-supply rebalancing)
3. Fleet Management (fleet status, drivers, vehicle docs)
4. Route Optimization / Scheduler (auto-schedule pickups)
5. Telemetry Monitor (live temp breaches & geofencing)
6. SLA Dashboard (pickup & delivery SLAs, SLA breaches)
7. Incident Response Center (assign mitigation)
8. Hub / Warehouse Management (cold storage capacities)
9. Integration Monitor (IoT devices / loggers status)

---

### Page: Network Map / Hubs View

**pageState (example):**

```json
{
  "page": "network_map",
  "hubs": [
    {"hub_id":"hub-001","name":"Hyderabad Hub","location":{"lat":17.3850,"lng":78.4867},"capacity":1000,"current_stock":790},
    {"hub_id":"hub-002","name":"Secunderabad Micro","location":{"lat":17.4400,"lng":78.4980},"capacity":200,"current_stock":120}
  ],
  "heatmap": { "metric":"near_expiry_count","timeRange":"72h" },
  "actions": ["Open Hub","Schedule Transfer"]
}
```

---

### Page: Distribution Planner

**Purpose:** rebalance stock from surplus to shortage using FEFO and predicted demand.
**pageState (example):**

```json
{
  "page": "distribution_planner",
  "shortageAlerts": [
    { "location":"Nalgonda","blood_group":"B+","units_needed":6,"priority":"HIGH" }
  ],
  "surplusBanks": [
    { "bank_id":"bank-uuid-501","blood_group":"B+","available_units":12,"distance_km":45 }
  ],
  "recommendations": [
    { "plan_id":"plan-9001","move_from":"bank-uuid-501","move_to":"hub-002","units":10,"mode":"courier","eta_hrs":2.5 }
  ],
  "actions": ["Approve Plan","Simulate Cost","Auto-Dispatch"]
}
```

---

### Page: Fleet Management

**pageState (example):**

```json
{
  "page": "fleet",
  "fleetSummary": { "totalVehicles": 125, "available": 98, "under_maintenance": 7 },
  "vehicles": [
    { "vehicle_id":"veh-9001","type":"Van","reg_no":"TS09AB6789","last_service":"2025-06-10","status":"AVAILABLE","assigned_agent":"agent-uuid-902" }
  ],
  "certValidations": ["cold_chain_cert","insurance","gps_tracker"]
}
```

---

### Page: Telemetry Monitor / Incident Center

**pageState (example):**

```json
{
  "page": "telemetry_monitor",
  "liveDeliveries": [
    { "delivery_id":"del-uuid-7010","temp_c":8.2,"status":"IN_TRANSIT","severity":"BREACH","last_update":"2025-08-20T07:55:10Z" }
  ],
  "incidentQueue": [
    { "incident_id":"inc-120","delivery_id":"del-uuid-7010","type":"TEMP_BREACH","severity":"CRITICAL","assigned_to":"ops-uuid-1","status":"OPEN" }
  ],
  "actions": ["Assign Engineer","Initiate Swap","Notify Admin","Open Ticket"]
}
```

---

## How to use these `pageState` JSONs

* Use as initial mock API responses for development or Storybook stories.
* Each `pageState` contains a compact, realistic dataset — adapt or extend fields to match your API/DB naming.
* Replace UUID placeholders like `res-uuid-1001` with real IDs in your system.
* For telemetry graphs, fetch `telemetry_points` separately and render charts client-side.


* Convert a single app (say Blood Bank UI) into a **React + Tailwind** one-file component (previewable), or
* Generate **OpenAPI** mocks for these page APIs, or
* Produce **Postman collection** with all endpoints and sample JSON responses from the `pageState` objects above.

Which one should I do next?
