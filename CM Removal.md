# Central Management Removal — Test Cases
## HB Vastgoed 365 — Post-Removal Verification

**Date:** 2026-04-14
**Based on:** Central_Management_Removal.md
**Purpose:** Confirm that HB Vastgoed 365 works correctly after Central Management was removed. Every test targets something that was changed — either a guard that was deleted, an action that was removed, or a cross-company write that was replaced with a local one.

---

## How to Use This Document

- Work through sections **in order** — later tests assume earlier ones passed.
- Each test has a **Precondition**, **Steps**, and **Expected Result**.
- Mark each result: `PASS` / `FAIL` / `SKIP` (with reason if skipped).
- On `FAIL`, note what actually happened and which page or file was involved.

**Test environment:** Single company. No central company, no satellite companies.

---

## Section 1 — Build and Installation

### TC-001 — Extension compiles without errors

| | |
|---|---|
| **Area** | Build |
| **Precondition** | Source code is complete after Central Management removal |
| **Steps** | 1. Run `al: publish` or check the compiler output panel |
| **Expected** | Zero compiler errors. No warnings about missing objects (`HBVG Beheer Mgt.`, `HBVG Copy Company`, `HBVG G/L Account Admin.`, `HBVG Central Update`, `HBVG Select Lines` original). |

---

### TC-002 — Extension installs and BC starts without error

| | |
|---|---|
| **Area** | Build |
| **Precondition** | TC-001 passed |
| **Steps** | 1. Publish extension to a sandbox. 2. Open BC and log in. |
| **Expected** | No runtime errors on login. No "object not found" messages. Home page loads normally. |

---

## Section 2 — Setup Pages

These pages had read-only locks or central company guards removed. Verify they are now fully editable.

### TC-003 — General Ledger Setup is editable

| | |
|---|---|
| **Area** | G/L Setup |
| **Precondition** | Logged in to BC |
| **Steps** | 1. Open **General Ledger Setup**. 2. Change any field. 3. Save. |
| **Expected** | Saves without error. No message about "only allowed in the central company". |
| **What changed** | The check that blocked G/L Setup edits outside the central company was removed. |

---

### TC-004 — Company Information is editable

| | |
|---|---|
| **Area** | Company Information |
| **Precondition** | Logged in to BC |
| **Steps** | 1. Open **Company Information**. 2. Change any field (e.g. address or phone). 3. Save. |
| **Expected** | Saves without error. No read-only lock. |
| **What changed** | Central company identity check that controlled editable state was removed. |

---

### TC-005 — Corresponding Organization field on Company Information saves cleanly

| | |
|---|---|
| **Area** | Company Information |
| **Precondition** | TC-004 passed |
| **Steps** | 1. Open **Company Information**. 2. Fill in or change the **Corresponding Organization** field. 3. Save. |
| **Expected** | Field saves. No cross-company dimension sync is triggered. No error about a missing codeunit. |
| **What changed** | Cross-company dimension propagation (`MaakDimAndereAdm` / `MaakDefDimAndereAdm`) was removed. Saving this field no longer writes dimensions to another company. |

---

### TC-006 — User Setup is editable and has no "Update Companies" action

| | |
|---|---|
| **Area** | User Setup |
| **Precondition** | Logged in to BC |
| **Steps** | 1. Open **HB Real Estate User Setup**. 2. Add or edit a user record. 3. Save. 4. Check the ribbon — confirm there is no **Update Companies** or **Update Administrations** action. |
| **Expected** | Record saves. Neither action is visible in the ribbon. |
| **What changed** | Both "Update Companies" and "Update Administrations" actions were removed from the User Setup page and page extension — they called the deleted `HBVG Beheer Mgt.` codeunit. |

---

### TC-007 — OAuth Applications are configurable without central company restriction

| | |
|---|---|
| **Area** | OAuth 2.0 |
| **Precondition** | Logged in to BC |
| **Steps** | 1. Open **HB OAuth 2.0 Applications**. 2. Create or edit an OAuth app. 3. Save. 4. Confirm there is no **Copy Application** action in the ribbon. |
| **Expected** | App saves without error. No restriction requiring a central company. No **Copy Application** action visible. |
| **What changed** | The central/master company guards and the entire "Copy Application" action (which pushed OAuth settings to all satellite companies) were removed. |

