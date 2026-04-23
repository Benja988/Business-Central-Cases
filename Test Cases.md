# HB Vastgoed 365 — Full Test Case Suite
## From First Process to Last

**System:** HB Vastgoed 365 on Microsoft Dynamics 365 Business Central
**Publisher:** HB Software
**Version:** 27.0
**Prepared:** 2026-03-30

---

## How to Read This Document

Each test case follows this structure:

| Field | Meaning |
|---|---|
| **TC-ID** | Unique test case identifier |
| **Module** | Which HB module is being tested |
| **Process** | The business process being tested |
| **Type** | Positive (happy path) / Negative (error path) / Edge case |
| **Prerequisites** | What must exist before running the test |
| **Steps** | Numbered steps to execute |
| **Expected Result** | What should happen |
| **BC Objects Involved** | Tables, Codeunits, Pages touched |

---

## TEST AREA INDEX

| # | Area | TC Range |
|---|---|---|
| 1 | Setup & Company Initialization | TC-001 – TC-010 |
| 2 | Real Estate Management (Vastgoedbeheer) — Complex & Contract | TC-011 – TC-025 |
| 3 | Project Management (Projecten) | TC-026 – TC-050 |
| 4 | Calculations (Calculaties) | TC-051 – TC-065 |
| 5 | Budget Management (Budget) | TC-066 – TC-085 |
| 6 | Additional Budget (Aanvullend Budget) | TC-086 – TC-095 |
| 7 | Project Planning (Planning) | TC-096 – TC-110 |
| 8 | Draft Orders (Opdrachten) | TC-111 – TC-125 |
| 9 | Obligations (Verplichtingen) | TC-126 – TC-145 |
| 10 | Purchase to Obligation Full Flow | TC-146 – TC-155 |
| 11 | Approval Workflows | TC-156 – TC-175 |
| 12 | Invoicing Engine (Facturatie) | TC-176 – TC-190 |
| 13 | Loans (Leningen) | TC-191 – TC-205 |
| 14 | Interest (Rente) | TC-206 – TC-215 |
| 15 | Forecasts (Prognoses) | TC-216 – TC-230 |
| 16 | Cash Flow (CashFlow) | TC-231 – TC-240 |
| 17 | General Costs & Provisions (AK-Voorziening) | TC-241 – TC-255 |
| 18 | Buyer Administration (Kopersadministratie) | TC-256 – TC-275 |
| 19 | More-Less Work (MMW) | TC-276 – TC-285 |
| 20 | Time Registration (Urenregistratie) | TC-286 – TC-295 |
| 21 | Coverage Surveillance (Dekkingtoezicht) | TC-296 – TC-305 |
| 22 | Risk Analysis (Risicoanalyse) | TC-306 – TC-315 |
| 23 | Profit Taking (Winstneming) | TC-316 – TC-330 |
| 24 | Mandatory Fields (Verplichte Velden) | TC-331 – TC-340 |
| 25 | Central Administration (Centraal) | TC-341 – TC-350 |
| 26 | Cloud File Storage | TC-351 – TC-358 |
| 27 | End-to-End Scenarios | TC-359 – TC-375 |

---

---

# AREA 1 — SETUP & COMPANY INITIALIZATION

---

## TC-001
**Module:** Setup
**Process:** HB Vastgoed Setup — Create Setup Record
**Type:** Positive

**Prerequisites:**
- Fresh company created in BC
- HB Vastgoed 365 app installed

**Steps:**
1. Open **HBVG Setup** page
2. Fill in **No. Series** for Projects
3. Fill in **No. Series** for Calculations
4. Fill in **No. Series** for Draft Orders
5. Fill in **No. Series** for Loans
6. Fill in **Default Dimension** for projects (e.g., Dimension Code = PROJECT)
7. Set **Extended Approval** = Enabled
8. Save the record

**Expected Result:**
- Setup record saved without errors
- All No. Series fields populated
- Setup is retrievable in subsequent sessions

**BC Objects:** Table 50110 HBVG Setup, Page HBVG Setup Card

---

## TC-002
**Module:** Setup
**Process:** Company Initialization from Central Company
**Type:** Positive

**Prerequisites:**
- Central (beheer) company exists and is configured
- HBVG Setup exists in central company with No. Series defined
- User has Admin permissions

**Steps:**
1. Open new satellite company
2. Navigate to **HBVG Setup**
3. Run action **"Setup from Central Administration"** (SetupVanuitCentraalBeheer)
4. Confirm the dialog

**Expected Result:**
- No. Series copied from central company to satellite company
- Company prefix (first 3 chars of company name) prefixed to No. Series codes
- **"HBVG Initialize Company"** flag on Company Information set to TRUE
- GL Account admin records created

**BC Objects:** Codeunit 50100 HBVGCompanyInitialize, Table 50110 HBVG Setup

---

## TC-003
**Module:** Setup
**Process:** Company Initialization — Missing Central Company
**Type:** Negative

**Prerequisites:**
- No central company configured

**Steps:**
1. Open HBVG Setup
2. Run **"Setup from Central Administration"**

**Expected Result:**
- Error message displayed: central company not found
- No records created
- Setup remains unchanged

---

## TC-004
**Module:** Setup
**Process:** Security Group Configuration
**Type:** Positive

**Prerequisites:**
- HBVG Setup exists

**Steps:**
1. Open **HBVG Security Groups** page
2. Create new Security Group: Code = "PROJ-MGR", Description = "Project Managers"
3. Add Security Group Options:
   - Enable: "Approve Planning"
   - Enable: "Approve Budget Request"
   - Enable: "Approve Prognose"
4. Save

**Expected Result:**
- Security Group created with correct options
- Group is available for assignment to users in HBVG User Setup

**BC Objects:** Table 50118 HBVG Security Group, Table 50120 HBVG Security Group Option

---

## TC-005
**Module:** Setup
**Process:** User Setup — Assign Security Group
**Type:** Positive

**Prerequisites:**
- Security Group "PROJ-MGR" created (TC-004)
- BC User exists

**Steps:**
1. Open **HBVG User Setup**
2. Select an existing BC user
3. Assign Security Group = "PROJ-MGR"
4. Save

**Expected Result:**
- User linked to security group
- User inherits approval authority flags from the group

**BC Objects:** PageExt HBVG UserSetup (extends User Setup)

---

## TC-006
**Module:** Setup
**Process:** Mandatory Fields Setup
**Type:** Positive

**Prerequisites:**
- HBVG Setup exists

**Steps:**
1. Open **HBVG Mandatory Fields** setup
2. Add rule: Table = "Purchase Header", Field = "HBVG Project No.", Mandatory = TRUE
3. Save

**Expected Result:**
- Rule saved
- When a Purchase Header is created without a Project No., system blocks posting with a validation error

**BC Objects:** Table 50122 HBVG Mandatory Fields Header, Codeunit HBVG MandFields

---

## TC-007
**Module:** Setup
**Process:** License Validation
**Type:** Positive

**Prerequisites:**
- Valid HB Vastgoed license installed

**Steps:**
1. Open **HBVG License** page
2. Verify license shows correct:
   - Company name
   - Expiry date
   - Modules enabled

**Expected Result:**
- License page opens without error
- All licensed modules shown as active

**BC Objects:** Table 50107 HBVG License

---

## TC-008
**Module:** Setup
**Process:** Dimension Setup for Projects
**Type:** Positive

**Prerequisites:**
- BC Dimensions exist (e.g., PROJECT, COMPLEX)

**Steps:**
1. Open **HBVG Dimension Setup**
2. Map: Dimension Role = "Project" → Dimension Code = "PROJECT"
3. Map: Dimension Role = "Complex" → Dimension Code = "COMPLEX"
4. Save

**Expected Result:**
- Dimension mappings saved
- When a project is created, the PROJECT dimension is auto-populated on G/L entries

**BC Objects:** Table 50109 HBVG Dimension Setup, Codeunit 50357 HBVGDimensionMgt

---

## TC-009
**Module:** Setup
**Process:** No. Series Validation — Duplicate Series
**Type:** Negative

**Prerequisites:**
- HBVG Setup exists with Project No. Series = "PROJ"

**Steps:**
1. Open HBVG Setup
2. Try to assign the same No. Series "PROJ" to both Projects and Calculations

**Expected Result:**
- System warns or errors: No. Series already in use
- Or: Two separate records created but with duplicate series (test that posting fails gracefully)

---

## TC-010
**Module:** Setup
**Process:** Posting Period Control
**Type:** Positive

**Prerequisites:**
- HBVG Setup exists
- Multiple companies exist

**Steps:**
1. In Central company, run **"Block Posting"** for period Jan 2026
2. Verify all satellite companies now have posting blocked for Jan 2026

**Expected Result:**
- All companies simultaneously blocked
- Attempting to post in Jan 2026 in satellite company gives: "Posting not allowed for this period"

**BC Objects:** Codeunit 50171 HBVGBeheerMgt

---

---

# AREA 2 — REAL ESTATE MANAGEMENT (VASTGOEDBEHEER)

---

## TC-011
**Module:** Vastgoedbeheer
**Process:** Create Complex (Building)
**Type:** Positive

**Prerequisites:**
- HBVG Setup configured
- BC Location exists

**Steps:**
1. Open **HBVG Complex List**
2. Create New Complex:
   - No.: AUTO (from No. Series)
   - Description: "De Lindenhof"
   - Address: "Lindenlaan 1"
   - City: "Amsterdam"
   - Location Code: (select valid location)
   - No. of Units: 30
3. Save

**Expected Result:**
- Complex created with auto-number
- Record retrievable in Complex List
- Complex available as lookup in Project, Contract, and other modules

**BC Objects:** Table 50100 HBVG Complex, Page HBVG Complex Card

---

## TC-012
**Module:** Vastgoedbeheer
**Process:** Create Complex — Missing Mandatory Fields
**Type:** Negative

**Prerequisites:**
- HBVG Setup configured

**Steps:**
1. Open **HBVG Complex List** → New
2. Leave Description blank
3. Try to close/save

**Expected Result:**
- Validation error: "Description must not be empty" or similar
- Record not saved

---

## TC-013
**Module:** Vastgoedbeheer
**Process:** Add Contacts to Complex
**Type:** Positive

**Prerequisites:**
- Complex "De Lindenhof" exists (TC-011)
- BC Contact records exist

**Steps:**
1. Open Complex Card for "De Lindenhof"
2. Navigate to **Complex Contacts** subpage
3. Add Contact: Role = "Technical Manager", Contact No. = (select valid BC Contact)
4. Add Contact: Role = "Housing Officer"
5. Save

