# Take-Home Test — Workshop Operations (Vehicle Management System)

Welcome! This take-home exercise reflects the kind of work you’ll do on our Vehicle Management System for a multinational fitment centre. We care most about clean engineering, sensible tradeoffs, and a solution that’s easy to understand and extend.

---

## 1) Timebox (Important)

- Target: **6–10 hours total**
- Deadline: **3–5 days** from receiving the repo
- Please **stop at ~10 hours**, even if you haven’t finished everything. We value prioritization and clarity over “finishing at all costs”.

If you hit the timebox, stop and document what you would do next (see section 12).

---

## 2) Tech Options (Pick One)

### Option A (Preferred): Next.js Full-Stack
- One **Next.js** app
- API via **Route Handlers** (`/app/api/...`) or `/pages/api/...`
- UI built in Next.js (client components where needed)

### Option B: Split FE + API
- **Next.js** front end
- **Node.js + TypeScript + Express** REST API

✅ Either option is acceptable. Choose what helps you deliver the best quality within the timebox.

---

## 3) What We’re Evaluating

We will assess:
- Domain modelling & workflow thinking (workshop ops)
- API design & consistency (resources, status codes, errors)
- TypeScript quality (types at boundaries, structure)
- Validation & security basics (tenant isolation, input validation)
- Testing (key flows + edge cases)
- Pragmatism (tradeoffs, maintainability)
- Developer experience (setup, scripts, docs)

---

## 4) Core Path (Designed to Fit 6–10 Hours)

This is the **required scope**. Anything beyond this is **bonus**.

You will build a small internal portal for workshop staff to:
- view the **work queue** of work orders
- open a work order
- complete a checklist
- move the work order through a workflow (with rules enforced)

---

## 5) Minimum Acceptance Checklist (Required)

Your submission is considered complete if it includes:

### Backend (Required)
- ✅ Workflow rules enforced (no skipping statuses)
- ✅ Checklist gating enforced (cannot move to QA/Complete until checklist is done)
- ✅ Runtime validation
- ✅ Consistent error response format
- ✅ Automated tests covering:
  - invalid status transition rejected
  - checklist gating


### Frontend (Required)
- ✅ Work Queue screen (list + basic filters)
- ✅ Work Order Detail screen (checklist + status updates)
- ✅ Loading/error/empty states handled

### Docs (Required)
- ✅ `README.md` with setup instructions, key decisions, and next steps

---

## 6) Domain Overview

A fitment centre needs a simple internal portal to manage workshop operations.

### Core Entities (Suggested)
Keep this minimal—do not overbuild. You may adjust as long as you justify changes in the README.

#### Vehicle (minimal supporting entity)
- `id`
- `tenantId`
- `registrationNumber` (required)
- `vin` (optional)
- `make`, `model`, `year` (optional)

#### WorkOrder (primary entity)
- `id`
- `tenantId`
- `workshopId` (optional, but recommended)
- `vehicleId` (or embed a small vehicle summary)
- `serviceType` (TYRES, BATTERY, ALIGNMENT, BRAKES, OTHER)
- `priority` (LOW, MEDIUM, HIGH)
- `status` (see section 7)
- `scheduledAt` (optional)
- `notes` (optional)
- `checklist: ChecklistItem[]`

#### ChecklistItem
- `id`
- `label`
- `completed` (boolean)
- `completedAt` (optional)

---

## 7) Workflow Rules (Required)

### Status values
Use these (or a very close equivalent):
- `CREATED`
- `IN_PROGRESS`
- `READY_FOR_QA`
- `COMPLETED`

### Allowed transitions
- `CREATED → IN_PROGRESS → READY_FOR_QA → COMPLETED`
- You **cannot skip** statuses (e.g., `CREATED → READY_FOR_QA` is invalid)

### Checklist gating rule
- You **cannot** transition to `READY_FOR_QA` or `COMPLETED` unless **all checklist items are completed**

---

## 8) Multi-Tenancy Requirement (Required)

Your system must enforce tenant isolation server-side.

Rules:
- All records must be scoped to a tenant (`tenantId`)
- If a resource id exists but belongs to a different tenant, return an appropriate error (e.g., `404` or `403`, document your choice)

---

## 9) Persistence Requirement (Required)

To keep the exercise lightweight, a real DB is **not required**.

### Required
Implement a repository layer with an **in-memory** store (e.g., Map-based) that supports:
- create/read/update/list
- tenant isolation
- basic filtering for work queue