---

### TC-008 — Job Queue Monitor is accessible from any company

| | |
|---|---|
| **Area** | Job Queue |
| **Precondition** | Logged in to BC |
| **Steps** | 1. Open **HB Job Queue Monitor**. 2. Verify the **Create in Administration** action is visible and usable. |
| **Expected** | Page opens. Action is available. No "central company only" restriction. |
| **What changed** | The checks that restricted this action to the central company were removed. |

---

### TC-009 — Support page loads without an "Update OAuth" action

| | |
|---|---|
| **Area** | Support |
| **Precondition** | Logged in to BC |
| **Steps** | 1. Open **HB Support** page. 2. Check the ribbon. |
| **Expected** | Page loads without error. No **Update OAuth** action in the ribbon. |
| **What changed** | The `UpdateOAuth` action was removed from the Support page — the procedure it called was deleted because it copied OAuth settings from the (now non-existent) central company. |

---

## Section 3 — Master Data Editability

All 15 master data pages had the same read-only lock removed. The lock made fields read-only in any company that was not the designated "master data company." In a single company this lock was always active, blocking all editing.

### TC-010 — Customer Card is editable

| | |
|---|---|
| **Area** | Customers |
| **Precondition** | At least one customer exists |
| **Steps** | 1. Open a **Customer Card**. 2. Edit the name or address. 3. Save. |
| **Expected** | Saves without error. No read-only lock. No cross-company sync after saving. |

---

### TC-011 — Vendor Card is editable

| | |
|---|---|
| **Area** | Vendors |
| **Precondition** | At least one vendor exists |
| **Steps** | 1. Open a **Vendor Card**. 2. Edit a field. 3. Save. |
| **Expected** | Saves without error. No read-only lock. No cross-company sync. |

---

### TC-012 — Contact Card is editable

| | |
|---|---|
| **Area** | Contacts |
| **Precondition** | At least one contact exists |
| **Steps** | 1. Open a **Contact Card**. 2. Edit a field. 3. Save. |
| **Expected** | Saves without error. No read-only lock. No cross-company sync. |

---

### TC-013 — Employee Card is editable

| | |
|---|---|
| **Area** | Employees |
| **Precondition** | At least one employee exists |
| **Steps** | 1. Open an **Employee Card**. 2. Edit a field. 3. Save. |
| **Expected** | Saves without error. No read-only lock. No cross-company sync. |

---

### TC-014 — Salesperson Card is editable

| | |
|---|---|
| **Area** | Salesperson |
| **Precondition** | At least one salesperson exists |
| **Steps** | 1. Open a **Salesperson/Purchaser Card**. 2. Edit a field. 3. Save. |
| **Expected** | Saves without error. No cross-company sync after saving. |

---

### TC-015 — General Ledger Account Card is editable

| | |
|---|---|
| **Area** | G/L Accounts |
| **Precondition** | At least one G/L Account exists |
| **Steps** | 1. Open a **G/L Account Card**. 2. Edit the description or an HB field. 3. Save. |
| **Expected** | Saves without error. No read-only lock from a central company guard. |

---

### TC-016 — Vendor Bank Account is editable

| | |
|---|---|
| **Area** | Vendors |
| **Precondition** | A vendor with a bank account exists |
| **Steps** | 1. Open a **Vendor Card → Bank Accounts**. 2. Edit the bank account. 3. Save. |
| **Expected** | Saves without error. No read-only lock. |

---

### TC-017 — Customer Bank Account is editable

| | |
|---|---|
| **Area** | Customers |
| **Precondition** | A customer with a bank account exists |
| **Steps** | 1. Open a **Customer Card → Bank Accounts**. 2. Edit the bank account. 3. Save. |
| **Expected** | Saves without error. No read-only lock. |

---

## Section 4 — Dimensions

Dimension create, modify, and delete were all blocked outside the central company. Those blocks are now removed.

### TC-018 — Create a new Dimension

