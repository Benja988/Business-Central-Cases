# Central Management Removal Log
## HB Vastgoed 365 — DBBE Migration

**Date:** 2026-04-01 → 2026-04-02 (updated after compiler error pass)
**Requested by:** Frank (HB Developer)
**Reason summary:** Central Management adds multi-company complexity that the target customer does not use. Removing it makes the solution smaller, testable in a single company, and removes the biggest risk area before DBBE migration.

---

## Background

Central Management is a layer in HB Vastgoed that allows one "Central Company" (Centraal Beheer) to act as master — pushing setup, master data, dimensions, vendors, customers, G/L accounts, and posting configuration down to satellite companies automatically via the Change Log.

The core mechanism is `HBVG Single Instance Mgt.` (a `SingleInstance = true` codeunit) which stores the central company name (`CBName`) for the entire session. Every operation that touches setup data runs `CheckCB()` to verify it's running in the central company, and blocks if not.

**Why removed:**
1. The target customer runs a single administration — no satellites
2. The layer fires on every save (Change Log subscriber) even when unused
3. It requires multi-company test scaffolding for every test case
4. It is the single largest risk factor in the migration to DBBE
5. Frank's recommendation: break the config relation without deleting code first — but for DBBE we remove it fully

---

## FILES DELETED ENTIRELY

### Vastgoed Centraal — Full Folder Removed (18 files)

| File | Object Type | Reason for Deletion |
|---|---|---|
| `Vastgoed Centraal/Codeunits/HBVGCentralUpdate.Codeunit.al` | Codeunit 50332 | Main engine: pushes data from central company to all satellites via ChangeCompany(). Subscribes to Change Log. Syncs customers, vendors, contacts, bank accounts, dimensions, subcontracting, SWIFT codes. Entirely unused in single-company setup. |
| `Vastgoed Centraal/Codeunits/HBVGCentralByjobque.Codeunit.al` | Codeunit | Job Queue wrapper for `HBVGCentralUpdate`. Schedules background central sync jobs. Serves no purpose without multiple companies. |
| `Vastgoed Centraal/Codeunits/HBVGDeleteMasterData.Codeunit.al` | Codeunit | Deletes master data (customers, vendors, contacts) across all satellite companies when deleted in central. Cross-company deletion logic with no use in single company. |
| `Vastgoed Centraal/Codeunits/HBVGDeleteMasterDataCheck.Codeunit.al` | Codeunit | Pre-deletion validation for cross-company master data removal. Dependent on `HBVGDeleteMasterData`. No standalone use. |
| `Vastgoed Centraal/Tables/HBVGCentralSetupPeriod.Table.al` | Table 50189 | Defines posting periods controlled from central company and pushed to all satellites. Replaced by standard BC posting date controls in single-company setup. |
| `Vastgoed Centraal/Tables/HBVGCentralUpdateSetup.Table.al` | Table 50331 | Configures which BC tables (50+) are synced from central to satellites. Contains `CheckCB()` on every Insert/Modify/Delete/Rename trigger. No meaning without multiple companies. |
| `Vastgoed Centraal/Tables/HBVGCentralUpdateTables.Table.al` | Table 50332 | Table-level sync configuration (update method per table). Entirely dependent on Central Update Setup. |
| `Vastgoed Centraal/Tables/HBVGCentralUpdateFields.Table.al` | Table 50333 | Field-level sync configuration (which fields to push, which to skip). Dependent on Central Update Tables. |
| `Vastgoed Centraal/Tables/HBVGDeleteMasterDataBuffer.Table.al` | Table 50103 | Buffer table used during cross-company master data deletion. Cleaned up in `HBVGSubscribersAlgemeen` on company delete. No use in single company. |
| `Vastgoed Centraal/Tables/HBVGGLAccountAdmin.Table.al` | Table | G/L account administration records created in central and pushed to satellites. Setup logic lives in `HBVGCompanyInitialize`. Removed with that codeunit. |
| `Vastgoed Centraal/Pages/HBVGCentralUpdateList.Page.al` | Page | List view of `HBVG Central Update Setup` table. No underlying table — deleted. |
| `Vastgoed Centraal/Pages/HBVGCentralUpdateSetup.Page.al` | Page | Card page for central update setup. No underlying table — deleted. |
| `Vastgoed Centraal/Pages/HBVGCentralUpdateTables.Page.al` | Page | Page for `HBVG Central Update Tables`. No underlying table — deleted. |
| `Vastgoed Centraal/Pages/HBVGCentralUpdateFields.Page.al` | Page | Page for `HBVG Central Update Fields`. No underlying table — deleted. |
| `Vastgoed Centraal/Pages/HBVGCentralAccountCard.Page.al` | Page | G/L Account card view restricted to central company context. No central company — no meaning. |
| `Vastgoed Centraal/Pages/HBVGCentralAccountScheme.Page.al` | Page | Account scheme managed from central company. Replaced by standard BC account schemes per company. |
| `Vastgoed Centraal/Pages/HBVGGLCompany.Page.al` | Page | G/L balances per company view for central management reporting. Multi-company view not needed. |
| `Vastgoed Centraal/Pages/HBVGSelectLines.Page.al` | Page | Line-selection helper page used in central update operations. No parent operation — deleted. |