**Expected Result:**
- Contacts linked to complex
- Both contacts visible in Complex Contacts list

**BC Objects:** Table 50278 HBVG Complex Contacts

---

## TC-014
**Module:** Vastgoedbeheer
**Process:** Create Contract (Rental Agreement)
**Type:** Positive

**Prerequisites:**
- Complex exists (TC-011)
- BC Customer (tenant) exists

**Steps:**
1. Open **HBVG Contract List** → New
2. Fill:
   - Complex No.: "De Lindenhof"
   - Contract No.: AUTO
   - Customer No.: (select tenant customer)
   - Start Date: 01-01-2026
   - Rent Amount: 750.00
   - Contract Status: Active
3. Add Contract Lines:
   - Line 1: Type = Rent, Amount = 750.00
   - Line 2: Type = Service Costs, Amount = 85.00
4. Save

**Expected Result:**
- Contract created and linked to complex
- Contract lines saved
- Contract visible in Complex Card's contracts subpage

**BC Objects:** Table 50281 HBVG Contract, Table 50282 HBVG Contract Line

---

## TC-015
**Module:** Vastgoedbeheer
**Process:** Contract — Invalid Status Transition
**Type:** Negative

**Prerequisites:**
- Active contract exists (TC-014)
- Contract has posted invoices against it

**Steps:**
1. Open the contract
2. Try to set Contract Status = "Draft" (downgrade)

**Expected Result:**
- Error: Cannot revert status when active entries exist
- Status remains "Active"

---

## TC-016
**Module:** Vastgoedbeheer
**Process:** Contract Termination
**Type:** Positive

**Prerequisites:**
- Active contract exists (TC-014)
- No open invoices

**Steps:**
1. Open Contract Card
2. Set Status = "Terminated"
3. Fill End Date = 31-03-2026
4. Save

**Expected Result:**
- Status updated to Terminated
- End Date saved
- Contract no longer available for new transactions

---

## TC-017
**Module:** Vastgoedbeheer
**Process:** Complex G/L Line Setup
**Type:** Positive

**Prerequisites:**
- Complex exists
- BC G/L Accounts exist

**Steps:**
1. Open Complex Card
2. Navigate to **G/L Lines** subpage (HBVG Complex - G/L Line)
3. Add line: G/L Account = "Rental Income", Percentage = 100%
4. Save

**Expected Result:**
- G/L line saved
- G/L line used during invoice creation from this complex

**BC Objects:** Table 50352 HBVG Complex - G/L Line

---

---

# AREA 3 — PROJECT MANAGEMENT (PROJECTEN)

---

## TC-026
**Module:** Projecten
**Process:** Create Project
**Type:** Positive

**Prerequisites:**
- HBVG Setup with Project No. Series configured
- Complex exists ("De Lindenhof")
- Project Posting Group exists
- BC Customer exists (for Bill-To)

**Steps:**
1. Open **HBVG Project List** → New
2. Fill:
   - No.: AUTO
   - Description: "Renovatie De Lindenhof 2026"
   - Complex No.: "De Lindenhof"
   - Project Type: Renovation
   - Project Category: Social Housing
   - Project Posting Group: (select valid group)
   - Bill-To Customer No.: (select customer)
   - Global Dimension 1 Code: (select valid dimension value)
   - Status: Initiation
3. Save

**Expected Result:**
- Project created with auto-number
- Project dimension automatically set on the project card
- Project visible in Project List

**BC Objects:** Table 50xxx HBVG Project, Page HBVG Project Card

---

## TC-027
**Module:** Projecten
**Process:** Create Project — Missing Posting Group
**Type:** Negative

**Steps:**
1. Create new project (same as TC-026)
2. Leave **Project Posting Group** blank
3. Try to save

**Expected Result:**
- Error: "Project Posting Group must not be empty"
- Record not saved

---

## TC-028
**Module:** Projecten
**Process:** Project Status Transitions
**Type:** Positive

**Prerequisites:**
- Project exists in "Initiation" status (TC-026)

**Steps:**
1. Open Project Card
2. Change Status: Initiation → Feasibility → Planning → Execution → Completed
3. Save at each stage

**Expected Result:**
- Each status transition saved correctly
- Status history can be tracked via planning stages

---

## TC-029
**Module:** Projecten
**Process:** Project Status — Cannot Change if Entries Exist
**Type:** Negative

**Prerequisites:**
- Project exists with at least one posted G/L entry

**Steps:**
1. Open Project Card
2. Try to change Status from "Execution" back to "Planning"

**Expected Result:**
- Error: "You cannot change Status because this Project has one or more entries"
- Status reverts to original

---

## TC-030
**Module:** Projecten
**Process:** Project Blocking
**Type:** Positive

**Prerequisites:**
- Active project exists

**Steps:**
1. Open Project Card
2. Set **Blocked** = "All Posting"
3. Save
4. Try to create a Purchase Order referencing this project

**Expected Result:**
- Purchase Order cannot be created: "Project is blocked"
- No new G/L entries can reference this project

---

## TC-031
**Module:** Projecten
**Process:** Add Project Team Members
**Type:** Positive

**Prerequisites:**
- Project exists
- BC Users / Employees exist
- Security Groups exist

**Steps:**
1. Open Project Card
2. Navigate to **Project Team** subpage
3. Add member: User = "user1", Job Title = "Project Manager", Security Group = "PROJ-MGR"
4. Add member: User = "user2", Job Title = "Technical Manager"
5. Save

**Expected Result:**
- Team members saved
- Members visible in Project Team subpage
- Approval routing will use these team members

---

## TC-032
**Module:** Projecten
**Process:** Set Up Approval per GL Account
**Type:** Positive

**Prerequisites:**
- Project exists
- Security Groups configured

**Steps:**
1. Open Project Card
2. Navigate to **Approval per G/L Account** (HBVG Approval Proj./GL Account)
3. Add line: G/L Account = "6100 Construction Costs", Max Amount = 50,000, Security Group = "PROJ-MGR"
4. Add line: G/L Account = "6100", Max Amount = 200,000, Security Group = "DIRECTOR"
5. Save

**Expected Result:**
- Approval thresholds saved per GL account
- Amounts ≤ 50,000 routed to PROJ-MGR
- Amounts > 50,000 routed to DIRECTOR

**BC Objects:** Table 50240 HBVG Approval Proj./GL Account

---

## TC-033
**Module:** Projecten
**Process:** Project Deletion — With Entries
**Type:** Negative

**Prerequisites:**
- Project with posted purchase invoice exists

**Steps:**
1. Open Project List
2. Select project with entries
3. Try to delete

**Expected Result:**
- Error: "Project cannot be deleted because Purchase Invoice entries exist"
- Project remains in database

---

## TC-034
**Module:** Projecten
**Process:** Project Deletion — No Entries
**Type:** Positive

**Prerequisites:**
- Project exists with no entries, no orders, no budgets

**Steps:**
1. Open Project List
2. Select empty project
3. Delete

**Expected Result:**
- Project deleted without error
- Related dimensions cleaned up (DeleteDimensions called)

---

## TC-035
**Module:** Projecten
**Process:** Project Stages and Statuses
**Type:** Positive

**Prerequisites:**
- Project exists

**Steps:**
1. Open Project Card → Stages subpage
2. Add Stage: Code = "1-DESIGN", Description = "Design Phase", Start Date = 01-01-2026, End Date = 31-03-2026
3. Add Stage: Code = "2-CONSTRUCTION", Description = "Construction", Start Date = 01-04-2026, End Date = 31-12-2026
4. Save

**Expected Result:**
- Stages saved with dates
- Stages visible in project planning overview

---

---

# AREA 4 — CALCULATIONS (CALCULATIES)

---

## TC-051
**Module:** Calculaties
**Process:** Create Calculation
**Type:** Positive

**Prerequisites:**
- Project exists (TC-026)
- Calculation stages/checkpoints defined in setup

**Steps:**
1. Open **HBVG Calculation List** → New
2. Fill:
   - No.: AUTO
   - Project No.: (select project from TC-026)
   - Stage: (select valid stage)
   - Checkpoint: (select valid checkpoint)
   - Description: "Initial Cost Estimate"
3. Save

**Expected Result:**
- Calculation record created and linked to project
- Calculation number generated from No. Series

**BC Objects:** Table 50146 HBVG Calculation

---

## TC-052
**Module:** Calculaties
**Process:** Add Calculation Lines
**Type:** Positive

**Prerequisites:**
- Calculation exists (TC-051)
- Construction types configured

**Steps:**
1. Open Calculation Card
2. Navigate to **Calculation Lines** subpage
3. Add Line 1: Construction Type = "Foundation", Description = "Foundation works", Quantity = 1, Unit Cost = 80,000
4. Add Line 2: Construction Type = "Roof", Description = "New roof", Quantity = 1, Unit Cost = 120,000
5. Add Line 3: Construction Type = "Installations", Description = "HVAC", Quantity = 1, Unit Cost = 60,000
6. Save

**Expected Result:**
- Lines saved
- Total amount on Calculation header = 260,000
- Lines available for budget proposal

**BC Objects:** Table 50102 HBVG Calculation Line

---

## TC-053
**Module:** Calculaties
**Process:** Import Calculation
**Type:** Positive

**Prerequisites:**
- Valid calculation import file (CSV/Excel) prepared
- Import format matches HBVG template

**Steps:**
1. Open Calculation Card
2. Run **"Import Calculation"** action (Report 50175)
3. Select import file
4. Confirm import

**Expected Result:**
- Calculation lines populated from file
- Imported By = current user
- Imported At = current datetime
- Qty. Errors = 0, Qty. Warnings = 0

---

## TC-054
**Module:** Calculaties
**Process:** Import Calculation — File with Errors
**Type:** Negative

**Steps:**
1. Import a file with invalid Construction Types

**Expected Result:**
- Import completes but Qty. Errors > 0
- Error log shown listing unrecognized values
- Invalid lines not imported (or imported with error flag)

---

## TC-055
**Module:** Calculaties
**Process:** Mark Calculation as Processed
**Type:** Positive

**Prerequisites:**
- Calculation with lines exists
- All errors resolved

**Steps:**
1. Open Calculation Card
2. Run **"Process Calculation"** action
3. Confirm

**Expected Result:**
- Calculation Processed = TRUE
- Processing Date = today
- Processed By = current user
- Calculation locked for further editing