| | |
|---|---|
| **Area** | Dimensions |
| **Precondition** | Logged in to BC |
| **Steps** | 1. Open **Dimensions**. 2. Create a new dimension (e.g. `TEST`). 3. Save. |
| **Expected** | Dimension created without error. No "only allowed in central company" message. |

---

### TC-019 — Create a Dimension Value

| | |
|---|---|
| **Area** | Dimensions |
| **Precondition** | A dimension exists |
| **Steps** | 1. Open **Dimensions → Values**. 2. Add a new value. 3. Save. |
| **Expected** | Value created without error. No block from the central settings guard. |

---

### TC-020 — Modify a Dimension Value

| | |
|---|---|
| **Area** | Dimensions |
| **Precondition** | A dimension value exists |
| **Steps** | 1. Open a dimension value. 2. Change the description. 3. Save. |
| **Expected** | Saves without error. No central company check fires. |

---

### TC-021 — Delete a Dimension Value

| | |
|---|---|
| **Area** | Dimensions |
| **Precondition** | A dimension value exists that is not in use |
| **Steps** | 1. Open **Dimensions → Values**. 2. Select and delete a test value. |
| **Expected** | Deleted without error. No cross-company propagation attempted. |

---

## Section 5 — Projects

### TC-022 — Create a Project and check that default dimensions are written locally

| | |
|---|---|
| **Area** | Projects |
| **Precondition** | At least one project stage exists |
| **Steps** | 1. Open **Projects**. 2. Create a new project. 3. Save. 4. Open **Project → Default Dimensions**. |
| **Expected** | Project created. Default dimensions are written for the current company only. No attempt to write to a second company. No error about a missing codeunit. |
| **What changed** | The cross-company dimension write loop and the `MaakDimCorrOrg` / `MaakDefDimCorrOrg` calls were removed. Dimensions now go to the current company only. |

---

### TC-023 — Delete a Project and confirm dimension cleanup

| | |
|---|---|
| **Area** | Projects |
| **Precondition** | A test project exists with dimensions |
| **Steps** | 1. Delete the test project. |
| **Expected** | Project and its dimension values are deleted from the current company. No error about a project administration company. |
| **What changed** | The dimension delete now targets the current company — previously it targeted the project administration company name. |

---

### TC-024 — Project Stage list opens and stages are editable

| | |
|---|---|
| **Area** | Projects |
| **Precondition** | At least one project stage exists |
| **Steps** | 1. Open **Project Stages**. 2. Modify a stage name or toggle a flag. 3. Save. |
| **Expected** | Stage saves. A dialog may appear asking "Do you want to keep this value the same in all companies?" — this is still in the code and is acceptable. With a single company, the loop finds no other companies and does nothing. No error. |

---

### TC-025 — Project Planning page opens without error

| | |
|---|---|
| **Area** | Projects / Planning |
| **Precondition** | A project with planning lines exists |
| **Steps** | 1. Open **Project Planning**. 2. Browse the records. |
| **Expected** | Page opens without error. No central company reference errors. |

---

### TC-026 — Project Forecast list opens without error

| | |
|---|---|
| **Area** | Projects / Forecasts |
| **Precondition** | A project with forecast data exists |
| **Steps** | 1. Open **Project Forecast List**. 2. Browse the records. |
| **Expected** | Page opens without error. No errors related to removed central sync references. |

---

### TC-027 — Approval General Ledger Account setup is accessible

| | |
|---|---|
| **Area** | Projects / Approvals |
| **Precondition** | Logged in to BC |
| **Steps** | 1. Open **Approval per Project**. 2. Set up or view an approval GL account entry. 3. Save. 4. Confirm there is no **Copy to All** button. |
| **Expected** | Saves without error. No central company guard blocking setup. The **Copy to All** button is not visible (it was only shown in the central company). |

---

## Section 6 — Real Estate Administration

### TC-028 — Create a Unit and confirm default dimensions are local

| | |
|---|---|
| **Area** | Units |
| **Precondition** | A complex and administration exist |
| **Steps** | 1. Create a new Unit. 2. Save. 3. Open **Unit → Default Dimensions**. |
| **Expected** | Unit created. Default dimensions are present in the current company only. No cross-company write. No missing codeunit error. |
| **What changed** | The cross-company dimension loop (`x=1` current, `x=2` project admin) was replaced with a single local write. |