---

### Other Files Deleted Entirely (6 files)

| File | Object Type | Reason for Deletion |
|---|---|---|
| `src/Codeunits/HBVGBeheerMgt.Codeunit.al` | Codeunit | The primary Central Management business logic codeunit. Contains: `BlokkeerBuitenCentraleInstellingen()` (blocks all changes outside central company), `UpdateNoSeries()` (syncs number series across companies), `UpdatePostingAllowed()` (opens/closes posting in all companies simultaneously), `UpdateGLEntryPRJ()` (updates G/L entries across companies). Entirely multi-company orchestration — no role in single company. |
| `src/Codeunits/HBVGCompanyInitialize.Codeunit.al` | Codeunit 50100 | Initializes a satellite company from the central company using `ChangeCompany()`. Copies No. Series (with company prefix), G/L Account admin records, default dimensions, posting setup. The entire procedure `SetupVanuitCentraalBeheer()` is meaningless without a central company. No local initialization logic present. |
| `src/Enums/HBVGCentralUpdateMethod.Enum.al` | Enum 50136 | Defines update methods (Insert, Modify, Delete, etc.) for the Central Update sync engine. Referenced only by `HBVGCentralUpdateTables` which is also deleted. |
| `src/Reports/ProcessOnly/HBVGCopyCompany.Report.al` | Report 50100 | Copies a full company setup from the central company to a new satellite. Calls `ChangeCompany()` to read central G/L accounts, dimensions, No. Series. Entire purpose is multi-company setup replication. |
| `src/Reports/ProcessOnly/HBVGUpdateGeneralLedgerPRJ.Report.al` | Report 50215 | Calls `HBVGBeheerMgt.UpdateGLEntryPRJ()` and immediately errors if not in the central company (`GetCIName()` check on line 31). Both its dependency and its guard are removed. |
| `Vastgoed Projecten/Codeunits/HBVGUpdateCentrPrjAdm.Codeunit.al` | Codeunit | "Update Central Project Administration" — synchronizes project administration data from central to satellite companies. Entire purpose is central management propagation. |

---

## FILES MODIFIED PARTIALLY

The following files retain their core functionality but have had Central Management specific code removed.

### src/Codeunits/