---

## TC-056
**Module:** Calculaties
**Process:** Calculation History
**Type:** Positive

**Prerequisites:**
- Multiple versions of calculation exist for same project

**Steps:**
1. Open Project Card
2. Navigate to Calculation History (Table 50175 HBVG Calculation History)
3. Review history entries

**Expected Result:**
- All calculation versions visible
- Each version shows: date, version, total amount, stage
- Original amounts preserved even after new versions

---

---

# AREA 5 — BUDGET MANAGEMENT (BUDGET)

---

## TC-066
**Module:** Budget
**Process:** Create Budget Name
**Type:** Positive

**Prerequisites:**
- HBVG Setup exists

**Steps:**
1. Open **HBVG Budget Names** list → New
2. Fill:
   - Budget Name: "PRJ2026-001"
   - Description: "Budget Renovation De Lindenhof 2026"
   - Budget Type: Project
3. Set Budget Dimensions: Dimension 1 = PROJECT, Dimension 2 = COMPLEX
4. Save

**Expected Result:**
- Budget Name record created
- Dimension setup saved for this budget
- Budget available for posting

**BC Objects:** Table 50346 HBVG Budget Name

---

## TC-067
**Module:** Budget
**Process:** Create Budget Posts
**Type:** Positive

**Prerequisites:**
- Budget Name "PRJ2026-001" exists (TC-066)
- G/L Accounts exist (Construction Costs, Labor, etc.)
- Project dimension values exist

**Steps:**
1. Open **HBVG Budget** page for "PRJ2026-001"
2. Add Budget Post 1:
   - Date: 31-03-2026
   - G/L Account: 6100 (Construction Costs)
   - Project No.: (select project)
   - Amount: 260,000
3. Add Budget Post 2:
   - Date: 30-06-2026
   - G/L Account: 6200 (Labor)
   - Project No.: same project
   - Amount: 80,000
4. Save

**Expected Result:**
- Budget posts saved
- Dimension Set ID auto-calculated from Project No. + dimensions
- Total budget = 340,000

**BC Objects:** Table 50347 HBVG Budgetpost

---

## TC-068
**Module:** Budget
**Process:** Budget Dimension Validation
**Type:** Negative

**Prerequisites:**
- Budget Name exists with Dimension 1 = PROJECT (mandatory)

**Steps:**
1. Create Budget Post
2. Leave Global Dimension 1 Code (PROJECT) blank
3. Try to save

**Expected Result:**
- Error: "Dimension PROJECT must be specified"
- Record not saved

---

## TC-069
**Module:** Budget
**Process:** Budget vs Actual Comparison
**Type:** Positive

**Prerequisites:**
- Budget posts exist (TC-067)
- At least one Purchase Invoice posted against same project and G/L Account

**Steps:**
1. Open **HBVG Budget Overview** or Analysis View
2. Filter by Project No. and G/L Account 6100
3. Review: Budget Amount vs Actual Amount

**Expected Result:**
- Budget column shows: 260,000
- Actual column shows: posted invoice amounts
- Variance column shows: difference
- Remaining budget = Budget - Actual

---

## TC-070
**Module:** Budget
**Process:** Budget Cannot Be Changed if in Analysis View
**Type:** Negative

**Prerequisites:**
- Budget post exists
- Analysis View updated and includes this budget

**Steps:**
1. Open Budget Post record
2. Try to change Amount or G/L Account

**Expected Result:**
- Error: "You cannot change this field because related Analysis View Budget Entries exist"
- Field remains unchanged

**BC Objects:** Codeunit VerifyNoRelatedAnalysisViewBudgetEntries in HBVGBudgetpost

---

## TC-071
**Module:** Budget
**Process:** Budget Dimension Set ID Recalculation
**Type:** Positive

**Prerequisites:**
- Budget post exists with Dimension Set ID

**Steps:**
1. Open Budget Post
2. Change Global Dimension 2 Code to different valid value
3. Save

**Expected Result:**
- Dimension Set ID automatically recalculated (UpdateDimensionSetId called)
- New Dimension Set ID saved
- No manual recalculation needed

---

## TC-072
**Module:** Budget
**Process:** Budget Transfer (Neutral Budget Shift)
**Type:** Positive

**Prerequisites:**
- Budget exists for two G/L Accounts on same project
- Both have budget amounts

**Steps:**
1. Open **HBVG Neutral Budget Shift** page (Page 50184)
2. From G/L Account: 6100, Amount: 20,000
3. To G/L Account: 6200
4. Project: same project
5. Confirm transfer

**Expected Result:**
- Budget post for 6100 reduced by 20,000
- Budget post for 6200 increased by 20,000
- Total project budget unchanged
- Transfer logged

---

---

# AREA 6 — ADDITIONAL BUDGET (AANVULLEND BUDGET)

---

## TC-086
**Module:** Aanvullend Budget
**Process:** Create Additional Budget Request
**Type:** Positive

**Prerequisites:**
- Project exists with approved budget
- Unexpected costs identified (e.g., asbestos removal)
- User has appropriate role

**Steps:**
1. Open **HBVG Additional Budget Request** list → New
2. Fill:
   - Project No.: (select project)
   - G/L Account: 6150 (Asbestos removal)
   - Requested Amount: 60,000
   - Motivation: "Asbestos found in building B during renovation"
   - Request Date: today
3. Save

**Expected Result:**
- Request record created
- Status = Draft
- Request linked to project

**BC Objects:** Table 50249 HBVG Additional Budget Request

---

## TC-087
**Module:** Aanvullend Budget
**Process:** Send Additional Budget for Approval
**Type:** Positive

**Prerequisites:**
- Additional Budget Request exists in Draft (TC-086)
- Approval workflow configured for Budget Requests

**Steps:**
1. Open Additional Budget Request
2. Run action: **"Send Approval Request"**

**Expected Result:**
- Status changes to "Pending Approval"
- Approval entry created for correct approver
- Approver receives notification

---

## TC-088
**Module:** Aanvullend Budget
**Process:** Approve Additional Budget Request
**Type:** Positive

**Prerequisites:**
- Additional Budget Request pending approval (TC-087)
- Logged in as approver

**Steps:**
1. Open **Requests to Approve** list
2. Find the additional budget request
3. Click **Approve**

**Expected Result:**
- Approval Entry status = Approved
- Additional Budget Request status = Approved
- Budget amount added to project budget
- Approved By and Approved Date populated

---

## TC-089
**Module:** Aanvullend Budget
**Process:** Reject Additional Budget Request
**Type:** Positive

**Prerequisites:**
- Additional Budget Request pending approval

**Steps:**
1. Open Requests to Approve
2. Find request
3. Click **Reject**
4. Fill rejection reason

**Expected Result:**
- Status = Rejected
- Rejection reason saved
- Budget NOT increased
- Requester notified

---

---

# AREA 7 — PROJECT PLANNING (PLANNING)

---

## TC-096
**Module:** Planning
**Process:** Create Project Planning
**Type:** Positive

**Prerequisites:**
- Project exists (TC-026)

**Steps:**
1. Open Project Card
2. Navigate to **Project Planning** (Table 50300)
3. Create Planning Header:
   - Project No.: (select project)
   - Planning Date: 01-01-2026
   - Description: "Master Planning De Lindenhof"
4. Add Planning Lines:
   - Line 1: Phase = Design, Planned Start = 01-01-2026, Planned End = 31-03-2026
   - Line 2: Phase = Permits, Planned Start = 01-02-2026, Planned End = 31-07-2026
   - Line 3: Phase = Construction, Planned Start = 01-08-2026, Planned End = 31-07-2027
   - Line 4: Phase = Handover, Planned Start = 01-08-2027, Planned End = 31-08-2027
5. Save

**Expected Result:**
- Planning header and lines saved
- Planning linked to project
- Gantt / timeline view shows phases

**BC Objects:** Table 50300 HBVG Project Planning, Table 50301 HBVG Project Planning Line

---

## TC-097
**Module:** Planning
**Process:** Send Planning for Approval
**Type:** Positive

**Prerequisites:**
- Planning exists in Draft status
- Approval workflow for Planning configured
- IsDVUserAllowed returns TRUE (user in project team with "Approve Planning" flag)

**Steps:**
1. Open Project Planning Card
2. Run **"Send for Approval"** action

**Expected Result:**
- Approval Entry created, routed to correct approver (FindApproverPlanning called)
- Planning status = "Pending Approval"
- Approver notified

**BC Objects:** Codeunit 50385 HBVGApprovalsManagement (SendPlanningApprovalRequest)

---

## TC-098
**Module:** Planning
**Process:** Planning Approval — Approver Not Found
**Type:** Negative

**Prerequisites:**
- Planning exists
- NO user configured with "Approve Planning" authority for this project

**Steps:**
1. Open Project Planning
2. Run **"Send for Approval"**

**Expected Result:**
- Error: "Approver not found"
- No approval entry created
- Status remains Draft

---

## TC-099
**Module:** Planning
**Process:** Add Planning Comment
**Type:** Positive

**Prerequisites:**
- Project Planning exists

**Steps:**
1. Open Planning Card
2. Navigate to Comments subpage (Table 50302 HBVG Project Planning Comment)
3. Add comment: "Permits expected to be delayed due to neighbourhood objections"
4. Save

**Expected Result:**
- Comment saved with date and user
- Comment visible in planning comments list

---

---

# AREA 8 — DRAFT ORDERS (OPDRACHTEN)

---

## TC-111
**Module:** Opdrachten
**Process:** Create Draft Order
**Type:** Positive

**Prerequisites:**
- Project exists and is active (not blocked)
- Budget exists for the project
- Vendor exists

**Steps:**
1. Open **HBVG Draft Orders** list → New
2. Fill Header:
   - No.: AUTO
   - Project No.: (select project)
   - Vendor No.: (select contractor)
   - Order Date: today
   - Description: "Roofing works Building A"
3. Add Order Lines:
   - Line 1: Type = G/L Account, No. = 6100, Description = "Materials", Quantity = 1, Unit Cost = 80,000
   - Line 2: Type = G/L Account, No. = 6200, Description = "Labor", Quantity = 1, Unit Cost = 40,000
4. Save

**Expected Result:**
- Draft Order created and linked to project
- Total amount = 120,000
- Draft Order status = Open

**BC Objects:** Table 50315 HBVG Draft Order, Table 50316 HBVG Draft Order Line

---

## TC-112
**Module:** Opdrachten
**Process:** Draft Order — Project Blocked
**Type:** Negative