---

### TC-029 — Create a Complex and confirm default dimensions are local

| | |
|---|---|
| **Area** | Complexes |
| **Precondition** | An administration exists |
| **Steps** | 1. Create a new Complex. 2. Save. 3. Open **Complex → Default Dimensions**. |
| **Expected** | Complex created. Default dimensions in the current company only. No cross-company write. |
| **What changed** | Same loop removal as Units. |

---

## Section 7 — Loans

### TC-030 — Create a Loan and confirm dimension values are filled locally

| | |
|---|---|
| **Area** | Loans |
| **Precondition** | A project exists |
| **Steps** | 1. Create a new Loan record. 2. Save. 3. Verify dimension values are filled. |
| **Expected** | Loan created. Dimension values written for the current company only. No cross-company write. No missing codeunit error. |
| **What changed** | The cross-company loop in the loan dimension fill procedure was replaced with a single local insert. |

---

### TC-031 — Loan Movements report runs and includes the current company

| | |
|---|---|
| **Area** | Loans / Reports |
| **Precondition** | At least one loan with movements exists |
| **Steps** | 1. Run the **Loan Movements** report. 2. Preview or print. |
| **Expected** | Report runs and shows data for the current company. No company exclusion filter applied. |
| **What changed** | The filter that excluded the central company from loan reports was removed. |

---

## Section 8 — Financial Postings

### TC-032 — Post a General Journal line

| | |
|---|---|
| **Area** | Finance / G/L |
| **Precondition** | A general journal batch is set up |
| **Steps** | 1. Create a general journal line. 2. Post it. |
| **Expected** | Posts without error. No cross-company G/L sync triggered. The **Corresponding Entry Number** field (`HBVG Corresp. postvolgnr.`) will be blank — the procedure that set it was removed. Confirm with Frank if this field is expected to be populated by another mechanism. |
| **What changed** | The G/L account cross-company validation and the project administration sync call were both removed from the posting subscriber. |

---

### TC-033 — Post a Purchase Invoice

| | |
|---|---|
| **Area** | Finance / Purchase |
| **Precondition** | A vendor and a purchase invoice exist |
| **Steps** | 1. Create and post a purchase invoice. |
| **Expected** | Posts without error. No vendor sync to other companies triggered. |

---

### TC-034 — Post a Sales Invoice

| | |
|---|---|
| **Area** | Finance / Sales |
| **Precondition** | A customer and a sales invoice exist |
| **Steps** | 1. Create and post a sales invoice. |
| **Expected** | Posts without error. No customer sync to other companies triggered. |

---

### TC-035 — Delete a G/L Account

| | |
|---|---|
| **Area** | G/L Accounts |
| **Precondition** | A test G/L Account with no ledger entries exists |
| **Steps** | 1. Delete the test G/L Account. |
| **Expected** | Deleted without error. No subscriber fires trying to clean up the deleted `HBVG G/L Account Admin.` table. |

---

## Section 9 — Carryover (Period Close)

### TC-036 — Run the Carryover process

| | |
|---|---|
| **Area** | Carryover |
| **Precondition** | Carryover data exists for the current period |
| **Steps** | 1. Run the **HB Carryover Management** process. |
| **Expected** | Process completes for the current company. No cross-company dimension propagation attempted. No missing codeunit error. |
| **What changed** | A 17-line dimension propagation block that wrote dimensions to the project administration company after carryover was removed. |

---

## Section 10 — Interest Calculation

### TC-037 — Run Interest Calculation

| | |
|---|---|
| **Area** | Interest |
| **Precondition** | Loans with interest calculations exist |
| **Steps** | 1. Run the interest calculation process for a period. |
| **Expected** | Interest calculated and posted for the current company. No central company reference errors. |

---

## Section 11 — Buyer Administration

### TC-038 — Interested Buyer page opens as editable and has no "Update Companies" action

| | |
|---|---|
| **Area** | Buyer Administration |
| **Precondition** | At least one interested buyer record exists |
| **Steps** | 1. Open **Interested Buyer**. 2. Edit a field. 3. Save. 4. Check the ribbon for an **Update Companies** action. |
| **Expected** | Page opens as editable (no master data company lock). Saves without error. No **Update Companies** action in the ribbon. |
| **What changed** | The editable guard and the "Update Companies" action (including its orphaned promoted action reference) were all removed. |