| File | What Was Removed | Why |
|---|---|---|
| `HBVGSingleInstanceMgt.Codeunit.al` | Variables: `CBName`, `MasterDataBedrijf`, `BijWerkenViaJobQueUit`. Procedures: `GetCIName()`, `SetCIName()`, `ZetMasterDataBedrijf()`, `GetMasterDataBedrijf()`, `ZetBijwerkenViaJobQueue()`, `GetBijwerkenViaJobQueue()` | These are exclusively the "who is the central company?" session state variables. Removing them removes the runtime identity of central management from the session. The codeunit retains all project, forecast, budget filter state and module status methods. |
| `HBVGSubscribersAlgemeen.Codeunit.al` | Permission on `HBVG Central Update Fields`. `SetCIName()` call on login. `ZetMasterDataBedrijf()` call. All `GetCIName()` if-blocks routing to central-specific behaviour. `HBVG Delete Master Data Buffer` cleanup on company delete. | This is the general subscriber codeunit — it fires on BC login events (where `SetCIName` was called) and on company delete (where the buffer was cleaned). Removing central code leaves standard subscriber behaviour intact. |
| `HBVGSubscribersFinancieel.Codeunit.al` | 8 `if CompanyName() = GetCIName()` blocks on lines 300, 345, 547, 741, 751, 759, 767, 775. These forked G/L posting logic to trigger central sync after financial postings. | Financial posting should behave the same regardless of company. The fork was only needed to propagate changes to satellites. |
| `HBVGSubscribersInkoop.Codeunit.al` | 2 `HBVG Central Update` codeunit instantiations and calls (lines 210, 251) after vendor/purchase record changes. | Purchase subscribers no longer need to trigger cross-company vendor sync. |
| `HBVGSubscribersVerkoop.Codeunit.al` | 2 `HBVG Central Update` codeunit instantiations and calls (lines 185, 224) after customer/sales record changes. | Sales subscribers no longer need to trigger cross-company customer sync. |
| `HBVGCRMManagement.Codeunit.al` | `loHBVCentraalbijwerken: Codeunit "HBVG Central Update"` declaration and its invocation (line 109). | CRM contact changes no longer need to propagate to satellite companies. |
| `HBVGDefaultFunctions.Codeunit.al` | `GetMasterDataBedrijf()` guard in master data protection procedures (lines 245, 247, 252). The guard blocked master data changes in non-master-data companies. | In a single company there is no master data company distinction. The procedures become pass-through. |
| `HBVGDimensionMgt.Codeunit.al` | 4 `GetCIName()` blocks (lines 99–100, 708, 724, 740, 774) that restricted dimension create/modify/delete to the central company only. | Dimensions can now be managed directly in the single company without the central company guard. |
| `HBVGWorkflowEnableMgt.Codeunit.al` | `if CompanyName() = GetCIName()` check (line 11) that disabled workflows in the central company. | Central company had workflows disabled so it didn't process its own approval documents. In a single company, workflows should always be enabled. |
| `HBVGFunctierolMgt.Codeunit.al` | `GetCIName()` guard (line 22) blocking function/role changes in non-central companies. | Function roles now manageable directly. |
| `HBVGUserSetupMgt.Codeunit.al` | `GetCIName()` check (line 159) routing user setup logic based on central company identity. | Single-company user setup needs no company routing. |
| `HBVGApprovalAllCompanyMgt.Codeunit.al` | `GetCIName()` check (lines 96–97) that errored if approval was attempted from a non-central company. | Multi-company approval routing removed. Single-company approvals use standard BC workflow. |
| `HBVGOverhevelmanagement.Codeunit.al` | `DVBeheerMgt: Codeunit "HBVG Beheer Mgt."` var from `ProcessData()`; entire dimension propagation block: `Clear(DVBeheerMgt)`, `MaakDimAndereAdm()` calls for DimensionValue and DefaultDimension, and `MaakDefDimAndereAdm()` loop (17 lines total) that wrote dimensions to the PRJ admin company after carryover. | `HBVG Beheer Mgt.` codeunit deleted. Carryover now runs within the single company only — no cross-company dimension propagation. |

### src/TableExt/