**Prerequisites:**
- Project with Blocked = "All Posting"

**Steps:**
1. Create new Draft Order
2. Set Project No. to blocked project

**Expected Result:**
- Error: "Project is blocked"
- Project No. cannot be set

---

## TC-113
**Module:** Opdrachten
**Process:** Add Text to Draft Order
**Type:** Positive

**Prerequisites:**
- Draft Order exists

**Steps:**
1. Open Draft Order Card
2. Navigate to **Draft Order Text** (Table 50318)
3. Add text block: Title = "Scope", Body = "All roofing works as per specification ref. 2026-R-001"
4. Save

**Expected Result:**
- Text block saved and linked to Draft Order
- Text visible on Draft Order print/report

**BC Objects:** Table 50318 HBVG Draft Order Text

---

## TC-114
**Module:** Opdrachten
**Process:** Convert Draft Order to Purchase Order
**Type:** Positive

**Prerequisites:**
- Draft Order approved (or in correct status)
- Vendor has valid posting setup in BC

**Steps:**
1. Open Draft Order Card
2. Run action: **"Create Purchase Order"**
3. Confirm

**Expected Result:**
- BC Purchase Order created with same lines
- Draft Order linked to Purchase Order (Purchase Order No. stored)
- Draft Order status updated to "Ordered"
- HBVG Obligation created automatically (one per G/L Account line)

---

## TC-115
**Module:** Opdrachten
**Process:** Send Draft Order for Approval
**Type:** Positive

**Prerequisites:**
- Draft Order in Draft status
- Approval workflow active for Draft Orders

**Steps:**
1. Open Draft Order
2. Run **"Send for Approval"**

**Expected Result:**
- Approval entries created per GL Account line (based on amount thresholds)
- Status = Pending Approval
- Approvers notified

---

---

# AREA 9 — OBLIGATIONS (VERPLICHTINGEN)

---

## TC-126
**Module:** Verplichtingen
**Process:** Obligation Created from Purchase Order
**Type:** Positive

**Prerequisites:**
- Purchase Order created with HBVG Project No. populated on lines

**Steps:**
1. Create BC Purchase Order with:
   - Vendor No.: Bouw BV
   - Line: G/L Account 6100, Quantity 1, Direct Unit Cost 80,000
   - HBVG Project No.: (select project)
2. Release Purchase Order
3. Open **HBVG Obligation List**

**Expected Result:**
- Obligation record created:
  - Obligation No. = Purchase Order No.
  - Obligation Line No. = Purchase Order Line No.
  - Amount = 80,000
  - Status = Open
  - HBVG Obligation Closed = FALSE

**BC Objects:** Table (Purchase Line extension), Codeunit HBVGSubscrInkoopT38T39

---

## TC-127
**Module:** Verplichtingen
**Process:** Obligation Updated When Invoice Posted
**Type:** Positive

**Prerequisites:**
- Open obligation exists for PO (TC-126)
- Purchase Invoice created referencing same PO

**Steps:**
1. Create Purchase Invoice from PO (receive and invoice)
2. Post the Purchase Invoice

**Expected Result:**
- Codeunit HBVGSubscrInkoopC90 fires during posting
- HBVG Posted Amount on obligation line updated
- If Posted Amount = Order Amount → HBVG Obligation Closed = TRUE
- Purchase Header: HBVG Obligation Closed = TRUE (if all lines closed)

**BC Objects:** Codeunit 50xxx HBVGSubscrInkoopC90 (C90CheckAutomaticClosePurch)

---

## TC-128
**Module:** Verplichtingen
**Process:** Partial Invoice — Obligation Stays Open
**Type:** Positive

**Prerequisites:**
- Open obligation for 80,000

**Steps:**
1. Create Purchase Invoice for 50% of PO (40,000)
2. Post the invoice

**Expected Result:**
- Posted Amount = 40,000
- Obligation Closed = FALSE (still open for remaining 40,000)
- Budget consumed by 40,000

---

## TC-129
**Module:** Verplichtingen
**Process:** Credit Memo Reopens Obligation
**Type:** Positive

**Prerequisites:**
- Obligation fully closed (TC-127)

**Steps:**
1. Create Purchase Credit Memo referencing original invoice
2. Set Accept Credit Amounts = TRUE on line
3. Post the credit memo

**Expected Result:**
- Codeunit C90UpdateAndOpenPurch fires
- HBVG Obligation Closed = FALSE (reopened)
- HBVG Accept Credit Amounts = TRUE
- Posted Amount reduced by credit amount

**BC Objects:** Codeunit HBVGSubscrInkoopC90 (C90UpdateAndOpenPurch)

---

## TC-130
**Module:** Verplichtingen
**Process:** Outstanding Obligations Report
**Type:** Positive

**Prerequisites:**
- Multiple open and closed obligations exist

**Steps:**
1. Run Report 50174: **"HBVG Uitstaande Verplichtingen"**
2. Filter: Show only Open obligations
3. Filter by Project No.

**Expected Result:**
- Report shows only unclosed obligations
- Total amount = sum of all open obligation amounts
- Grouped by Project and G/L Account

---

## TC-131
**Module:** Verplichtingen
**Process:** Current Account Obligation — Missing Ratio Setup
**Type:** Negative

**Prerequisites:**
- Purchase line with "HBVG Curr. Account Company" populated
- No HBVG Current Account Ratio configured for that company

**Steps:**
1. Post Purchase Invoice with current account line

**Expected Result:**
- Error: "Relationship with [company name] does not exist"
- Posting blocked

**BC Objects:** Codeunit HBVGSubscrInkoopC90 (C90ControleerRekeningCourant)

---

## TC-132
**Module:** Verplichtingen
**Process:** Current Account — Missing Dimension
**Type:** Negative

**Prerequisites:**
- Current Account Ratio configured
- Purchase line has current account but dimension is blank

**Steps:**
1. Post Purchase Invoice

**Expected Result:**
- Error: "An invalid current account dimension has been specified"
- Posting blocked

---

## TC-133
**Module:** Verplichtingen
**Process:** Link Purchase Invoice to Existing Obligation
**Type:** Positive

**Prerequisites:**
- Open Purchase Order (obligation) exists
- Separate Purchase Invoice received

**Steps:**
1. Open **HBVG Link Purch.Inv/Obligation** page (Page 50138)
2. Select Purchase Invoice
3. Select matching Obligation (PO line)
4. Confirm link

**Expected Result:**
- Invoice linked to obligation
- Obligation Updated Amount reflects linked invoice amount

---

---

# AREA 10 — PURCHASE TO OBLIGATION FULL FLOW

---

## TC-146
**Module:** Full Flow
**Process:** End-to-End Purchase Cycle with Obligation Tracking
**Type:** Positive

**Prerequisites:**
- Project with approved budget exists
- Vendor configured
- G/L Account with dimension setup
- Approval workflow configured

**Steps:**
1. **Create Draft Order** for project (TC-111)
2. **Send for Approval** (TC-115)
3. **Approve** as authorized user
4. **Convert to Purchase Order** → Obligation auto-created (TC-114)
5. **Receive goods/services** (post receipt)
6. **Receive Invoice** → Create Purchase Invoice from PO
7. **Approve Invoice** through workflow
8. **Post Invoice** → Obligation auto-updated (TC-127)
9. **Pay Vendor** via Payment Journal
10. Check: Budget consumed, Obligation closed, Vendor paid

**Expected Result at Each Step:**
- Step 3: Approval Entry = Approved
- Step 4: PO created, Obligation created (Open, Amount = PO amount)
- Step 8: GL Entry posted with Project dimension, Obligation Closed = TRUE
- Step 9: Vendor Ledger Entry = Closed, Bank = reduced
- Budget remaining = Original Budget - Posted Amount

---

---

# AREA 11 — APPROVAL WORKFLOWS

---

## TC-156
**Module:** Approvals
**Process:** Send Purchase Invoice for Approval
**Type:** Positive

**Prerequisites:**
- Purchase Invoice exists with HBVG Project No. on lines
- Approval workflow for Purchase Invoices active
- Approver configured for the project/GL Account combination

**Steps:**
1. Open Purchase Invoice
2. Run **"Send Approval Request"**

**Expected Result:**
- HBVGApprovalsManagement.FindApproverPurchase() executes
- System groups lines by Project + GL Account
- Approval entries created for each unique combination
- "HBVG Approval Status" on header = Pending
- Approver notified

**BC Objects:** Codeunit 50385 HBVGApprovalsManagement

---

## TC-157
**Module:** Approvals
**Process:** Approval — Already Exists
**Type:** Negative

**Prerequisites:**
- Approval already sent for document

**Steps:**
1. Try to send approval request again for same document

**Expected Result:**
- Error: "Approval requests already exist"
- No duplicate approval entries created

---

## TC-158
**Module:** Approvals
**Process:** Approval — No Amount Setting Available
**Type:** Negative

**Prerequisites:**
- Purchase Invoice with GL Account 6100, Amount 500,000
- No approval threshold configured for GL Account 6100 at 500,000 level

**Steps:**
1. Send Invoice for Approval

**Expected Result:**
- Error: "No amount setting available for GL Account 6100 at amount 500,000"
- OR: Error about GoedkeuringVolledig not complete
- Approval blocked

---

## TC-159
**Module:** Approvals
**Process:** Approval — User Not in Project Team
**Type:** Negative

**Prerequisites:**
- User NOT in project team for the project on the invoice

**Steps:**
1. Try to approve a document referencing a project user is not part of

**Expected Result:**
- IsDVUserAllowed returns FALSE
- Error: User not authorized to approve for this project

---

## TC-160
**Module:** Approvals
**Process:** Multi-Company Approval
**Type:** Positive

**Prerequisites:**
- Company A has a document to approve
- Approver is set up in Company B (HBVG Approval All Companies)

**Steps:**
1. In Company A: Create Purchase Order → Send for Approval
2. Switch to Company B: Open **HBVG Approval All Companies** list
3. Find approval entry for Company A document
4. Approve

**Expected Result:**
- Approval shows cross-company
- Company A document status updated to Approved
- No need to switch back to Company A to approve

**BC Objects:** Table 50132 HBVG Approval All Companies, Codeunit 50132 HBVGApprovalAllCompanyMgt

---

## TC-161
**Module:** Approvals
**Process:** Delegation — Not Allowed with Extended Approvals
**Type:** Negative

**Prerequisites:**
- Extended Approval module enabled in HBVG Setup

