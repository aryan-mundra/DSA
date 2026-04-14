Full prompt 
Role
You are a senior engineer implementing a local-only billing application for a small office. The app is not hosted; it runs only on the user’s laptop/desktop. Use Java and JavaFX. Core features must work without the internet.
Product summary
The app replaces an Excel-style workflow: the user fills a bill, uses Print Preview, then prints or saves PDF via the system print dialog. There is no database and no saving of customer or bill data to disk—only in-memory state until the user prints or exports through the OS.
The bill contains two separate line-item areas shown side by side (like two tables on one sheet). Each area has the same three columns: Product Number, Quantity, Total (line amount). The two areas do not share rows; they calculate independently. Each area has its own subtotal (e.g. subtotal quantity and/or subtotal amount—specify in UI). The grand total is the sum of subtotal A and subtotal B (for amounts; if quantity grand total is shown, define as sum of both section quantities or document otherwise).
Print: A4 landscape, two halves: office copy (left) = where the user edits; customer copy (right) = live mirror of the full page (including both sections, headers, subtotals, grand total). No duplicate typing for the customer half.
Requirements
The application runs only locally; it is not hosted.
The application must not depend on the internet for core billing, editing, print preview, or printing.
The application is built with Java (current LTS; state version in docs).
The user interface is built with JavaFX.
The user can create and edit a bill with header fields and line items.
Header fields include at minimum:
a. Document title
b. Name
c. Address
d. Date

Editable on the office / master side; mirrored on the customer half for print.
The bill body contains two independent sections (call them Section A and Section B) side by side on the editing view:
a. Section A columns: Product Number, Quantity, Total (line amount).
b. Section B columns: Product Number, Quantity, Total (line amount).
c. Sections do not share line rows; each section has its own list of rows and its own running calculations.
Column semantics (each section):
a. Product Number: integer or category/label row (text) per existing rules.
b. Quantity: integer for data rows.
c. Total: Product Number × Quantity only when both are integers (same integer gate as before); category rows no auto multiply; avoid misleading zeros.
Independence: Changing rows in Section A must not alter Section B line data except grand total (and any full-page summary that aggregates both). Subtotals are computed per section only from that section’s rows.
Subtotals:
a. Subtotal A = aggregate of Section A (at minimum sum of line totals for data rows; optionally sum of quantities if required—implement consistently).
b. Subtotal B = same for Section B.
c. Subtotals update when any line in that section changes.
Grand total:
a. Grand total (amount) = Subtotal A (amount) + Subtotal B (amount).
b. If the UI shows a grand quantity, define it as qty A + qty B (or document an alternative—default is sum of both sections’ quantities).
Product names are not required in the grid (labels may read Product Number / category as per business).
Category / section header rows (text in Product Number column) supported; no multiplication on those rows within that block.
Print layout: A4, landscape.
Printed page: left half = office copy (source of editing), right half = customer copy = mirror of entire bill (both sections A and B, headers, subtotals A/B, grand total). No duplicate data entry for the right half.
Print Preview is mandatory before printing.
User can print or use system print dialog to save as PDF.
No database.
No persistent customer/bill storage as an app feature; bill lives in memory until app closes or user prints/PDFs.
No mandatory cloud, login, or telemetry.
Architecture: domain (two sections + lines), calculation services, print layout, UI—clear packages.
Unit tests: integer multiply rule per section, Subtotal A, Subtotal B, Grand = A + B, mirroring not duplicated in tests if UI-only—test model and totals.
Deliverables: runnable app, README, short user guide, tests.
Packaging documented (e.g. jpackage / fat JAR) for at least one OS.
(Add 25., 26., … as needed.)
Out of scope (unless added later)
Database, bill archive, CRM, sync, web/mobile, payments, tax engines.
Suggested implementation order
Domain: Section A and Section B line lists + row types (category vs data).
Calculation: integer gate per line, subtotal A, subtotal B, grand total.
JavaFX: two grids side by side + summary rows.
Print: A4 landscape; mirror full content to right half.
Print preview, system print, tests, README, packaging.
























 
Phase 2 prompt — “New” control & “New Total” column (rewrite)
Assumption: Phase 1 exists or will be built first: local Java + JavaFX billing app, offline, in-memory, no DB, two side-by-side sections (A and B), each with Product Number, Quantity, Total (integer gate for multiply), Subtotal A / Subtotal B, Grand total = A + B, A4 landscape print with office left / customer right mirror, Print Preview, no persistent customer data.
Role
Extend Phase 1 with a “New” control and an optional fourth column “New Total”, with per-line quantity discounts. Discounts apply only to the row that qualifies—not to the whole bill, not to other rows.
Product summary
While creating or editing the bill, show a “New” control at the top-right of the bill area. It is only for the editing UI: hide it in Print Preview and do not print it.
When “New” is on, each section shows a fourth column: New Total. When “New” is off, hide that column and use Phase 1 behavior only.
For each data row (integer Product Number × integer Quantity → Total):
Quantity	New Total	What counts toward section subtotal for this row
0, 1, 2	Blank (empty)	Total only
3, 4, 5	Line after 10% discount on Total	New Total
6 or more	Line after 15% discount on Total	New Total
 
Category / header rows (non-data rows): no Total multiply, New Total blank; exclude from numeric subtotals as in Phase 1.
Section subtotal (amount): For each section, sum one number per data row: use New Total when it is not blank; when New Total is blank (qty 0–2), use Total for that row.
Grand total (amount) = Subtotal A + Subtotal B (using the subtotal rule above when “New” is on; when “New” is off, subtotals match Phase 1).
Clarify in UI whether one global “New” toggles both sections (recommended) or per-section toggles—document the choice.
Requirements
“New” control is placed top-right of the bill area (layout relative to the bill content).
“New” is visible only during create/edit on the main screen; it is not shown in Print Preview and not included on printed or PDF output.
When “New” is enabled, show column “New Total” for Section A and Section B (or only the sections your Phase 1 spec defines—document).
When “New” is disabled, hide “New Total”; calculations and displays match Phase 1 (subtotals from Total only).
Discounts are per row only. Changing one row does not apply discount to other rows.
For a data row with valid Total = Product Number × Quantity:
a. Qty 3, 4, or 5: New Total = Total × 0.90 (10% discount). Rounding rule: document (e.g. integer or 2 decimals).
b. Qty ≥ 6: New Total = Total × 0.85 (15% discount). Same rounding rule.
c. Qty 0, 1, or 2: New Total is blank (empty). Do not copy Total into New Total.
Subtotal A (amount) with “New” on: for each data row in A, add New Total if it is not blank; otherwise add Total (covers qty 0–2). Exclude category/header rows.
Subtotal B (amount) with “New” on: same rule as requirement 7 for Section B.
Grand total (amount) = Subtotal A + Subtotal B (always, using the active subtotal rules).
Editing Product Number or Quantity recalculates Total and New Total (or clears New Total per rules) for that row only.
Unit tests: qty 2 → New Total blank, row contributes Total to subtotal; qty 4 → 10%; qty 6+ → 15%; mixed rows → subtotal = sum of correct branch per row; “New” off → Phase 1 subtotals.
User guide: explains “New”, print visibility, discount tiers, blank New Total for qty 0–2, and subtotal = sum of New Total or Total per row as above.
Out of scope (Phase 2)
Database, cloud, bill history, global discount on entire bill without per-line rules, changing Phase 1 no-persistence policy.
Dependency
Phase 1 bill model (two sections, three base columns, integer multiply, mirrors, print).
End of Phase 2 prompt.
 
 
 