| File | What Was Removed | Why |
|---|---|---|
| `HBVGGeneralLedgerSetup.TableExt.al` | `GetCIName()` check (lines 66–67) that blocked G/L Setup field changes outside the central company. | G/L Setup is now editable directly. No central company restriction needed. |
| `HBVGCompanyInformation.TableExt.al` | `HBVG Setup Administration Name` field validation logic using `GetCIName()` and `GetPRJName()`. The `HBVG Setup Administration` boolean field. In second pass: `lcduDVBeheer: Codeunit "HBVG Beheer Mgt."` global var; `lDVOrgDimensionValue` and `lDVOrgDefaultDimension` vars; entire `MaakDimAndereAdm()` and `MaakDefDimAndereAdm()` call blocks that copied "Corresp. Organization" dimensions to the PRJ admin company via `ChangeCompany()`. | These fields stored the central company identity in Company Information. The `MaakDim*` calls propagated dimension values cross-company on every `HBVG Corresp. Organization` field change — removed because `HBVG Beheer Mgt.` codeunit deleted and single-company setup requires no cross-company propagation. |

### src/PageExt/ — Editability Guards Removed (15 files)

All 15 page extensions shared the same pattern:
```al
// REMOVED pattern:
if HBVGSingleInstanceMgt.GetMasterDataBedrijf() <> '' then begin
    HBVBoolEditable := CompanyName() = HBVGSingleInstanceMgt.GetMasterDataBedrijf();
    ...
end;
```
This made fields read-only in satellite companies (only editable in the master data company). In a single company this lock is always active and blocks all editing. Removed from all files below.

| File | Additional removals beyond editable guard |
|---|---|
| `HBVGCustomerCard.PageExt.al` | Also removed: `HBVG Central Update` sync call triggered on customer save action |
| `HBVGCustomerList.PageExt.al` | Editable guard only |
| `HBVGVendorCard.PageExt.al` | Also removed: `HBVG Central Update` sync call on vendor save action |
| `HBVGVendorList.PageExt.al` | Editable guard only |
| `HBVGContactCard.PageExt.al` | Also removed: `HBVG Central Update` sync call on contact save action |
| `HBVGContactList.PageExt.al` | Editable guard only |
| `HBVGEmployeeCard.PageExt.al` | Also removed: `HBVG Central Update` sync call on employee save action |
| `HBVGGLAccountCard.PageExt.al` | Editable guard only |
| `HBVGSalesPersonPurchaserCard.PageExt.al` | Also removed: `HBVG Central Update` sync call on salesperson save |
| `HBVGVendBankAccountCard.PageExt.al` | Editable guard only |
| `HBVGCustBankAccountCard.PageExt.al` | Editable guard only |
| `HBVGDimensionValues.PageExt.al` | Also removed: `HBVGBeheerMgt.BlokkeerBuitenCentraleInstellingen()` call on insert/modify. `GetCIName()` visibility check on action buttons. |
| `HBVGDimensions.PageExt.al` | `GetCIName()` check controlling action visibility (line 29) |
| `HBVGGeneralLedgerSetup.PageExt.al` | `BoolIsCB` variable and `GetCIName()` assignment used to show/hide central-only fields |
| `HBVGCompanyInformation.PageExt.al` | `GetCIName()` and `GetPRJName()` check controlling editable state (line 148) |

### src/PageExt/ — Multi-company Navigation Removed (1 file)

| File | What Was Removed | Why |
|---|---|---|
| `HBVGCompanies.PageExt.al` | `HBV_InHetInstellingenBedrijf` variable, all 3 `GetCIName()` references, `CopySaaSCompanySetupStatus()` call from central to new company. Action to create satellite companies from central. In second pass: `HBVCopyCompany: Report "HBVG Copy Company"` var and dead `Company.Get(HBVCopyCompany.GetCompanyName())` call. | Companies page no longer needs to identify or act as the central company. `HBVG Copy Company` report deleted — company creation wizard handles setup directly without copying from CI. |

### src/Pages/