**Steps:**
1. Open pending approval entry
2. Try to Delegate to another user

**Expected Result:**
- Error: "When using the extended approval module, delegation is not allowed"
- Delegation blocked

---

## TC-162
**Module:** Approvals
**Process:** Approval Overview Template
**Type:** Positive

**Steps:**
1. Run Report 50103: **"HBVG Approval Overview Template"**
2. Select project filter

**Expected Result:**
- Report shows all approval entries for project
- Status per document shown
- Approver names and dates shown

---

---

# AREA 12 — INVOICING ENGINE (FACTURATIE)

---

## TC-176
**Module:** Facturatie
**Process:** Auto-Generate Sales Invoice from GL Entries
**Type:** Positive

**Prerequisites:**
- G/L entries exist with:
  - HBVG Create Sales Invoice = TRUE
  - HBVG Bill-To = valid Customer No.
  - HBVG VAT Post = FALSE
- Customer has valid Sales posting setup

**Steps:**
1. Open **HBVG Blanket Orders Invoicing** or run Facturatie codeunit
2. Select Invoice Type = "GB" (General without project)
3. Run

**Expected Result:**
- Sales Invoice created per unique Bill-To customer
- Sales Invoice lines: one per GL entry, Unit Price = GL Entry amount
- GL Entry field "HBVG Invoice Created" = Sales Invoice No.
- Sales Invoice status = Open (not posted yet)

**BC Objects:** Codeunit 50188 HBVGFacturatie

---

## TC-177
**Module:** Facturatie
**Process:** Project Invoicing (GBPrj type)
**Type:** Positive

**Prerequisites:**
- G/L entries with HBVG Create Sales Invoice = TRUE and HBVG Project No. filled
- Project has "Sales Account Invoiced WIP" configured in Posting Group
- "HBVG Amount To Be Invoiced" > 0 on GL entry

**Steps:**
1. Run Facturatie with Type = "GBPrj"

**Expected Result:**
- Sales Invoice created for project
- WIP account used on invoice line
- Amount = HBVG Amount To Be Invoiced (not full GL amount)

---

## TC-178
**Module:** Facturatie
**Process:** Amount to Be Invoiced Not Filled
**Type:** Negative

**Prerequisites:**
- GL entry: HBVG Create Sales Invoice = TRUE, HBVG Project No. filled, but HBVG Amount To Be Invoiced = 0

**Steps:**
1. Run Facturatie with Type = "GBPrj"

**Expected Result:**
- Error: "Amount to be invoiced has not been filled in project [No.] document [No.]"
- No invoice created for this GL entry

---

## TC-179
**Module:** Facturatie
**Process:** Invoice Marking — Already Created
**Type:** Positive

**Prerequisites:**
- GL entry already has HBVG Invoice Created = "INV-001"

**Steps:**
1. Run Facturatie again

**Expected Result:**
- GL entry skipped (already invoiced)
- No duplicate invoice created
- INV-001 still the linked invoice

---

## TC-180
**Module:** Facturatie
**Process:** Cost Recharge (Doorberekening)
**Type:** Positive

**Prerequisites:**
- GL entries with Invoice Type = Doorberekening (cost allocation)
- Internal customer configured

**Steps:**
1. Run Facturatie with Type = "Doorberekening"

**Expected Result:**
- Sales Invoice created for internal customer
- Lines show cost allocation amounts
- Cost allocation description from GL entry preserved

---

---

# AREA 13 — LOANS (LENINGEN)

---

## TC-191
**Module:** Leningen
**Process:** Create Loan
**Type:** Positive

**Prerequisites:**
- HBVG Loan Setup configured (Table 50304)
- HBVG Loan Posting Group configured (Table 50305)
- Bank account exists in BC

**Steps:**
1. Open **HBVG Loan List** → New
2. Fill:
   - No.: AUTO
   - Description: "ABN AMRO Renovation Loan"
   - Loan Type: Long Term
   - Principal Amount: 2,000,000
   - Interest Rate: 3.5%
   - Start Date: 01-01-2026
   - End Date: 31-12-2055 (30 year term)
   - Loan Posting Group: (select)
   - Project No.: (link to project)
3. Save

**Expected Result:**
- Loan created with all fields
- Loan available in loan list
- Interest schedule can be generated

**BC Objects:** Table 50307 HBVG Loan

---

## TC-192
**Module:** Leningen
**Process:** Loan Interest Schedule
**Type:** Positive

**Prerequisites:**
- Loan exists (TC-191)

**Steps:**
1. Open Loan Card
2. Navigate to **Loan Interest** subpage (Table 50306)
3. Run action: **"Calculate Interest Schedule"**

**Expected Result:**
- Monthly interest lines generated
- Each line: Period, Principal Outstanding, Interest Amount, Repayment Amount, New Principal
- Total interest across all periods = correct mathematical result
- Schedule covers full loan term (360 months)

---

## TC-193
**Module:** Leningen
**Process:** Post Loan Receipt
**Type:** Positive

**Prerequisites:**
- Loan exists

**Steps:**
1. Open Loan Card
2. Run action: **"Post Loan Receipt"**
3. Posting Date: 01-01-2026, Amount: 2,000,000

**Expected Result:**
- G/L Entry: Debit Bank 2,000,000 / Credit Long-Term Loan Liability 2,000,000
- Loan balance = 2,000,000

---

## TC-194
**Module:** Leningen
**Process:** Post Monthly Interest
**Type:** Positive

**Prerequisites:**
- Loan with interest schedule exists

**Steps:**
1. Run **HBVG Interest Calculation** (Report 50106) for January 2026
2. Review journal lines
3. Post

**Expected Result:**
- Gen. Journal Line: Debit Interest Expense / Credit Bank
- Amount = 3.5% ÷ 12 × 2,000,000 = 5,833.33
- Project dimension on G/L entry

**BC Objects:** Report 50106 HBVG Interest Calc., Table 50267 HBVG Interest

---

## TC-195
**Module:** Leningen
**Process:** Loan Default Interest
**Type:** Positive

**Prerequisites:**
- Loan with default interest rate configured (Table 50310)

**Steps:**
1. Simulate missed payment
2. Apply default interest rate
3. Post default interest

**Expected Result:**
- Default interest amount calculated at higher rate
- Posted to correct G/L Account (interest expense, default rate)

---

## TC-196
**Module:** Leningen
**Process:** Loan Movements Report
**Type:** Positive

**Steps:**
1. Run Report 50181: **"HBVG Loan Movements"**
2. Filter by Loan No.

**Expected Result:**
- Report shows: Opening Balance, Interest Paid, Repayments, Closing Balance
- All movements chronologically listed
- Totals correct

---

---

# AREA 14 — INTEREST (RENTE)

---

## TC-206
**Module:** Rente
**Process:** Interest Setup
**Type:** Positive

**Prerequisites:**
- G/L Accounts for interest exist

**Steps:**
1. Open **HBVG Interest Setup** (Table 50212)
2. Configure:
   - Interest G/L Account: 8100
   - Counterpart Account: 1600 (loan account)
   - Calculation Method: Straight-line
3. Save

---

## TC-207
**Module:** Rente
**Process:** Standard Interest Creation
**Type:** Positive

**Steps:**
1. Open **HBVG Standard Interest** (Table 50271)
2. Create standard interest template:
   - Rate: 3.5%
   - Type: Annual
3. Link to Loan

**Expected Result:**
- Standard interest template available for multiple loans

---

## TC-208
**Module:** Rente
**Process:** Interest Calculation Report
**Type:** Positive

**Steps:**
1. Run Report 50106 **"HBVG Interest Calc."** for period Q1 2026
2. Filter by project

**Expected Result:**
- Interest calculated per loan per project
- Correct amounts based on outstanding principal × rate × days/365
- Journal lines ready for posting

---

---

# AREA 15 — FORECASTS (PROGNOSES)

---

## TC-216
**Module:** Prognoses
**Process:** Create Project Forecast
**Type:** Positive

**Prerequisites:**
- Project exists with budget

**Steps:**
1. Open Project Card → navigate to **Forecasts**
2. Create new Forecast:
   - Forecast Date: 01-03-2026
   - Description: "Q1 2026 Update"
3. Add Forecast Lines:
   - Line 1: G/L Account 6100, Forecasted Amount: 270,000 (budget was 260,000 — over by 10,000)
   - Line 2: G/L Account 6200, Forecasted Amount: 75,000 (under budget)
4. Save

**Expected Result:**
- Forecast saved with lines
- System shows: Budget vs Forecast variance
- Budget 6100: 260,000 vs Forecast 270,000 → variance +10,000 (over)
- Budget 6200: 80,000 vs Forecast 75,000 → variance -5,000 (under)

**BC Objects:** Table 50209 HBVG Project Forecast, Table 50210 HBVG Project Forecast Line

---

## TC-217
**Module:** Prognoses
**Process:** Send Forecast for Approval
**Type:** Positive

**Prerequisites:**
- Forecast exists in Draft
- User configured with "Approve Prognose" in security group

**Steps:**
1. Open Forecast
2. Run **"Send for Approval"**

**Expected Result:**
- Approval entry created (FindApproverPrognose called)
- Forecast status = Pending Approval
- Approver notified

---

## TC-218
**Module:** Prognoses
**Process:** Forecast Approval — No Authority
**Type:** Negative

**Prerequisites:**
- No user with "Approve Prognose" flag for this project

**Steps:**
1. Send Forecast for Approval

**Expected Result:**
- Error: "Approver not found"
- Status remains Draft

---

## TC-219
**Module:** Prognoses
**Process:** Processed Forecast
**Type:** Positive

**Prerequisites:**
- Approved forecast exists

**Steps:**
1. Run Report 50109 **"HBVG Indexing Forecast"** or process forecast
2. Mark forecast as processed

**Expected Result:**
- Forecast Processed = TRUE
- Table 50202 HBVG Processed Forecast updated
- Forecast locked

---

---

# AREA 16 — CASH FLOW (CASHFLOW)

---

## TC-231
**Module:** CashFlow
**Process:** Create Cash Flow Schedule
**Type:** Positive

**Prerequisites:**
- Project with budget and obligations exists

**Steps:**
1. Open **HBVG Cashflow Schedule** (Table 50176)
2. Create schedule line:
   - Project No.: (select project)
   - Period: March 2026
   - Expected Outflow: 120,000 (contractor payment due)
   - Expected Inflow: 0
3. Create another:
   - Period: April 2026
   - Expected Inflow: 15,000 (rent income)