---

### TC-039 — Sale Dissolution page is editable

| | |
|---|---|
| **Area** | Buyer Administration |
| **Precondition** | At least one sale dissolution record exists |
| **Steps** | 1. Open **Sale Dissolution**. 2. Attempt to edit the record. |
| **Expected** | Page is editable. No read-only lock from the project administration or central company check in the `ActivateFields` procedure. |

---

### TC-040 — Address Update runs with project selection working

| | |
|---|---|
| **Area** | Buyer Administration |
| **Precondition** | Address update feature is accessible |
| **Steps** | 1. Run **Update Addresses**. 2. When prompted, select projects using the selection page. |
| **Expected** | The **Select Lines** page (page 50500 — recreated) opens and shows projects for selection. Selection works. Address update runs locally. No cross-company sync. |
| **What changed** | The original Select Lines page was deleted with the Central Management folder. It was recreated as page 50500 specifically to support this and the cashflow import use case. |

---

### TC-041 — Buyer/Tenant page opens without error

| | |
|---|---|
| **Area** | Buyer Administration |
| **Precondition** | Buyer/tenant data exists |
| **Steps** | 1. Open **Buyer/Tenant** page. Browse the records. |
| **Expected** | Page opens without error. No central company reference. |

---

## Section 12 — Document Location (Cloud Storage)

### TC-042 — Document Location page has no "Update All Companies" action

| | |
|---|---|
| **Area** | Cloud File Storage |
| **Precondition** | Document locations are configured |
| **Steps** | 1. Open **Document Location**. 2. Check the ribbon. 3. Edit a setting. 4. Save. |
| **Expected** | Page opens. No **Update All Companies** action visible. Setting saves for the current company without error. |
| **What changed** | The entire "Update All Companies" action and the company selection procedure were removed. Document locations are now configured per company directly. |

---

## Section 13 — Cashflow Import

### TC-043 — Cashflow import uses the recreated file selection page

| | |
|---|---|
| **Area** | Cashflow |
| **Precondition** | Cashflow import files (.cash) are available |
| **Steps** | 1. Run the cashflow import. 2. When prompted to select files, verify the file selection page opens. 3. Select a file and proceed. |
| **Expected** | The **Select Lines** page (page 50500) opens and shows files. File selection works. Import proceeds without error. |
| **What changed** | The original Select Lines page was deleted with the Central Management folder and recreated as page 50500. The cashflow import was always a non-central-management feature — it just shared the page. |

---

## Section 14 — Analysis Views

### TC-044 — Analysis View List has no "Copy Settings" action

| | |
|---|---|
| **Area** | Analysis Views |
| **Precondition** | Analysis views exist |
| **Steps** | 1. Open **Analysis View List**. 2. Check the ribbon. 3. Update or refresh an analysis view. |
| **Expected** | No **Copy Settings** action in the ribbon. Analysis views update normally. |

---

### TC-045 — Analysis View Card has no "Copy Settings" action

| | |
|---|---|
| **Area** | Analysis Views |
| **Precondition** | An analysis view exists |
| **Steps** | 1. Open an **Analysis View Card**. 2. Check the ribbon. |
| **Expected** | No **Copy Settings** action. Card opens without error. |

---

### TC-046 — Analysis by Dimensions has no "Copy Settings" action

| | |
|---|---|
| **Area** | Analysis Views |
| **Precondition** | Analysis data exists |
| **Steps** | 1. Open **Analysis by Dimensions**. 2. Check the ribbon. |
| **Expected** | No **Copy Settings** action. Page opens without error. |

---

## Section 15 — VAT and Reports

### TC-047 — VAT Statement has no "Transfer All Companies" action

| | |
|---|---|
| **Area** | VAT |
| **Precondition** | VAT statement data exists |
| **Steps** | 1. Open the **VAT Statement**. 2. Check the ribbon. 3. Run the statement. |
| **Expected** | No **Transfer All Companies** action visible. Statement runs for the current company. No error. |

