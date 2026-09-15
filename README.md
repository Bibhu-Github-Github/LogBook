Task: Build "Issue Stage by Market Categories" Report
Inputs
op03 dump.xlsx — contains: Issue ID, Severity, Status, Market Type, Ownership (and possibly other columns).
lookup.xlsx (second file) — contains: Issue ID, MSII Theme, DQ Issue Category.
Step 1 — Join
LEFT JOIN op03 dump.xlsx (left table) with lookup.xlsx (right table) on Issue ID.
Keep every row from op03 dump.xlsx, even if there is no match. Unmatched rows get blank MSII Theme / DQ Issue Category.
Audit: log total row count of op03 dump, total row count after join (must be identical), and count of unmatched Issue IDs.
Step 2 — Normalize text fields
For Ownership, Market Type, MSII Theme, DQ Issue Category, Severity:
Trim whitespace.
Treat these values as blank (case-insensitive): null, empty string, "-", "TBD", "TBC".
Lowercase Ownership → Ownership_norm.
Lowercase Market Type → MarketType_norm. Expected values: home, network, value, or blank.
Severity_norm: if the value matches pattern "<digit>. <word>" (e.g. "2. High"), strip the number and keep only the word, lowercase it (e.g. "2. High" → "high"). "TBC" / "5. TBC" → blank.
Audit: log a distinct-value count table for each normalized field (before → after) so mapping mistakes are visible immediately.
Step 3 — WPB reclassification ("MSII to be raised")
IF Ownership_norm = "wpb" AND MSII Theme is blank AND DQ Issue Category is blank:
Set MSII Theme = "MSII to be raised"
Set DQ Issue Category = "TBC"
Do NOT delete these rows.
Audit: log count of rows reclassified this way.
Step 4 — Bucket mapping (from Status)
Create new column Bucket:
Status contains
Bucket
Draft Issue, Validate Issue, Identify Root Source, Handover to Producer
Yet to Start
Identify Solution, Identify Root Cause
Identify Solution
Implement Solution
Implement Solution
Verify Solution
Verify Solution
Business Accepted, On Hold, Resolved
Exclude
After mapping, delete all rows where Bucket = "Exclude".
Audit: log row count before and after exclusion, and count removed per Status value.
Step 5 — Display fields (controls pivot row grouping)
Create Display MSII Theme:
If Ownership_norm = "wpb" → use the real MSII Theme value (including "MSII to be raised" from Step 3).
Else → "TBC".
Create Display DQ Issue Category:
If Ownership_norm = "wpb" → use the real DQ Issue Category value.
Else → use the raw, original Ownership value exactly as it appears in the source file — no renaming, no suffix (e.g. if Ownership = "Risk", show "Risk"; if Ownership = "UK CDAO", show "UK CDAO"; if Ownership = "Issues yet to be handed over", show that literal value as-is).
Audit: log the distinct list of all non-WPB Ownership values that will become rows, so nothing silently merges or disappears.
Step 6 — Build the pivot: "Issue Stage by Market Categories"
Rows (in this exact order):
WPB section: all rows where Display MSII Theme != "TBC", grouped by Display MSII Theme → Display DQ Issue Category.
Subtotal row: "WPB Total" = sum of all WPB rows above.
TBC section: one row per distinct raw Ownership value found in Step 5, sorted alphabetically.
Final row: "Overall Total" = WPB Total + all TBC section rows.
Columns (two-level header):
Level 1: Home Markets, Network Markets (add Value Markets only if data contains that market type).
Level 2, under each market group, in this exact order: Yet to Start, Identify Solution, Implement Solution, Verify Solution.
Final column: Total Issues = row total across all displayed market/bucket columns.
Cell values: COUNT of DISTINCT Issue ID for that row × column combination.
Audit: log grand total distinct Issue ID count from the raw (post-Step-4) dataset, and confirm it equals the "Overall Total" → "Total Issues" cell. If they don't match, log the list of Issue IDs excluded and why (e.g., blank Market Type, blank Bucket).
Step 7 — Output
Export to one Excel workbook with these sheets:
7A — Main pivot: sheet named Issue Stage by Market Categories, built per Step 6.
7B — Filters on the main pivot (or slicers next to it):
Market Type filter (Home / Network / Value)
Ownership filter (must include at least: wpb, finance, gpb, and blanks/other as applicable)
7C — Severity-split pivots (same row/column structure, ordering, and distinct-Issue-ID counting as Step 6, just filtered):
Sheet High Risk Issues: only rows where Severity_norm = "high". Exclude everything else including blank severity.
Sheet Medium & Low Severity Issues: only rows where Severity_norm IN ("medium", "low"). Exclude everything else including blank severity.
Do NOT include TBC/blank severity issues in either severity sheet.
Step 8 — Validation checks (must run and report before finishing)
Print a short audit summary with:
Row counts at each step (raw → joined → after exclusion → final).
WPB Total = sum of WPB section rows? (Yes/No, show the two numbers)
Overall Total = WPB Total + sum of TBC section rows? (Yes/No, show the two numbers)
Sum of all "Total Issues" column values across every row = Overall Total's "Total Issues" cell? (Yes/No)
Count of Issue IDs in the High Risk sheet + Medium & Low sheet + (blank/TBC severity, excluded) = total Issue ID count in the main pivot? (Yes/No)
List any Issue ID that appears zero times or more than once across the final pivot (should never happen — flag if it does).
List of distinct Ownership values used to build the TBC section rows (so I can visually confirm nothing was merged incorrectly).
If any check fails, stop and show me exactly which Issue IDs are causing the mismatch, rather than only showing the wrong total.