4. Save

**Expected Result:**
- Schedule lines saved
- Running cash flow balance calculable per period

---

## TC-232
**Module:** CashFlow
**Process:** Import Cash Flow from Liquidity File
**Type:** Positive

**Steps:**
1. Run Report 50105 **"HBVG Start Cashflw-Liq. Import"**
2. Select import file with cash flow data
3. Run import

**Expected Result:**
- HBVG Cashflow Posts (Table 50177) populated
- Schedule lines created from import
- No import errors

---

## TC-233
**Module:** CashFlow
**Process:** Cash Flow — Negative Balance Alert
**Type:** Positive

**Prerequisites:**
- Cash flow schedule shows outflow > inflow for a period

**Steps:**
1. Open Cash Flow overview
2. Review period where outflow > inflow

**Expected Result:**
- Period highlighted or marked as negative
- Management can see liquidity shortfall before it happens

---

---

# AREA 17 — GENERAL COSTS & PROVISIONS (AK-VOORZIENING)

---

## TC-241
**Module:** AK-Voorziening
**Process:** Create Provision
**Type:** Positive

**Prerequisites:**
- Project exists
- G/L Accounts for provisions exist

**Steps:**
1. Open **HBVG Provision List** (Table 50270) → New
2. Fill:
   - Code: "ROOF-PROV"
   - Description: "Provision for roof replacement"
   - Annual Amount: 10,000
   - Project No.: (select project)
   - G/L Account: 8200 (Provision expense)
   - Provision Balance Account: 3100 (Provision liability)
3. Save

**Expected Result:**
- Provision created
- Annual provision posting ready for scheduling

---

## TC-242
**Module:** AK-Voorziening
**Process:** Create Default Provision
**Type:** Positive

**Steps:**
1. Open **HBVG Default Provision** (Table 50274)
2. Create template provision to be applied across all new projects
3. Save

**Expected Result:**
- Default provision template saved
- When new project created, default provisions auto-applied

---

## TC-243
**Module:** AK-Voorziening
**Process:** General Costs Allocation
**Type:** Positive

**Steps:**
1. Open **HBVG General Costs** (Table 50268)
2. Create entry:
   - Type: Management Fee
   - Project No.: (select)
   - Annual Amount: 5,000
3. Link to G/L Account
4. Save

**Expected Result:**
- General cost saved
- Cost allocable to project on annual basis

---

## TC-244
**Module:** AK-Voorziening
**Process:** Proposal Generation
**Type:** Positive

**Steps:**
1. Open **HBVG Prop. Gen. Cost/Provision** (Table 50235)
2. Run proposal for period 2026
3. Review lines

**Expected Result:**
- Proposal generated showing all provisions and general costs
- Amounts calculated per period
- Ready for posting approval

---

---

# AREA 18 — BUYER ADMINISTRATION (KOPERSADMINISTRATIE)

---

## TC-256
**Module:** Kopersadministratie
**Process:** Create Construction Number (Bouwnummer)
**Type:** Positive

**Prerequisites:**
- Project exists
- Complex exists
- Customer (buyer) exists in BC

**Steps:**
1. Open **HBVG Construction Numbers** (Table 50163) → New
2. Fill:
   - Construction No.: AUTO
   - Project No.: (select)
   - Complex No.: (select)
   - Unit: "Apartment 5"
   - Buyer Customer No.: (select Tim as customer)
   - Status: Available
3. Save

**Expected Result:**
- Construction number created
- Unit linked to project and complex
- Status = Available (not yet sold)

---

## TC-257
**Module:** Kopersadministratie
**Process:** Assign Buyer to Construction Number
**Type:** Positive

**Prerequisites:**
- Construction Number exists (TC-256)
- Customer (buyer) exists

**Steps:**
1. Open Construction Number Card
2. Set Status = "Sold"
3. Set Buyer Customer No. = Tim
4. Set Sale Price = 280,000
5. Set Transfer Date = 01-06-2026
6. Save

**Expected Result:**
- Status = Sold
- Buyer linked to unit
- Sale price recorded

---

## TC-258
**Module:** Kopersadministratie
**Process:** Buyer Term Contract
**Type:** Positive

**Prerequisites:**
- Construction Number with buyer assigned

**Steps:**
1. Open Construction Number Card
2. Navigate to **Buyer Terms** (Table 50206 HBVG Constr. No. Buyer Term)
3. Add payment term:
   - Term Date: 01-03-2026
   - Description: "Land purchase"
   - Amount: 50,000
4. Add Term 2: 01-06-2026, "Structure complete", 100,000
5. Add Term 3: 01-09-2026, "Final payment", 130,000
6. Save

**Expected Result:**
- Three payment terms totaling 280,000
- Terms linked to construction number
- Each term = installment invoice trigger

---

## TC-259
**Module:** Kopersadministratie
**Process:** Interest on Late Buyer Payment
**Type:** Positive

**Prerequisites:**
- Buyer term with due date passed and not yet paid

**Steps:**
1. Open **HBVG Constr. No. Interest Calc** (Table 50213)
2. Calculate interest for overdue term
3. Post interest charge

**Expected Result:**
- Interest calculated: overdue days × rate × term amount
- Interest invoice created for buyer
- Linked to construction number

---

## TC-260
**Module:** Kopersadministratie
**Process:** Contract Terms Setup
**Type:** Positive

**Steps:**
1. Open **HBVG Constr. No. Contract Terms** (Table 50205)
2. Add contractual conditions for buyer
3. Link to Construction Number

**Expected Result:**
- Contract terms saved
- Terms printable on buyer agreement

---

## TC-261
**Module:** Kopersadministratie
**Process:** Complex Unit Values
**Type:** Positive

**Steps:**
1. Open **HBVG Complex Unit Values** (Table 50285)
2. Enter WOZ value, market value, and rental points per unit
3. Save

**Expected Result:**
- Values saved per unit
- Values used for rent calculation and reporting

---

---

# AREA 19 — MORE-LESS WORK (MMW)

---

## TC-276
**Module:** MMW
**Process:** Create MMW Record (Change Order)
**Type:** Positive

**Prerequisites:**
- Purchase Order with contractor exists
- Contractor has done extra work (asbestos found)

**Steps:**
1. Open **HBVG More-Less Work** list (Table 50111) → New
2. Fill:
   - Related PO No.: (select open PO)
   - Classification: (Table 50112 — e.g., "Meerwerk")
   - Description: "Asbestos removal not in original scope"
   - Amount: 15,000
   - Direction: More Work (+)
3. Save

**Expected Result:**
- MMW record created and linked to Purchase Order
- Running total of more/less work visible
- Net change = +15,000

**BC Objects:** Table 50111 HBVG More-Less Work

---

## TC-277
**Module:** MMW
**Process:** Minderwerk (Less Work)
**Type:** Positive

**Steps:**
1. Create MMW record with Direction = Less Work (-)
2. Description: "Balcony not constructed per client request"
3. Amount: -8,000

**Expected Result:**
- MMW record: amount = -8,000
- Net MMW for contract: 15,000 - 8,000 = +7,000

---

## TC-278
**Module:** MMW
**Process:** MMW Status Handling
**Type:** Positive

**Steps:**
1. Open MMW record
2. Progress status through: Draft → Submitted → Agreed → Posted
3. At "Agreed" status: both parties have signed off

**Expected Result:**
- Status transitions saved
- At "Posted" status: amount incorporated into final contract value
- Budget automatically adjusted for agreed MMW

**BC Objects:** Table 50114 HBVG MMW Status Handling

---

## TC-279
**Module:** MMW
**Process:** MMW Divide Key
**Type:** Positive

**Steps:**
1. Open **HBVG MMW Divide Key** (Table 50113)
2. Create divide key to split MMW cost across multiple projects/units

**Expected Result:**
- Division key saved
- Cost split calculable across projects

---

---

# AREA 20 — TIME REGISTRATION (URENREGISTRATIE)

---

## TC-286
**Module:** Urenregistratie
**Process:** Work Type Setup
**Type:** Positive

**Steps:**
1. Open **HBVG Work Type Employee** (Table 50140)
2. Create Work Type: Code = "PROJ-MGT", Description = "Project Management", Hourly Rate = 95.00
3. Link to G/L Account: 6300 (Internal Labor)
4. Save

**Expected Result:**
- Work type saved with rate
- Available for timesheet entry

---

## TC-287
**Module:** Urenregistratie
**Process:** Register Hours on Project
**Type:** Positive

**Prerequisites:**
- Work type exists (TC-286)
- Employee linked to user
- Project exists

**Steps:**
1. Open Time Sheet (BC standard, extended by HB)
2. Select week: 23-03-2026 to 27-03-2026
3. Add line: Project No. = (select), Work Type = "PROJ-MGT", Monday = 8h, Tuesday = 7h, Wednesday = 8h
4. Submit timesheet

**Expected Result:**
- Time sheet lines saved
- Hours × Rate = cost ready to post

**BC Objects:** TableExt HBVG TimeSheet (extends BC Time Sheet)

---

## TC-288
**Module:** Urenregistratie
**Process:** Process Timesheet — Post to G/L
**Type:** Positive

**Prerequisites:**
- Approved timesheet exists

**Steps:**
1. Run Report 50190 **"HBVG Process Timesheet"**
2. Select timesheet period
3. Confirm posting

**Expected Result:**
- G/L Entry created: Debit Internal Labor 6300, Credit Internal Cost Allocation
- Project dimension on G/L entry
- Hours and cost visible in project cost overview

---

## TC-289
**Module:** Urenregistratie
**Process:** Function Group Setup
**Type:** Positive

**Steps:**
1. Open **HBVG Work Type Function Group** (Table 50139)
2. Create function group: Project Managers, Technicians, Administrative

**Expected Result:**
- Function groups saved
- Used for timesheet categorization and reporting

---

---

# AREA 21 — COVERAGE SURVEILLANCE (DEKKINGTOEZICHT)

---

## TC-296
**Module:** Dekkingtoezicht
**Process:** Create Surveillance Coverage Record
**Type:** Positive

**Steps:**
1. Open **HBVG Surveillance Coverage** (Table 50181) → New
2. Fill:
   - Project No.: (select)
   - Coverage %: 85%
   - Minimum Required Coverage: 80%
   - Monitoring Date: today
3. Save

**Expected Result:**
- Coverage record saved
- Status = OK (85% > 80%)

---

## TC-297
**Module:** Dekkingtoezicht
**Process:** Coverage Below Minimum
**Type:** Positive