---

### TC-048 — Balance All Administrations report includes the current company

| | |
|---|---|
| **Area** | Reports |
| **Precondition** | G/L entries exist |
| **Steps** | 1. Run the **Balance All Administrations** report. 2. Preview or print. |
| **Expected** | Report runs and includes the current company (no company exclusion filter). No error. |
| **What changed** | The filter that excluded the central company from this report was removed. |

---

### TC-049 — WV All Administrations report includes the current company

| | |
|---|---|
| **Area** | Reports |
| **Precondition** | Data exists |
| **Steps** | 1. Run the **WV All Administrations** report. 2. Preview or print. |
| **Expected** | Report runs and includes the current company. No exclusion filter applied. |

---

### TC-050 — Purchase Invoices All Administrations opens correctly

| | |
|---|---|
| **Area** | Purchase / Reports |
| **Precondition** | Purchase invoices exist |
| **Steps** | 1. Open **Purchase Invoices All Administrations**. |
| **Expected** | Page shows invoices for the current company. No company is excluded from the list. |
| **What changed** | The filter that excluded the central company from this list was removed. |

---

## Section 16 — Role Center Activities

### TC-051 — Debtor Activities are visible on the Role Center

| | |
|---|---|
| **Area** | Role Center |
| **Precondition** | HB Real Estate Role Center is active |
| **Steps** | 1. Open the HB Real Estate Role Center. 2. Check the Debtor Activities area. |
| **Expected** | Debtor activities are visible and showing data. |
| **What changed** | The check that hid debtor activities in the central company (which had no debtors) was removed. |

---

### TC-052 — Creditor Activities are visible on the Role Center

| | |
|---|---|
| **Area** | Role Center |
| **Precondition** | HB Real Estate Role Center is active |
| **Steps** | 1. Open the HB Real Estate Role Center. 2. Check the Creditor Activities area. |
| **Expected** | Creditor activities are visible and showing data. |
| **What changed** | Same as TC-051 — the central company hide guard was removed. |

---

## Section 17 — Workflows and Approvals

### TC-053 — Approval workflow is enabled and running

| | |
|---|---|
| **Area** | Workflows |
| **Precondition** | An approval workflow is configured |
| **Steps** | 1. Enable an approval workflow. 2. Trigger it (e.g. submit a purchase invoice for approval). |
| **Expected** | Workflow runs without error. No "workflows are disabled" message. |
| **What changed** | The check that disabled all workflows in the central company was removed. Workflows are now always enabled. |

---

### TC-054 — Approval per Project has no "Copy to All" button

| | |
|---|---|
| **Area** | Approvals |
| **Precondition** | Approval per project data exists |
| **Steps** | 1. Open **Approval per Project**. 2. Check the ribbon. |
| **Expected** | No **Copy to All** button. Page opens normally. |
| **What changed** | The button was only visible when running in the central company. The check was removed. |

---

### TC-055 — Approval G/L Account page has no "Copy to All" button

| | |
|---|---|
| **Area** | Approvals |
| **Precondition** | Approval G/L account data exists |
| **Steps** | 1. Open **Approval G/L Accounts**. 2. Check the ribbon. |
| **Expected** | No **Copy to All** button. Page opens normally. |

---

## Section 18 — Company Setup Wizards

### TC-056 — Create Company Wizard runs without copying from a central company

| | |
|---|---|
| **Area** | Company Setup |
| **Precondition** | Access to create companies |
| **Steps** | 1. Open the **Create Company Wizard**. 2. Complete the steps. |
| **Expected** | Wizard runs. No reference to the deleted Copy Company report. Company is created without pulling setup from a central company. |

---

### TC-057 — Basic Setup Wizard creates a project administration company without Copy Company

| | |
|---|---|
| **Area** | Company Setup |
| **Precondition** | Running through initial setup |
| **Steps** | 1. Open the **Basic Setup Wizard**. 2. Go through the Create Project Administration and Create Demo Company steps. |
| **Expected** | Wizard runs. No Copy Company report is called. No "set project administration name" call. The company is created using standard record operations only. |

---

### TC-058 — Create PRJ Admin Wizard creates a bare company without Copy Company