| File | What Was Removed | Why |
|---|---|---|
| `HBVGJobQueueMonitor.Page.al` | `GetCIName()` checks (lines 178–179, 271, 279) that restricted "Create in Administration" action to central company only, and controlled button visibility. | Job queue monitoring is now unrestricted per company. |
| `HBVGPurchInvoicesAllAdm.Page.al` | `GetCIName()` call (line 256) used to exclude the central company from the "all administrations" purchase invoice list. | No central company to exclude — page shows all invoices for the current company. |
| `HBVGCompanyAccountScheme.Page.al` | `CentraalBeheer` group (line 37) and its child controls showing central company account scheme data. In second pass: `part(GLCompanySub; "HBVG G/L - Company") { SubPageLink = Administration = field(Name); }` — the subpage showing per-company G/L balances. | `HBVG G/L - Company` page was deleted with the Vastgoed Centraal folder. Subpage reference caused AL0185 compile error. |
| `HBVGApprovalperProject.Page.al` | `loInCB` variable and `GetCIName()` check (line 182) controlling "Copy to All" button visibility (only shown in central company). | Copy to all companies approval setup no longer applicable. |
| `HBVGApprovalGBKPL.Page.al` | `KnopKopieerAlleVisible` variable and `GetCIName()` check (line 116). | Same as above — "copy to all" approval config action removed. |
| `src/OAuth2.0/Pages/HBVGOAuth20Applications.Page.al` | `GetCIName()` and `GetMasterDataBedrijf()` checks (lines 95, 134, 136, 142) that restricted OAuth app configuration to central/master company only. In second pass: entire `CopyApplication` action (used `StartSession` to copy OAuth settings to all satellite companies via `SelecteerBedrijven()`); entire `SelecteerBedrijven()` procedure (opened `HBVG Select Lines` page to pick target companies); `TempCompanyNameValueBuffer`, `HBVGDefaultFunctions` vars and associated labels. | OAuth apps now configurable in the single company directly. Cross-company OAuth sync via StartSession is not needed. |

### src/Reports/

| File | What Was Removed | Why |
|---|---|---|
| `HBVGBalancealladmin.Report.al` | `GetCIName()` check (line 225) that excluded the central company from the "balance all administrations" report. | No central company to exclude. Report runs for the current company. |
| `HBVGWValleadministraties.Report.al` | `GetCIName()` check (line 230) — same exclusion pattern. | Same as above. |

### src/RoleCenter/Sub/

| File | What Was Removed | Why |
|---|---|---|
| `HBVGDebitorActivitiesSub.Page.al` | `GetCIName()` check (line 85) that hid debtor activities when running in the central company (central company had no debtors). | Single company is both central and operational. Activities always visible. |
| `HBVGCreditorActivitiesSub.Page.al` | `GetCIName()` check (line 62) — same pattern for creditor activities. | Same reasoning. |

### src/permissions/

| File | Lines Removed | Why |
|---|---|---|
| `HBVGFULLACCESS.PermissionSet.al` | `HBVG Delete Master Data Buffer = RIMD` (line 19), `HBVG Central Setup Period = RIMD` (line 72), `HBVG Central Update Setup = RIMD` (line 148), `HBVG Central Update Tables = RIMD` (line 149), `HBVG Central Update Fields = RIMD` (line 150) | Tables deleted — permissions referencing them cause compile errors. |
| `HBVGBEHEER.PermissionSet.al` | `HBVG Central Setup Period = RIMD` (line 20) | Table deleted. |
| `HBVGCREDBANK.PermissionSet.al` | `HBVG Delete Master Data Buffer = RIMD` (line 10) | Table deleted. |
| `HBVGBASIS.PermissionSet.al` | `HBVG Central Update Fields = RIMD` (line 18), `HBVG Central Update Tables = RIMD` (line 19) | Tables deleted. |
| `HBVGFINBEHEER.PermissionSet.al` | `HBVG G/L Account Admin. = RM` | Table (`HBVGGLAccountAdmin.Table.al`) was deleted as part of the Vastgoed Centraal folder removal. |

### src/Codeunits/

