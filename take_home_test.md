# Take-Home Test — Workshop Operations (Vehicle Management System)

Welcome! This take-home exercise reflects the kind of work you’ll do on our Vehicle Management System for a multinational fitment centre. We care most about clean engineering, sensible tradeoffs, and a solution that’s easy to understand and extend.

---

## The Context 
We are building an internal portal to manage workshop operations for a multinational fitment centre. We care most about clean engineering, sensible trade-offs, and a solution that is easy to understand and extend.

---

## 1) Timebox (Important)

- Target: **6–10 hours total**
- Deadline: **3–5 days** from receiving the repo
- Please **stop at ~10 hours**, even if you haven’t finished everything. We value prioritization and clarity over “finishing at all costs”.

If you hit the timebox, stop and document what you would do next (see section 12).

---

## 2) Functional Requirements 

Build a full-stack application that enables workshop staff to manage daily operations. The system must support the following capabilities:
- **Vehicle Intake**: Users can register a vehicle into the system for a specific tenant.
- **Work Order Management**: Users can create a work order for a vehicle, specifying the service type (e.g., Tyres, Battery, Brakes) and priority.
- **The Work Queue**: Users can view a list of active work orders, filter them by status, and search by vehicle details.
- **Execution & Checklists**: Users can open a specific work order, view details, complete a checklist of tasks, and move the work order through its lifecycle.

---

## 3) Core Business Rules 

Your domain model must strictly enforce the following rules:
- **Multi-Tenancy**: The system will be used by multiple independent workshops. All data (vehicles, work orders) must be strictly isolated by tenantId. A user from Tenant A must never be able to read or mutate data from Tenant B.
- **Workflow Enforcement**: Work orders must follow a strict status lifecycle: Created → In Progress → Ready for QA → Completed. You cannot skip statuses.
- **Checklist Gating**: A work order cannot be moved to Ready for QA or Completed unless all items on its associated checklist are marked as done.

--- 

## 4) Technical Expectations 

You have complete freedom over the technical implementation, API design, and data persistence strategy (in-memory, SQLite, Mongo, etc.), provided it is a TypeScript/JavaScript stack. You can build a monolith (e.g., Next.js) or a split frontend/backend.

We will be evaluating:

- **Domain Modeling**: How you structure the core entities and business logic.
- **API & Boundary Design**: How your front-end communicates with your back-end, including error handling and runtime validation.
- **Automated Testing**: We expect tests covering the critical business rules (workflow transitions, checklist gating, and tenant isolation).
- **Developer Experience**: How easy it is to set up and run your code.

--- 

## 5) Deliverables

- A private repository containing your code.
- A README.md that includes:
  - Setup and run instructions (e.g., npm run dev, npm test).
  - A brief overview of your data model and API design choices.
  - An explanation of how you handled tenant isolation.
  - Trade-offs you made to fit the timebox and what you would build next.

---

## 6) Domain Overview

A fitment centre needs a simple internal portal to manage workshop operations.

### Core Entities (Suggested)
You may adjust as long as you justify changes in the README.

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

## 7) Workflow Rules (Suggested)

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

## 8) Submission Instructions

1) Create a **private** repo using **Use this template**
2) Implement your solution
3) Confirm required scripts work
4) Email us as per README instructions. 

Please keep your solution **private** until the process ends. Do not share your solution publicly

Good luck, we’re looking forward to reviewing your approach!