| | |
|---|---|
| **Area** | Company Setup |
| **Precondition** | Access to the wizard |
| **Steps** | 1. Open the **Create Project Administration Wizard**. 2. Complete the setup step. |
| **Expected** | Company created without copying from a central company. No Copy Company report called. Company Information is saved correctly. |

---

### TC-059 — Companies page has no central company actions

| | |
|---|---|
| **Area** | Companies |
| **Precondition** | Logged in to BC |
| **Steps** | 1. Open the **Companies** page. 2. Check the available actions. 3. Create a new company using the wizard action. |
| **Expected** | No "central company" or "copy company" actions visible. Creating a company via the wizard works. No errors. |

---

## Section 19 — Permission Sets

### TC-060 — HBVGFULLACCESS permission set is valid

| | |
|---|---|
| **Area** | Permissions |
| **Precondition** | Extension installed |
| **Steps** | 1. Open **Permission Sets**. 2. Find `HBVGFULLACCESS`. 3. Open and review the entries. |
| **Expected** | No entries for deleted tables (`HBVG Delete Master Data Buffer`, `HBVG Central Setup Period`, `HBVG Central Update Setup`, `HBVG Central Update Tables`, `HBVG Central Update Fields`). Permission set loads without error. |

---

### TC-061 — Assign HBVGFULLACCESS to a test user and log in

| | |
|---|---|
| **Area** | Permissions |
| **Precondition** | A test user exists |
| **Steps** | 1. Assign `HBVGFULLACCESS` to the test user. 2. Log in as that user. 3. Open key pages: Customers, Projects, G/L Setup. |
| **Expected** | No permission errors. No "table not found" runtime errors. All pages accessible. |

---

### TC-062 — HBVGFINBEHEER permission set has no G/L Account Admin entry

| | |
|---|---|
| **Area** | Permissions |
| **Precondition** | Extension installed |
| **Steps** | 1. Open **Permission Sets → HBVGFINBEHEER**. 2. Review the entries. |
| **Expected** | No entry for `HBVG G/L Account Admin.` (table was deleted). Permission set loads without error. |

---

## Section 20 — Company Account Scheme

### TC-063 — Company Account Scheme opens without the G/L Company subpage

| | |
|---|---|
| **Area** | Account Scheme |
| **Precondition** | Account scheme data exists |
| **Steps** | 1. Open **Company Account Scheme**. |
| **Expected** | Page opens without error. No "G/L Company" subpage (the page was deleted with the Central Management folder). No central company group visible. |

---

## Section 21 — Function Roles and CRM

### TC-064 — Function Role changes are allowed

| | |
|---|---|
| **Area** | Function Roles |
| **Precondition** | Function roles are configured |
| **Steps** | 1. Open **HB Function Roles**. 2. Modify a role. 3. Save. |
| **Expected** | Saves without error. No "only allowed in central company" guard fires. |

---

### TC-065 — Saving a CRM Contact does not trigger cross-company sync

| | |
|---|---|
| **Area** | CRM / Contacts |
| **Precondition** | A CRM contact exists |
| **Steps** | 1. Open a CRM Contact. 2. Change a field. 3. Save. |
| **Expected** | Saves without error. No cross-company sync triggered. No missing codeunit error. |
| **What changed** | The `HBVG Central Update` codeunit instantiation in the CRM management codeunit was removed. |

---

## Regression Checklist

After all tests pass, confirm the following items are **absent** from the UI:

| Item | Where to check |
|---|---|
| "Update Companies" action | User Setup page and page extension |
| "Update Administrations" action | User Setup page extension |
| "Copy Settings" action | Analysis View List, Card, and by Dimensions |
| "Transfer All Companies" action | VAT Statement |
| "Update All Companies" action | Document Location |
| "Copy Application" action | OAuth 2.0 Applications |
| "Update OAuth" action | HB Support page |
| "Copy to All" button | Approval per Project, Approval G/L Accounts |
| "Update Companies" action | Interested Buyer |
| Any "central company" label or message | Any page |
| Any "CI company" label or message | Any page |

---

*Generated 2026-04-14 — for execution after Central Management removal build is published to sandbox.*