### Optional (Bonus)
Add MongoDB/Mongoose support behind the same repository interface.

---

## 10) API Requirements (Required)

### General
- JSON request/response
- Runtime validation for inputs
- Consistent error response format

### Error response format (example)
You may design your own, but it must be consistent.

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid request",
    "details": [
      { "field": "status", "message": "Invalid status transition" }
    ]
  }
}
```

---

## 11) Required Endpoints (Core Path)

### Vehicles

1) `POST /api/tenants/:tenantId/vehicles`
- Create a vehicle
- Minimum required fields:
  - `registrationNumber`
- Optional fields:
  - `vin`, `make`, `model`, `year`
- Uniqueness rules are optional; document your approach

2) `GET /api/tenants/:tenantId/vehicles?search=...`
- Search vehicles by `search` query
- Match against:
  - `registrationNumber` (required)
  - optionally `vin`, `make`, `model`

### Work Orders

3) `POST /api/tenants/:tenantId/work-orders`
- Create a new work order for an existing vehicle
- Minimum required fields:
  - `vehicleId`
  - `serviceType`
  - `priority`
- Should initialize a **default checklist** (at least 3–6 items)

4) `GET /api/tenants/:tenantId/work-orders`
- List work orders (work queue)
- Filters (minimum):
  - `status` (optional)
  - `search` (optional) — match vehicle registration (and/or vin if stored)
  - `branchId` (optional)

5) `GET /api/tenants/:tenantId/work-orders/:id`
- Retrieve a work order by id

6) `PATCH /api/tenants/:tenantId/work-orders/:id`
- Update work order fields **with rules**
- Must support:
  - updating checklist item completion
  - updating status (enforce workflow + checklist gating)

> You may split the PATCH into `/status` and `/checklist` endpoints if you prefer, but keep it simple.

---

## 12) Frontend Requirements (Required)

Build a small usable UI for workshop staff.

### Screen 1: Work Queue
- Display a table/list of work orders
- Filter by status (minimum)
- Show key fields:
  - vehicle registration number
  - service type
  - priority
  - status
- Click a row to open the work order detail
- Handle loading/error/empty states

### Screen 2: Work Order Detail
- Display work order and vehicle details
- Display checklist and allow marking items complete/incomplete
- Provide status transition controls that respect the workflow rules
- Show API validation errors clearly in the UI

---

## 13) Testing Requirements (Required)

Implement automated tests (unit and/or integration) that cover:

1) **Workflow transition validation**
- Reject invalid transitions (e.g., `CREATED → COMPLETED`)
- Allow valid transitions

2) **Checklist gating**
- Cannot move to `READY_FOR_QA` or `COMPLETED` when checklist is incomplete
- Can move once all items are completed

3) **Tenant isolation**
- Create data under tenant A
- Ensure tenant B cannot read/update it

Tooling:
- Any reasonable test setup is fine (Jest/Vitest/etc.)
- Tests must be runnable via `npm test`

---

## 14) Bonus (Optional — Only If You Have Time)

Pick any that fits your timebox:

### Mongo/Mongoose
- Implement Mongoose repositories behind the same interfaces
- Provide schema + recommended indexes (examples):
  - `{ tenantId, "vehicle.registrationNumber" }`
  - `{ tenantId, status }`
  - `{ tenantId, createdAt }`

### Real-world robustness
- Audit log for status transitions (who/when/what)
- Optimistic concurrency (version or `updatedAt` checks)

---

## 15) Scripts (Required)

Ensure these work:
- `npm install`
- `npm run dev`
- `npm test`

If you split FE/API, provide scripts to run both easily (and document them).

---

## 16) README Requirements (Required)

In `README.md`, include:
- How to run the app and tests
- Which tech option you chose (Next-only vs split Express)
- Short data model overview
- How tenant isolation is enforced
- Notes on validation + error strategy
- Tradeoffs you made and why
- What you’d build next with more time

---

## 17) If You Run Out of Time (Expected and OK)

Please don’t rush low-quality code. If you hit the timebox:
- stop coding
- add a short “Next Steps” section to the README:
  - what remains
  - how you would implement it
  - risks/edge cases you would handle

---

## 18) Submission Instructions

1) Create a **private** repo using **Use this template**
2) Implement your solution
3) Confirm required scripts work
4) Email us as per README instructions. 

Please keep your solution **private** until the process ends. Do not share your solution publicly

Good luck, we’re looking forward to reviewing your approach!