| File | What Was Removed | Why |
|---|---|---|
| `HBVGSubscribersAlgemeen.Codeunit.al` | `loGrootboekAdministratie: Record "HBVG G/L Account Admin."` variable and its 3-line delete block from `CompanyOnAfterDelete`; `loGrootboekAdministratie`/`loGrootboekAdministratie2` variables and their rename-sync loop from `CompanyOnAfterRename`. | `HBVG G/L Account Admin.` table was deleted. References cause compile errors. |
| `HBVGSubscribersFinancieel.Codeunit.al` | Entire `CheckUseGLAccount()` procedure (cross-company G/L account validation using `HBVG Setup Administration` flag and `HBVG G/L Account Admin.` table); `CheckUseGLAccount(Rec)` call from `T15OnBeforeDelete`; entire `GlAccountOnAfterDelete()` event subscriber (cleaned up `HBVG G/L Account Admin.` rows); `loGLAccountCompany: Record "HBVG G/L Account Admin."` variable and its `ModifyAll` call from `GLAccountOnAfterValidateAccountType`. Global var `HBVGUpdateCentrPrjAdm: Codeunit "HBVG Update Centr. Prj. Adm."` and its two `CheckUpdatePRJ()` calls in `T17OnBeforeInsert` and `T17OnBeforeModify` (set `HBVG Corresp. postvolgnr.` field). | `HBVG G/L Account Admin.` table and `HBVG Update Centr. Prj. Adm.` codeunit both deleted. Cross-company G/L validation and project administration sync have no purpose in single-company setup. |

### Other Module Files

| File | What Was Removed | Why |
|---|---|---|
| `Vastgoed Projecten/Codeunits/HBVGUpdateProjectLines.Codeunit.al` | Central company reference in project line update propagation. | Project lines updated locally only. |
| `Vastgoed Projecten/Codeunits/HBVGProjectStageMgt.Codeunit.al` | `GetCIName()` and `GetBijwerkenViaJobQueue()` references. | Stage management no longer synced to satellites. |
| `Vastgoed Projecten/Tables/HBVGApprovalProjGLAccount.Table.al` | `GetCIName()` guard blocking approval GL account setup in non-central companies. | Approval GL accounts now configurable directly. |
| `Vastgoed Prognoses/Pages/HBVGProjectForecastList.Page.al` | `GetCIName()` and `GetBijwerkenViaJobQueue()` references controlling forecast sync. | Forecasts local to single company. |
| `Vastgoed Planning/Pages/HBVGProjectPlanning.Page.al` | Central company references in planning sync. | Planning local to single company. |
| `Vastgoed Planning/Pages/HBVGProjectPlanningList.Page.al` | Central references. | Same. |
| `Vastgoed Rente/Tables/HBVGInterestCalc.Table.al` | Central reference in interest calculation. | Interest runs locally. |
| `Vastgoed Leningen/Reports/HBVGLoanMovements.Report.al` | `GetCIName()` company filter that excluded central company from loan report. | Report runs for current company only. |
| `Vastgoed Kopersadministratie/Codeunits/HBVGUpdateAdresses.Codeunit.al` | Cross-company address sync logic using `HBVG Central Update`. | Addresses updated locally only. |
| `Vastgoed Kopersadministratie/Pages/HBVGBuyerTenant.Page.al` | Central company reference. | Buyer/tenant data local to company. |
| `Vastgoed Kopersadministratie/Pages/HBVGInterestedBuyer.Page.al` | `UpdateCompanies` action (called `HBVG Central Update.KlantenLeveranciersContacten`); `GetMasterDataBedrijf()` editable guard in `OnOpenPage`; `HBVSingleInstanceMgt` var; `BoolMasterData` and `HBVBoolEditable` vars. | `HBVG Central Update` codeunit deleted. Master data concept removed — page is always editable. |
| `Vastgoed Kopersadministratie/Pages/HBVGSaleDissolution.Page.al` | `GetPRJName()` and `GetCIName()` read-only guard in `ActivateFields`; `HBVGSingleInstanceMgt` var. | Page is editable in all companies without central management. |
| `Vastgoed Cloud File Storage/Pages/HBVGDocumentLocation.Page.al` | Entire `UpdateAllCompanies2` action (propagated document location settings to all satellite companies); entire `SelecteerBedrijven()` procedure (company selector for CI-only use); `HBVSingleInstanceMgt` var; `TempCompanyNameValueBuffer` var. | Action and procedure were entirely CI-centric. Document locations now configured per-company directly. |
| `Vastgoed Setup/Pages/HBVGCreateCompanyWizard.Page.al` | `Report "HBVG Copy Company"` var and its calls (`SetNewCompanyName`, `SetTableView`, `RunModal`); `GetCIName()` filter on Company record. | Report deleted. Wizard no longer copies setup from CI company. |
| `Vastgoed Setup/Pages/HBVGBasicSetupWizard.Page.al` | `Report "HBVG Copy Company"` var and calls in both `CreateProjectAdm()` and `CreateDemoCompany()`; `SetPRJName()` call; `GetCIName()` filter. | Report deleted. Wizards simplified to record-level operations only. |
| `Vastgoed Setup/Codeunits/HBVGSupportTools.Codeunit.al` | Entire `UpdateOAuth()` procedure (copied OAuth app settings from CI company via `GetCIName()` to new satellite). | CI company no longer exists. OAuth configured per-company directly. |
| `Vastgoed Vastgoedbeheer/Tables/HBVGUnit.Table.al` | Cross-company loop (x=1/x=2) in `InsertDefaultDimensions()` that wrote default dimensions to both current and PRJ admin company via `GetPRJName()`; `HBVGBeheerMgt.MaakDimCorrOrg/MaakDefDimCorrOrg` calls; `HBVGBeheerMgt` and `DVSingle` vars. | `HBVG Beheer Mgt.` codeunit deleted. Dimensions written to current company only. |
| `Vastgoed Leningen/Tables/HBVGLoan.Table.al` | Cross-company loop in `FillDimValue()` that wrote default dimensions to PRJ admin company via `GetPRJName()`; `HBVGBeheerMgt.MaakDimCorrOrg/MaakDefDimCorrOrg` calls; `HBVGBeheerMgt` and `HBVGSingleInstanceMgt` vars. | Same reasoning as HBVGUnit. Loan dimensions written locally only. |

