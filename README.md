Task: Build "Issue Stage by Market Categories"
Report
 READ THIS FIRST — WHAT YOU MUST DELIVER
Your job is to produce three finished pivot tables (Step 6/7A main pivot, plus the two
severity-split pivots from Step 7C). The audit checks in Step 8 are a supporting appendix,
not the deliverable — do not stop after validating the join or printing counts.
Do not pause, ask clarifying questions, or report back after any individual step (Step 1, Step
2, etc.). Execute all 8 steps in one continuous pass and only then show me your output, in
this exact order:
1. The main pivot table ("Issue Stage by Market Categories"), fully populated with
numbers.
2. The "High Risk Issues" pivot table, fully populated.
3. The "Medium & Low Severity Issues" pivot table, fully populated.
4. THEN the Step 8 audit summary (row counts, reconciliation checks, distinct ownership
list).
If you cannot complete all three tables, tell me exactly which step you got stuck on and why
— do not substitute the audit trail as if it were the finished output.
Inputs
1. op03 dump.xlsx — contains: Issue ID, Severity, Status, Market Type, Ownership (and
possibly other columns).
2. lookup.xlsx (second file) — contains: Issue ID, MSII Theme, DQ Issue Category.
Step 1 — Join
LEFT JOIN op03 dump.xlsx (left table) with lookup.xlsx (right table) on Issue ID .
Keep every row from op03 dump.xlsx , even if there is no match. Unmatched rows get
blank MSII Theme / DQ Issue Category.Audit: log total row count of op03 dump, total row count after join (must be identical),
and count of unmatched Issue IDs.
Step 2 — Normalize text fields
For Ownership , Market Type , MSII Theme , DQ Issue Category , Severity :
Trim whitespace.
Treat these values as blank (case-insensitive): null , empty string, "-" , "TBD" ,
"TBC" .
Lowercase Ownership → Ownership_norm .
Lowercase Market Type → MarketType_norm . Expected values: home , network ,
value , or blank.
Severity_norm : if the value matches pattern "<digit>. <word>" (e.g. "2. High" ),
strip the number and keep only the word, lowercase it (e.g. "2. High" → "high" ).
"TBC" / "5. TBC" → blank.
Audit: log a distinct-value count table for each normalized field (before → after) so
mapping mistakes are visible immediately.
Step 3 — WPB reclassification ("MSII to be raised")
IF Ownership_norm = "wpb" AND MSII Theme is blank AND DQ Issue Category is blank:
Set MSII Theme = "MSII to be raised"
Set DQ Issue Category = "TBC"
Do NOT delete these rows.
Audit: log count of rows reclassified this way.
Step 4 — Bucket mapping (from Status )
Create new column Bucket :
Status contains Bucket
Draft Issue, Validate Issue, Identify Root Source, Handover to Producer Yet to Start
Identify Solution, Identify Root Cause Identify Solution
Implement Solution Implement Solution
Verify Solution Verify Solution
Business Accepted, On Hold, Resolved Exclude
After mapping, delete all rows where Bucket = "Exclude" .
Audit: log row count before and after exclusion, and count removed per Status value.
Step 5 — Display fields (controls pivot row grouping)
Create Display MSII Theme :
If Ownership_norm = "wpb" → use the real MSII Theme value (including "MSII to be
raised" from Step 3).
Else → "TBC" .
Create Display DQ Issue Category :
If Ownership_norm = "wpb" → use the real DQ Issue Category value.
Else → use the raw, original Ownership value exactly as it appears in the source file
— no renaming, no suffix (e.g. if Ownership = "Risk", show "Risk"; if Ownership = "UK
CDAO", show "UK CDAO"; if Ownership = "Issues yet to be handed over", show that
literal value as-is).
Audit: log the distinct list of all non-WPB Ownership values that will become rows, so
nothing silently merges or disappears.
Step 6 — Build the pivot: "Issue Stage by Market Categories"
Rows (in this exact order):
1. WPB section: all rows where Display MSII Theme != "TBC" , grouped by Display MSII
Theme → Display DQ Issue Category .
2. Subtotal row: "WPB Total" = sum of all WPB rows above.
3. TBC section: one row per distinct raw Ownership value found in Step 5, sorted
alphabetically.
4. Final row: "Overall Total" = WPB Total + all TBC section rows.
Columns (two-level header):
Level 1: Home Markets , Network Markets (add Value Markets only if data contains
that market type).
Level 2, under each market group, in this exact order: Yet to Start , Identify
Solution , Implement Solution , Verify Solution .
Final column: Total Issues = row total across all displayed market/bucket columns.
Cell values: COUNT of DISTINCT Issue ID for that row × column combination.
Audit: log grand total distinct Issue ID count from the raw (post-Step-4) dataset, and
confirm it equals the "Overall Total" → "Total Issues" cell. If they don't match, log the list
of Issue IDs excluded and why (e.g., blank Market Type, blank Bucket).
Step 7 — Output
Export to one Excel workbook with these sheets:
7A — Main pivot: sheet named Issue Stage by Market Categories , built per Step 6.
7B — Filters on the main pivot (or slicers next to it):
Market Type filter (Home / Network / Value)
Ownership filter (must include at least: wpb, finance, gpb, and blanks/other as
applicable)
7C — Severity-split pivots (same row/column structure, ordering, and distinct-Issue-ID
counting as Step 6, just filtered):
Sheet High Risk Issues : only rows where Severity_norm = "high" . Exclude
everything else including blank severity.
Sheet Medium & Low Severity Issues : only rows where Severity_norm IN ("medium",
"low") . Exclude everything else including blank severity.
Do NOT include TBC/blank severity issues in either severity sheet.
Step 8 — Validation checks (run AFTER the three tables above
are already built and shown)
This is an appendix, not a substitute for the tables. Print a short audit summary with:
1. Row counts at each step (raw → joined → after exclusion → final).
2. WPB Total = sum of WPB section rows? (Yes/No, show the two numbers)
3. Overall Total = WPB Total + sum of TBC section rows? (Yes/No, show the two numbers)
4. Sum of all "Total Issues" column values across every row = Overall Total's "Total Issues"
cell? (Yes/No)
5. Count of Issue IDs in the High Risk sheet + Medium & Low sheet + (blank/TBC severity,
excluded) = total Issue ID count in the main pivot? (Yes/No)
6. List any Issue ID that appears zero times or more than once across the final pivot
(should never happen — flag if it does).
7. List of distinct Ownership values used to build the TBC section rows (so I can visually
confirm nothing was merged incorrectly).
If any check fails, stop and show me exactly which Issue IDs are causing the mismatch,
rather than only showing the wrong total.
