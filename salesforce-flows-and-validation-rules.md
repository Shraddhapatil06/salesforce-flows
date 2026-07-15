# Salesforce Flows & Validation Rules — Complete Guide

A practical, step-by-step reference for Salesforce Flows (types, use cases) and Validation Rules — including how to build each one from scratch.

---

## Table of Contents

1. [What is a Salesforce Flow?](#1-what-is-a-salesforce-flow)
2. [Types of Flows](#2-types-of-flows)
3. [Core Flow Building Blocks](#3-core-flow-building-blocks)
4. [How to Build a Flow — Step by Step](#4-how-to-build-a-flow--step-by-step)
5. [Worked Example: Record-Triggered Flow](#5-worked-example-record-triggered-flow)
6. [Worked Example: Screen Flow](#6-worked-example-screen-flow)
7. [What is a Validation Rule?](#7-what-is-a-validation-rule)
8. [How to Create a Validation Rule — Step by Step](#8-how-to-create-a-validation-rule--step-by-step)
9. [Common Validation Rule Formulas](#9-common-validation-rule-formulas)
10. [Flow vs Validation Rule — When to Use What](#10-flow-vs-validation-rule--when-to-use-what)
11. [Best Practices](#11-best-practices)
12. [Troubleshooting & Debugging](#12-troubleshooting--debugging)

---

## 1. What is a Salesforce Flow?

Salesforce **Flow** is a declarative (point-and-click) automation tool within Flow Builder that lets you automate business processes without writing Apex code. Flows can:

- Collect information from users via screens
- Create, update, delete, or look up records
- Call Apex classes, external services (via HTTP callouts / named credentials), and other Flows (subflows)
- Send emails, post to Slack/Chatter, or trigger platform events
- Run automatically based on record changes, a schedule, or a user/system action

Flow is the modern replacement for **Workflow Rules** and **Process Builder**, both of which Salesforce has officially retired in favor of Flow (Salesforce recommends migrating all Workflow Rules/Process Builder logic to Flow).

---

## 2. Types of Flows

| Flow Type | Trigger | Runs In | Typical Use Case |
|---|---|---|---|
| **Screen Flow** | User clicks a button, link, tab, Lightning page, or Experience Cloud page | Real-time, with UI | Guided data entry wizards, case creation forms, lead qualification screens |
| **Record-Triggered Flow** | A record is created, updated, or deleted | Background (no UI) | Auto-populate fields, send notifications, update related records |
| **Schedule-Triggered Flow** | Runs at a specific date/time, once or on a recurring batch schedule | Background | Nightly cleanup jobs, reminder emails, contract renewal checks |
| **Platform Event-Triggered Flow** | A platform event message is received | Background | Real-time integration handling, decoupled system-to-system automation |
| **Autolaunched Flow (No Trigger)** | Called by Apex, another Flow, a Quick Action, REST API, or Process Automation | Background or on-demand | Reusable logic (subflows), API-triggered automation |
| **Orchestration Flow** | Coordinates multi-step, multi-user business processes (often with approval-like stages) | Background, sequenced | Multi-stage onboarding, cross-team approval chains |

### 2.1 Record-Triggered Flow — Sub-options

When you create a Record-Triggered Flow, you choose:

- **Trigger:** A record is created / A record is updated / A record is created or updated / A record is deleted
- **Timing:**
  - **Before Save** — fastest; updates the triggering record itself without a second DML/save operation (no automatic re-save needed)
  - **After Save** — required when you need to update *related* records, send emails, call Apex, or perform actions that need the record to already be committed
- **Optimize for:** "Fast Field Updates" (before-save) vs "Actions and Related Records" (after-save)

### 2.2 Screen Flow — Where it can run

- Custom button/link on a record page
- Lightning App Builder (Home, Record, or App page)
- Utility bar
- Quick Action
- Experience Cloud (Community) page
- Embedded in Slack (via Slack integration)

---

## 3. Core Flow Building Blocks

| Element | Purpose |
|---|---|
| **Screens** | Display info / collect input from users (Screen Flows only) |
| **Get Records** | Query records matching filter criteria |
| **Create Records** | Insert new records |
| **Update Records** | Modify existing records |
| **Delete Records** | Remove records |
| **Decision** | Branch logic (if/else, switch-style paths) |
| **Loop** | Iterate over a collection of records/variables |
| **Assignment** | Set/change the value of a variable |
| **Subflow** | Call another flow (reusability) |
| **Action** | Call Apex, Send Email, Post to Chatter, HTTP Callout, etc. |
| **Variables / Collections / Formulas** | Store and calculate data used across the flow |

---

## 4. How to Build a Flow — Step by Step

### Step 1: Open Flow Builder
1. Click the **Gear icon** → **Setup**.
2. In Quick Find, type **Flows** → click **Flows**.
3. Click **New Flow**.

### Step 2: Choose the Flow Type
1. Select the flow type (Screen, Record-Triggered, Schedule-Triggered, Platform Event, Autolaunched, or Orchestration).
2. Click **Create**.

### Step 3: Configure the Trigger (for triggered flow types)
1. Choose the **Object** (e.g., Opportunity, Case, Custom_Object__c).
2. Choose the **trigger condition**: record created / updated / deleted.
3. Set **Entry Conditions** (field-level filters that decide when the flow should run — e.g., `Stage__c = Closed Won`).
4. Choose **Before Save** or **After Save** timing.
5. Optionally check **"Run Asynchronously"** for actions that don't need to block the save.

### Step 4: Build the Canvas
1. Click the **`+`** node on the canvas to add elements (Get Records, Decision, Assignment, Update Records, etc.).
2. Drag elements in the order your business logic requires.
3. For each **Get Records** element:
   - Choose the object.
   - Set filter conditions.
   - Choose "store first record only" vs "store all records" (collection).
4. For each **Decision** element:
   - Add outcome branches with logical conditions (AND/OR/custom formula).
   - Order matters — the first matching outcome wins.
5. For **Assignment** elements, set variable values step by step.
6. For **Create/Update/Delete Records**, map fields explicitly or pass a record variable.

### Step 5: Add Screens (Screen Flows only)
1. Drag a **Screen** element onto the canvas.
2. Add input components (Text, Picklist, Lookup, Checkbox, Data Table, etc.) from the left panel.
3. Set component **API names**, labels, required/optional, default values.
4. Add **validation** on individual screen components if needed (e.g., min/max length, regex).
5. Chain multiple screens for a multi-step wizard; use **Decision** elements between screens to branch the wizard path.

### Step 6: Add Resources (Variables, Constants, Formulas)
1. Click **Toolbox** → **New Resource**.
2. Choose resource type: Variable, Constant, Formula, Text Template, Collection Variable, etc.
3. Set the **Data Type** and, for variables, whether it's **Input**, **Output**, or **Input & Output** (needed if the flow is a subflow or called from a Quick Action).

### Step 7: Set Fault Paths (Error Handling)
1. Click any DML or Action element's connector.
2. Add a **Fault Path** to catch errors (e.g., record-locking issues).
3. Route fault paths to a "Send Email to Admin" action or a Screen showing a friendly error message.

### Step 8: Test the Flow
1. Click **Debug** (top-right).
2. For triggered flows, choose a sample record or manually set input values.
3. Review the execution path highlighted in the canvas and check the **Debug Details** panel for variable values at each step.
4. Fix any errors and re-test.

### Step 9: Activate
1. Click **Save** → give the Flow a **Label**, **API Name**, and **Description**.
2. Click **Activate** (top-right) once testing is complete.
3. Only one version of a flow can be active at a time — new activations deactivate the prior active version (older versions stay saved for rollback).

### Step 10: Monitor
1. Setup → **Flows** → open the flow → **View Run History** (for record-triggered/scheduled flows) to see past executions and errors.
2. Use **Paused and Failed Flow Interviews** (Setup) to inspect flows that errored out or are waiting (e.g., on a Pause element).

---

## 5. Worked Example: Record-Triggered Flow

**Goal:** When an Opportunity is marked `Closed Won`, automatically update the related Account's `Customer_Status__c` field to `Active Customer`.

1. **New Flow** → Record-Triggered Flow → Object: `Opportunity`.
2. Trigger: *A record is updated*.
3. Entry Condition: `StageName Equals Closed Won` (set "Only when a record is updated to meet the condition requirements" = **True** so it doesn't re-fire on every edit).
4. Optimize for: **Actions and Related Records** (After Save) — because we're updating a *related* record (Account), not the triggering record itself.
5. Add a **Get Records** element: Object = Account, Filter: `Id Equals {!$Record.AccountId}`, store first record only.
6. Add an **Update Records** element:
   - Choose "Use the IDs and all field values from a record variable" → pick the Account record.
   - Set field: `Customer_Status__c` = `Active Customer`.
7. Add a **Fault Path** off the Update Records element → Send Email action to notify the admin of failures.
8. Debug with a sample Opportunity, verify the Account updates correctly.
9. Save → Activate.

---

## 6. Worked Example: Screen Flow

**Goal:** A guided wizard for Sales Reps to log a new Case directly from the Account record page.

1. **New Flow** → Screen Flow.
2. Add a **Screen** element: "Case Details" — add fields Subject (Text, required), Description (Long Text Area), Priority (Picklist sourced from Case.Priority picklist values).
3. Add a **Decision** element: If Priority = "High" → branch to an extra Screen asking for an Escalation Reason; else → skip directly to record creation.
4. Add a **Create Records** element: Object = Case, map Subject/Description/Priority/AccountId (`{!$Record.Id}` if launched from the Account page) to the screen inputs.
5. Add a final **Screen**: "Success" — confirmation message with a link/formula showing the new Case number.
6. Save → set as a **Screen Flow**, expose it via a **Lightning Page** (Account Record Page) using the Flow component in Lightning App Builder, or via a **Quick Action**.
7. Debug, then Activate.

---

## 7. What is a Validation Rule?

A **Validation Rule** enforces data quality by preventing a record from being saved unless it meets criteria you define. It evaluates a **formula that returns TRUE/FALSE**:

- If the formula evaluates to **TRUE**, the record is **blocked from saving**, and Salesforce shows your custom **error message**.
- If **FALSE**, the record saves normally.

Validation rules run on every save (insert/update) unless scoped otherwise, including from the UI, Data Loader, APIs, and Flows — making them the most reliable last line of defense for data integrity.

---

## 8. How to Create a Validation Rule — Step by Step

1. Go to **Setup** (gear icon) → **Object Manager**.
2. Select the object (e.g., Opportunity, Account, or a custom object).
3. In the left sidebar, click **Validation Rules**.
4. Click **New**.
5. Enter:
   - **Rule Name** (no spaces; e.g., `Close_Date_Cannot_Be_Past`)
   - **Description** (recommended — explain business intent for future admins)
   - **Active** checkbox (checked = enforced)
6. In the **Error Condition Formula** box, write a formula that returns **TRUE when the record is invalid**. Use the **Insert Field** and **Insert Operator** buttons to build it, or the **Advanced Formula Editor** for complex logic.
7. Click **Check Syntax** to validate the formula compiles correctly.
8. Set the **Error Message** — the text users see when the rule blocks a save.
9. Set **Error Location**:
   - **Top of Page** — general banner error
   - **Field** — attaches the error directly under a specific field (better UX; pick the relevant field from the dropdown)
10. Click **Save**.
11. **Test it**: create/edit a record that should trigger the rule and confirm the error appears; then test a record that should pass.

---

## 9. Common Validation Rule Formulas

**Prevent a Close Date in the past (Opportunity):**
```
CloseDate < TODAY()
```

**Require a field when a picklist has a specific value:**
```
AND(
  ISPICKVAL(StageName, "Closed Lost"),
  ISBLANK(Loss_Reason__c)
)
```

**Enforce a minimum Amount for a specific Record Type:**
```
AND(
  RecordType.DeveloperName = "Enterprise_Deal",
  Amount < 10000
)
```

**Prevent editing a field after a record is Approved (Account example):**
```
AND(
  ISCHANGED(Status__c),
  PRIORVALUE(Status__c) = "Approved"
)
```

**Validate email format on a custom field:**
```
AND(
  NOT(ISBLANK(Secondary_Email__c)),
  NOT(REGEX(Secondary_Email__c, "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}"))
)
```

**Restrict rule to new records only (don't re-validate old data on unrelated edits):**
```
AND(
  ISNEW(),
  ISBLANK(Industry)
)
```

---

## 10. Flow vs Validation Rule — When to Use What

| Scenario | Use |
|---|---|
| Block a save outright based on field values | **Validation Rule** |
| Simple, single-object, formula-only logic | **Validation Rule** |
| Need to query/update *other* records | **Flow** |
| Need a UI/wizard for users | **Flow (Screen Flow)** |
| Need to call an external system (API callout) | **Flow** or Apex |
| Need scheduled/batch logic | **Flow (Schedule-Triggered)** |
| Need conditional error messages tied to complex cross-object logic | **Flow with a Screen "Custom Error" / `$Flow.FaultMessage`**, or a Validation Rule referencing a formula/roll-up field |
| Simple auto-population of a field on the same record | **Flow (Before-Save, Record-Triggered)** — faster than After-Save |

> **Rule of thumb:** Validation Rules are for *stopping bad data*. Flows are for *doing something* (updating records, notifying people, integrating systems, guiding users).

---

## 11. Best Practices

- **One Record-Triggered Flow per object per trigger context** where possible — multiple flows on the same object/trigger can be hard to sequence and debug. Use **Trigger Order** settings if you must have several.
- Prefer **Before-Save** flows for same-record field updates — they're significantly faster (no second DML).
- Always add **entry conditions** to avoid flows running (and consuming limits) when nothing relevant changed.
- Use **Fault Paths** on every DML/Action element — silent failures are hard to diagnose in production.
- Use **subflows** to modularize repeated logic instead of copy-pasting elements across flows.
- Name elements descriptively (`Get_Related_Contacts`, not `Get Records 1`) — critical for maintainability.
- Keep validation rule **error messages actionable** — tell the user exactly what to fix, not just that something is wrong.
- Avoid validation rules that conflict with automated Flow updates (a Flow trying to set a field that a Validation Rule then blocks causes silent failures) — coordinate the two.
- Bulkify: Flows should handle **collections** of records, not assume a single record, since they can run in bulk (e.g., Data Loader imports of thousands of rows).
- Document flows with the **Description** field and inline element descriptions — future admins (or you, in 6 months) will thank you.

---

## 12. Troubleshooting & Debugging

| Issue | Where to Look |
|---|---|
| Flow didn't run at all | Check **Entry Conditions**, check if the Flow is **Active**, check **Trigger Order** vs other flows |
| Flow ran but record wasn't updated | Use **Debug** mode, inspect variable values at each step, check Fault Paths |
| "Too many DML statements" / limit errors | Ensure Get/Create/Update elements are outside loops where possible; use bulk-safe patterns |
| Validation rule blocking unexpectedly | Setup → Object Manager → Validation Rules → temporarily deactivate to isolate, or use **Check Syntax** and test with **sample data** |
| Flow error emails to admin | Setup → **Process Automation Settings** → confirm the "Flow Error" email recipient is set correctly |
| Need historical run data | Setup → Flows → select flow → **View Run History** (for triggered/scheduled flows) |
| Stuck/paused interviews | Setup → **Paused and Failed Flow Interviews** |

---

*This guide is intended as an internal reference for Salesforce admins/developers. Screenshots and org-specific examples can be added as this repo evolves.*