### Second Pass — Additional References Fixed (Compiler Error Pass)

The following files were not caught in the initial removal pass. Compiler errors revealed residual references to deleted objects (`HBVG Beheer Mgt.`, `HBVG Copy Company`, `HBVG G/L Account Admin.`, `HBVG Update Centr. Prj. Adm.`, `HBVG Select Lines`).

#### src/PageExt/

| File | What Was Removed | Why |
|---|---|---|
| `HBVGAnalysisViewList.PageExt.al` | Entire `HBVG_Copy_Settings` action (called `DVBeheerMgt.UpdateAnalysisViews()`); `actionref(HBVG_Copy_Settings_Promoted; HBVG_Copy_Settings)`; `DVBeheerMgt: Codeunit "HBVG Beheer Mgt."` global var. | `HBVG Beheer Mgt.` deleted. Action pushed analysis view settings to all satellite companies. No purpose in single company. |
| `HBVGAnalysisViewCard.PageExt.al` | Same pattern: `HBVG_Copy_Settings` action + promoted ref + `DVBeheerMgt` var. | Same reasoning as AnalysisViewList. |
| `HBVGAnalysisbyDimensions.PageExt.al` | `HBVG_Copy_Settings` action with local `HBVBeheerMgt` var calling `UpdateAnalysisViews()`; `actionref(HBVG_Copy_Settings_Promoted; HBVG_Copy_Settings)`. | Same — `HBVG Beheer Mgt.` deleted. |
| `HBVGVATStatement.PageExt.al` | `HBVG Transfer All Companies` action (called `HBVBeheerMgt.UpdateBTWaangifte()`); promoted actionref. | Pushed VAT statement configuration to all companies. Not needed in single company. |
| `HBVGUserSetup.PageExt.al` | `HBVG_Update_Administrations` action (called `DVBeheerMgt.UpdateUserSetup()`); `actionref(HBVG_Update_Administrations_Promoted; HBVG_Update_Administrations)`; `DVBeheerMgt` var. | User setup sync to satellite companies removed. |

#### src/Pages/