**Steps:**
1. Update Coverage % to 75% (below 80% minimum)

**Expected Result:**
- Warning or alert generated
- Status = Warning / Below Minimum
- Report flags this project

---

## TC-298
**Module:** Dekkingtoezicht
**Process:** Dekkingtoezicht Report
**Type:** Positive

**Steps:**
1. Run Dekkingtoezicht report
2. Filter: Show projects below minimum coverage

**Expected Result:**
- Report shows only at-risk projects
- Coverage %, minimum, and gap clearly shown

---

---

# AREA 22 — RISK ANALYSIS (RISICOANALYSE)

---

## TC-306
**Module:** Risicoanalyse
**Process:** Create Risk Analysis
**Type:** Positive

**Prerequisites:**
- Project exists

**Steps:**
1. Open **HBVG Risk Analysis Header** (Table 50248) → New
2. Fill:
   - Project No.: (select)
   - Analysis Date: today
   - Description: "Initial Risk Assessment"
3. Navigate to Risk Analysis Lines (Table 50247)
4. Add Risk 1:
   - Risk Code: "ASBESTOS"
   - Description: "Asbestos found in building"
   - Probability: High
   - Impact: High
   - Mitigation: "Commission asbestos survey before construction"
5. Add Risk 2:
   - Risk Code: "PERMIT-DELAY"
   - Probability: Medium
   - Impact: High
6. Save

**Expected Result:**
- Risk analysis header and lines saved
- Risk matrix (Probability × Impact) correctly categorized

**BC Objects:** Table 50248 HBVG Risk Analysis Header, Table 50247 HBVG Risk Analysis Lines

---

## TC-307
**Module:** Risicoanalyse
**Process:** Risk Analysis from Template
**Type:** Positive

**Steps:**
1. Open **HBVG Risk Analysis Template Header** (Table 50250)
2. Create template with standard risk lines
3. Open new Risk Analysis
4. Apply template

**Expected Result:**
- Standard risk lines copied from template to analysis
- Template lines editable per project

**BC Objects:** Table 50250 HBVG RiskAnalysis Templ.Header, Table 50251 HBVG RiskAnalysis Templ.Lines

---

---

# AREA 23 — PROFIT TAKING (WINSTNEMING)

---

## TC-316
**Module:** Winstneming
**Process:** Create Profit Taking Proposal
**Type:** Positive

**Prerequisites:**
- Project with completed/sold units exists
- Revenue posted to project
- Costs fully posted

**Steps:**
1. Open **HBVG Prop. Prof. Taking Header** (Table 50180) → New
2. Fill:
   - Project No.: (select)
   - Proposal Date: today
   - Description: "Q1 2026 Profit Recognition"
3. Navigate to **HBVG Proposal Profit Taking** lines (Table 50182)
4. Add Line:
   - Construction No.: (sold unit)
   - Revenue: 280,000
   - Cost: 200,000
   - Profit: 80,000
5. Save

**Expected Result:**
- Proposal header and lines created
- Profit amount calculated
- Ready for approval and posting

---

## TC-317
**Module:** Winstneming
**Process:** Send Profit Taking for Approval
**Type:** Positive

**Prerequisites:**
- Profit Taking Proposal exists in Draft
- Approval workflow for Winstneming active

**Steps:**
1. Open Profit Taking Proposal
2. Run **"Send for Approval"**

**Expected Result:**
- Approval entry created
- Status = Pending Approval

---

## TC-318
**Module:** Winstneming
**Process:** Post Profit Taking
**Type:** Positive

**Prerequisites:**
- Profit Taking Proposal approved

**Steps:**
1. Open approved Proposal
2. Run **"Post Profit Taking"**

**Expected Result:**
- G/L Entry: Debit "Cost of Sold Units" 200,000, Credit "Completed Projects Asset" 200,000
- G/L Entry: Debit "Debtors" 280,000, Credit "Revenue from Sales" 280,000
- Net: Profit 80,000 recognized in P&L
- Proposal status = Posted

---

## TC-319
**Module:** Winstneming
**Process:** Profit Taking Report
**Type:** Positive

**Steps:**
1. Run Report 50180 **"HBVG Proposal Profit Taking I"**
2. Filter by project

**Expected Result:**
- Report shows all profit taking proposals
- Per unit: Revenue, Cost, Profit, Status
- Totals correctly summed

---

---

# AREA 24 — MANDATORY FIELDS (VERPLICHTE VELDEN)

---

## TC-331
**Module:** Verplichte Velden
**Process:** Mandatory Field Blocks Posting
**Type:** Positive (testing the block works)

**Prerequisites:**
- Mandatory field rule: Purchase Header, "HBVG Project No." = mandatory

**Steps:**
1. Create Purchase Invoice without filling HBVG Project No.
2. Try to post

**Expected Result:**
- Error: "Project No. is mandatory and must be filled"
- Posting blocked

---

## TC-332
**Module:** Verplichte Velden
**Process:** Mandatory Field — Bypassed When Exempt
**Type:** Positive

**Prerequisites:**
- Mandatory field rule exists for Project No.
- G/L Account configured as exempt (e.g., bank charges account — no project needed)

**Steps:**
1. Create Purchase Invoice with bank charges on exempt G/L Account
2. Leave Project No. blank
3. Post

**Expected Result:**
- Posting succeeds (exempt G/L Account bypasses mandatory field check)
- G/L Entry created without project dimension

---

---

# AREA 25 — CENTRAL ADMINISTRATION (CENTRAAL)

---

## TC-341
**Module:** Centraal
**Process:** Central Setup Period
**Type:** Positive

**Steps:**
1. Open **HBVG Central Setup Period** (Table 50189)
2. Define period: Jan 2026 - Dec 2026
3. Mark as Active for all companies
4. Save

**Expected Result:**
- Period active for all satellite companies
- Posting controls apply to this period across all companies

---

## TC-342
**Module:** Centraal
**Process:** Central Update — Push G/L Accounts to All Companies
**Type:** Positive

**Prerequisites:**
- New G/L Account created in Central company
- Central Update Setup configured for G/L Account table

**Steps:**
1. Open **HBVG Central Update Setup** (Table 50331)
2. Add update rule: Table = "G/L Account", Method = Insert/Modify
3. Run **"Execute Central Update"**

**Expected Result:**
- New G/L Account created in all satellite companies
- Central update log shows success per company
- No manual login to each company required

**BC Objects:** Table 50331 HBVG Central Update Setup, Table 50332 HBVG Central Update Tables, Table 50333 HBVG Central Update Fields

---

## TC-343
**Module:** Centraal
**Process:** Copy Company
**Type:** Positive

**Steps:**
1. Run Report 50100 **"HBVG Copy Company"**
2. Source: Central Company
3. Target: New satellite company
4. Select data to copy: Setup, G/L Accounts, Posting Groups, Dimensions

**Expected Result:**
- New satellite company populated with master data from central
- No. Series prefixed with company prefix
- Ready for operation without manual setup

---

---

# AREA 26 — CLOUD FILE STORAGE

---

## TC-351
**Module:** Cloud File Storage
**Process:** Configure Online Drive
**Type:** Positive

**Prerequisites:**
- Microsoft 365 / SharePoint / OneDrive configured
- OAuth 2.0 credentials available

**Steps:**
1. Open **HBVG Online Drive** (Table 50119)
2. Configure:
   - Drive Type: SharePoint / OneDrive
   - Root URL: (SharePoint site URL)
   - Folder Structure: (HBVG File Structure enum)
3. Run OAuth 2.0 authentication flow
4. Save

**Expected Result:**
- Drive configured
- Authentication token stored
- Connection test succeeds

---

## TC-352
**Module:** Cloud File Storage
**Process:** Attach Document to Project
**Type:** Positive

**Prerequisites:**
- Online Drive configured
- Project exists

**Steps:**
1. Open Project Card
2. Navigate to **Documents** FactBox
3. Upload file: "PermitApproval.pdf"
4. Assign Document Location = SharePoint

**Expected Result:**
- File uploaded to SharePoint
- HBVG Document Location record created with URL
- File link visible in project card
- HBVG Drive Item ID stored for reference

**BC Objects:** Table 50414 HBVG Document Location, Table 50116 HBVG Drive Item ID

---

## TC-353
**Module:** Cloud File Storage
**Process:** Retrieve Document
**Type:** Positive

**Steps:**
1. Open Project Card
2. Click document link

**Expected Result:**
- Document opens in browser / downloads
- Correct file retrieved from SharePoint

---

---

# AREA 27 — END-TO-END SCENARIOS

---

## TC-359
**Module:** Full E2E
**Process:** New Build Project — Complete Lifecycle
**Type:** Positive

This is the master test case covering the entire system from setup to profit recognition.

### Phase 1: Setup
1. Company initialized (TC-002)
2. Security groups configured (TC-004)
3. Users assigned (TC-005)
4. Dimension setup done (TC-008)

### Phase 2: Portfolio
5. Complex created: "Nieuw Lindenhof" — 50 new units (TC-011)
6. Construction numbers created for all 50 units (TC-256)

### Phase 3: Project
7. Project created: "Nieuwbouw Lindenhof 2026" (TC-026)
8. Project team assigned (TC-031)
9. Approval thresholds set (TC-032)

### Phase 4: Feasibility
10. Calculation created with 3 cost lines totaling €4,000,000 (TC-051 + TC-052)
11. Risk analysis created (TC-306)

### Phase 5: Budget
12. Budget Name created (TC-066)
13. Budget posts created: 4,000,000 total (TC-067)
14. Budget approved through workflow

### Phase 6: Planning
15. Project planning created with 5 phases (TC-096)
16. Planning sent for approval and approved (TC-097)

### Phase 7: Financing
17. Loan created: €3,000,000 at 3.0% (TC-191)
18. Loan received posted to BC (TC-193)
19. Interest schedule generated (TC-192)

### Phase 8: Procurement
20. Draft orders created per contractor (TC-111)
21. Draft orders approved (TC-115 → TC-160)
22. Draft orders converted to Purchase Orders (TC-114)
23. Obligations created automatically (TC-126)

### Phase 9: Construction
24. Milestone invoices received and posted (TC-127)
25. MMW (change orders) created and agreed (TC-276 → TC-278)
26. Monthly interest posted (TC-194)
27. Timesheets submitted and posted (TC-287 → TC-288)
28. Forecasts updated each quarter (TC-216 → TC-217)
29. Cash flow monitored (TC-231)

### Phase 10: Buyer Administration
30. Buyers assigned to units (TC-257)
31. Buyer payment terms created (TC-258)
32. Installment invoices generated (TC-176)
33. Buyers pay installments

### Phase 11: Completion
34. All obligations fully closed (TC-127 full close)
35. Final MMW agreed and posted (TC-278)
36. Provisions released to P&L (TC-243)

### Phase 12: Profit Recognition
37. Profit taking proposal created (TC-316)
38. Approved (TC-317)
39. Posted (TC-318)

**Expected Final State:**
- All obligations: Closed
- All PO amounts: Fully invoiced
- Budget vs Actual: Within approved variance
- All buyer installments: Paid
- Profit: Recognized in P&L
- Loan: Payments schedule active
- Project status: Completed

---

## TC-360
**Module:** Full E2E
**Process:** Renovation Project with Additional Budget
**Type:** Positive

**Scenario:** Mid-project asbestos discovery forces additional budget request.

1. Renovation project created and running
2. Construction starts — asbestos found
3. Additional Budget Request created: €60,000 (TC-086)
4. Approved (TC-088)
5. Budget updated
6. MMW created for asbestos removal (TC-276)
7. New contractor hired (Draft Order → PO → Obligation)
8. Invoice posted, obligation closed
9. Project completes over-budget but within approved total

**Expected Result:**
- Original budget: €900,000
- Additional budget: €60,000
- Total approved: €960,000
- Actual: Within €960,000
- All obligations closed
- Variance within acceptable range

---

## TC-361
**Module:** Full E2E
**Process:** Multi-Company Project
**Type:** Positive

**Scenario:** Project spans two companies — construction in Company A, financing in Company B.

1. Project created in Company A
2. Loan taken in Company B
3. Inter-company posting via Current Account (TC-131 positive scenario)
4. Purchase invoices in Company A → obligation tracking
5. Revenue recognized in Company A
6. Profit consolidated

**Expected Result:**
- Intercompany entries balanced
- No cross-company dimension errors
- Consolidated view in Central Administration

---

## TC-362
**Module:** Full E2E
**Process:** Tenant Move-In to Rent Collection
**Type:** Positive

1. Complex exists (TC-011)
2. Unit available (no tenant)
3. Tenant found via waiting list
4. Contract created (TC-014)
5. First rent invoice generated → posted to BC
6. Tenant pays via direct debit
7. Payment applied to invoice
8. Customer Ledger Entry closed
9. G/L: Bank +€750, Rental Income +€750

**Expected Result:**
- Complete rent cycle from contract to payment
- No open customer ledger entries after payment
- Revenue correctly posted with complex dimension

---

## TC-363
**Module:** Full E2E
**Process:** Year-End Closing
**Type:** Positive

1. All project invoices posted for the year
2. Provisions calculated and posted (TC-243)
3. Depreciation run on Fixed Assets (BC standard)
4. Interest accruals posted (TC-207)
5. Income statement closed (Report 50104 HBVG Close Income Statement)
6. Balance sheet carried forward
7. New year opened in all companies via Central Administration

**Expected Result:**
- Income statement accounts zeroed to equity
- Balance sheet accounts carried forward
- New period open in all companies
- Year-end report generated

---

---

# APPENDIX A — TEST DATA REQUIREMENTS

## Master Data Needed Before Running Tests

| Data Type | Minimum Required | Notes |
|---|---|---|
| G/L Accounts | 10+ | Rent income, construction costs, labor, interest, provisions, WIP |
| Customers | 5+ | 2 tenants, 2 buyers, 1 internal recharge customer |
| Vendors | 5+ | 3 contractors, 1 architect, 1 bank (for loan) |
| Dimensions | 2 | PROJECT, COMPLEX (dimension codes in BC) |
| Dimension Values | 10+ | Per project and complex |
| No. Series | 8+ | Projects, Calculations, Draft Orders, Loans, Contracts, Obligations, Forecasts, Budgets |
| Posting Groups | 3 | General, Project, Buyer |
| Security Groups | 2 | Project Manager, Director |
| Users | 4 | Admin, Project Manager, Director (approver), Financial Admin |
| BC Location | 1 | For complex |
| Approval Workflow | 3 | For Purchase Invoices, Planning, Forecasts |
| Project Stages | 3 | Design, Permits, Construction |
| Construction Types | 3 | Foundation, Structure, Installations |

---

# APPENDIX B — ERROR MESSAGES REFERENCE

| Error Message | Module | Trigger Condition |
|---|---|---|
| "Approval requests already exist" | Approvals | Sending approval when one already open |
| "Approver not found" | Approvals | No user configured with authority |
| "No amount setting available" | Approvals | Invoice amount exceeds all configured thresholds |
| "Based on settings, not possible to create approval entries for all lines" | Approvals | GoedkeuringVolledig = false |
| "When using extended approval module, delegation is not allowed" | Approvals | Delegate action used with extended approvals |
| "Amount to be invoiced has not been filled" | Facturatie | GBPrj type but Amount To Be Invoiced = 0 |
| "Relationship with [company] does not exist" | Obligations | Current Account Ratio missing |
| "You can only post to a G/L account with a current account" | Obligations | Wrong account type for current account posting |
| "An invalid current account dimension has been specified" | Obligations | Dimension blank on current account line |
| "Project is blocked" | Projects | Project Blocked = All Posting |
| "You cannot change Status because this Project has entries" | Projects | Status change attempted on project with GL entries |
| "Project cannot be deleted because Purchase Invoice entries exist" | Projects | Delete project with linked invoices |
| "You cannot change this field because related Analysis View Budget Entries exist" | Budget | Budget modification after Analysis View update |
| "Project No. is mandatory" | Verplichte Velden | Mandatory field rule violated |
| "Description must not be empty" | Vastgoedbeheer | Complex created without description |

---

# APPENDIX C — TEST EXECUTION ORDER

Run tests in this order to ensure dependencies are satisfied:

```
1.  TC-001 to TC-010    (Setup — must be first)
2.  TC-011 to TC-025    (Real Estate / Complex — needed for all other tests)
3.  TC-026 to TC-035    (Projects — needed for budget, planning, orders)
4.  TC-051 to TC-056    (Calculations)
5.  TC-066 to TC-072    (Budget — needed before obligations and orders)
6.  TC-096 to TC-099    (Planning)
7.  TC-111 to TC-115    (Draft Orders)
8.  TC-126 to TC-133    (Obligations — needs purchase orders)
9.  TC-146 to TC-155    (Full Purchase Flow)
10. TC-156 to TC-162    (Approvals — needs documents from above)
11. TC-176 to TC-180    (Invoicing — needs GL entries)
12. TC-086 to TC-089    (Additional Budget — needs budget from area 5)
13. TC-191 to TC-196    (Loans)
14. TC-206 to TC-208    (Interest)
15. TC-216 to TC-219    (Forecasts)
16. TC-231 to TC-233    (Cash Flow)
17. TC-241 to TC-244    (Provisions)
18. TC-256 to TC-261    (Buyer Admin — needs complex and project)
19. TC-276 to TC-279    (MMW — needs purchase order)
20. TC-286 to TC-289    (Time Registration)
21. TC-296 to TC-298    (Coverage Surveillance)
22. TC-306 to TC-307    (Risk Analysis)
23. TC-316 to TC-319    (Profit Taking — needs completed project)
24. TC-331 to TC-332    (Mandatory Fields)
25. TC-341 to TC-343    (Central Admin)
26. TC-351 to TC-353    (Cloud File Storage)
27. TC-359 to TC-363    (End-to-End — run last)
```

---

# APPENDIX D — KEY BC OBJECTS REFERENCE

| AL Object | ID | Type | Role in Tests |
|---|---|---|---|
| HBVG Setup | 50110 | Table | TC-001 to TC-010 |
| HBVG Complex | 50100 | Table | TC-011 to TC-017 |
| HBVG Contract | 50281 | Table | TC-014 to TC-016 |
| HBVG Contract Line | 50282 | Table | TC-014 |
| HBVG Budget Name | 50346 | Table | TC-066 |
| HBVG Budgetpost | 50347 | Table | TC-067 to TC-072 |
| HBVG Draft Order | 50315 | Table | TC-111 to TC-115 |
| HBVG Draft Order Line | 50316 | Table | TC-111 |
| HBVG Loan | 50307 | Table | TC-191 to TC-196 |
| HBVG Loan Interest | 50306 | Table | TC-192 |
| HBVG Project Forecast | 50209 | Table | TC-216 to TC-219 |
| HBVG Project Forecast Line | 50210 | Table | TC-216 |
| HBVG Cashflow Schedule | 50176 | Table | TC-231 |
| HBVG Cashflow Posts | 50177 | Table | TC-232 |
| HBVG Provision | 50270 | Table | TC-241 |
| HBVG General Costs | 50268 | Table | TC-243 |
| HBVG Construction No. | 50163 | Table | TC-256 to TC-261 |
| HBVG More-Less Work | 50111 | Table | TC-276 to TC-279 |
| HBVG Risk Analysis Header | 50248 | Table | TC-306 to TC-307 |
| HBVG Risk Analysis Lines | 50247 | Table | TC-306 |
| HBVG Prop. Prof. Taking Header | 50180 | Table | TC-316 to TC-319 |
| HBVG Proposal Profit Taking | 50182 | Table | TC-316 |
| HBVG Mandatory Fields Header | 50122 | Table | TC-331 to TC-332 |
| HBVG Central Update Setup | 50331 | Table | TC-342 |
| HBVG Document Location | 50414 | Table | TC-352 |
| HBVG Approval All Companies | 50132 | Table | TC-160 |
| HBVGApprovalsManagement | 50385 | Codeunit | TC-156 to TC-162 |
| HBVGFacturatie | 50188 | Codeunit | TC-176 to TC-180 |
| HBVGSubscrInkoopC90 | 50xxx | Codeunit | TC-127 to TC-132 |
| HBVGBeheerMgt | 50171 | Codeunit | TC-010, TC-342 |
| HBVGCompanyInitialize | 50100 | Codeunit | TC-002 to TC-003 |
| HBVGDimensionMgt | 50357 | Codeunit | TC-068, TC-071 |
| HBVGWorkflowEvents | 50396 | Codeunit | TC-087, TC-097, TC-115 |

---

*Document prepared for HB Vastgoed 365 — HB Software*
*Total Test Cases: 375*
*Coverage: All 26 functional modules from Setup to End-to-End*