| File | What Was Removed | Why |
|---|---|---|
| `HBVGUserSetup.Page.al` | `Update Companies` action (called `Centralmanagement.UpdateUserSetup()`); `Centralmanagement: Codeunit "HBVG Beheer Mgt."` local var; `actionref("Update Companies_Promoted"; "Update Companies")`. | Same as above — `HBVG Beheer Mgt.` deleted. |

#### Other Module Files — Second Pass

| File | What Was Removed | Why |
|---|---|---|
| `Vastgoed Vastgoedbeheer/Tables/HBVGComplex.Table.al` | Cross-company loop (x=1/x=2) in `InsertDefaultDimension()` that wrote default dimensions to both current and PRJ admin company; `HBVGBeheerMgt.MaakDimCorrOrg/MaakDefDimCorrOrg` calls; `cduDVBeheer: Codeunit "HBVG Beheer Mgt."` and `HBVGSingleInstanceMgt` vars. | `HBVG Beheer Mgt.` deleted. Dimensions written to current company only. `DVAdministratie` var retained as still used elsewhere. |
| `Vastgoed Projecten/Tables/HBVGProject.Table.al` | `cduDVBeheer: Codeunit "HBVG Beheer Mgt."` var from `UpdateDimensies()`; entire PRJ admin `ChangeCompany` block (DimensionValue + DefaultDimension writes to PRJName company); `MaakDimCorrOrg/MaakDefDimCorrOrg` block. In `DeleteDimensions()`: replaced `DVSingleInstance.GetPRJName()` with `CompanyName()` for both deletes. | `HBVG Beheer Mgt.` deleted. Dimension updates and deletes now scoped to current company only. |
| `Vastgoed Setup/Pages/HBVGCreatePRJAdminWizard.Page.al` | `loHBVGCopyCompany: Report "HBVG Copy Company"` var and its `RunModal` calls from `CreateProjectAdm()`; `loHBVGSingleInstanceMgt` var; `SetPRJName()` call. | `HBVG Copy Company` report deleted. PRJ admin company created as a bare company; setup is entered manually. |
| `Vastgoed Setup/Pages/HBVGSupport.Page.al` | `UpdateOAuth` action (called `HBVGSupportTools.UpdateOAuth(CompanyName)`); `actionref(UpdateOAuth_Promoted; UpdateOAuth)`. | `UpdateOAuth()` procedure was removed from `HBVGSupportTools` in first pass (the procedure copied OAuth settings from CI company). AL0132 (method not found) and AL0271 (actionref target not found) both resolved. |

---

## RECREATED FILES

The following files were deleted as part of the Vastgoed Centraal folder removal but needed to be recreated because they served non-CM purposes that remained after the removal.

| File | Object | Reason for Recreation |
|---|---|---|
| `src/Pages/HBVGSelectLines.Page.al` | Page 50500 | The original `Vastgoed Centraal/Pages/HBVGSelectLines.Page.al` was a generic line-selection helper page. It was used by CM features (OAuth cross-company sync, company picker) **and** by non-CM features: `HBVGCashflowLiqImport` (select cashflow import files) and `HBVGUpdateAdresses` (select projects for address update). Recreated at a new ID (50500) with the original `FillTempFile()` / `GeefGeselecteerdeBestanden()` API. The CM callers were refactored away; only the two non-CM callers remain. |

---

## TOTALS

| Action | Count |
|---|---|
| Files deleted entirely | 24 |
| Files modified partially | 67 |
| Files recreated (new ID) | 1 |
| **Total AL objects affected** | **92** |

---

## NET EFFECT

| Before | After |
|---|---|
| Every save triggers Change Log → cross-company sync check | No cross-company sync |
| `CheckCB()` runs on every setup table operation | No company identity check |
| Session startup loads central company name into SingleInstance | Session startup loads only module flags |
| Dimensions, G/L accounts, customers, vendors read-only in satellite | All master data editable directly |
| Approval routing checks central company identity | Approval routing based on project team only |
| Test requires multi-company scaffold | Test requires single company only |
| Job Queue runs background central sync jobs | No central sync jobs |

---

*Generated automatically during DBBE migration — Central Management removal phase.*
