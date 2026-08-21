# NovaMart Project: Consolidated Learning Notes

## Excel / Pivot Mechanics

**Pivot cache sharing causes silent bleed.** If two PivotTables share a cache, a field-grouping change on one can silently apply to the other. Give each analytical tile its own cache when tiles need independent grouping behavior.

**Rename PivotTable objects manually, not via script.** Scripted renaming was found to trigger cache corruption in this build. Manual rename via the PivotTable Options pane is safer.

**"% of Row Total" can't be sorted directly.** The percentage display value doesn't sort reliably on its own. Fix: keep a raw count/flag field, sort using an AVERAGE-based helper value instead of the percentage field itself, and use the percentage only for display.

**PERCENTILE.INC breaks silently on low-cardinality data.** Watch for this whenever the underlying value set has few distinct values. The function can return misleading results without throwing an error.

**MINIFS fails on text-formatted Order_IDs.** MINIFS needs numeric ranges to compare. If an ID field is stored as text, MINIFS returns errors or blanks. Prefer XLOOKUP with a combined key (e.g. Customer_ID + Date) when Order_ID alone isn't numeric or isn't a safe join key on its own.

**Same-day tie resolution.** When multiple orders share a date and you need a deterministic "first" record, MIN(date) alone isn't enough. Use FILTER + SORT with a secondary tiebreaker field (like Order_ID) to resolve ties consistently.

**Structured references.** `@` in a table formula means "just this row"; no `@` means "the whole column." Only prefix with the table name when referencing a *different* table.

**XLOOKUP over VLOOKUP.** Preferred for multi-condition lookups and for resilience to column insertion. Use a combined key when a single field isn't unique enough to look up on safely.

**Paste-as-values timing.** Only convert formulas to static values after an objective is fully validated and closed. Converting early removes the ability to trace back if something looks wrong later.

## Analytical Discipline

**Hypothesis-first, always.** Write the expected finding in plain business language before opening a pivot. This keeps the analytical question honest and separate from whatever the data happens to show.

**5-point audit checklist, every finding:**
1. Does the sample size actually support this claim?
2. Is the percentage type (row / column / grand total) the right one for this question?
3. Does it hold up when sliced a different way?
4. Am I treating correlation or co-occurrence as sequence or causation?
5. Would a skeptical reader find a hole in this in 30 seconds?

**Small samples invalidate signals.** A 29.17% return rate built on 7 returned orders is not a finding. It's noise wearing a percentage sign. Always check the denominator before trusting a rate.

**Correlation ≠ sequence ≠ causation.** A pivot showing two things co-occurring cannot tell you which happened first, let alone whether one caused the other. When the data can't establish this, say so and park the question rather than overreaching.

**Additive effects can look like interactions.** The city x category return-rate pattern turned out to be two separate effects stacking, not a true interaction between city and category. Worth explicitly testing for this distinction before writing it up as "categories behave differently across cities." That's a stronger and different claim than "returns are elevated in general in some cities, regardless of category."

## Write-Up Discipline

**Keep each finding's recommendation inside that finding's own evidence.** Don't let Finding 5's recommendation quietly borrow support from Finding 3, even when they're related in the reader's mind. If the data can't establish a cause (e.g., *why* two cities show elevated returns), say that plainly and recommend further investigation instead of inventing a plausible-sounding explanation.

**Structure that scales:** What we found → why it matters to the business → what we recommend. Applied consistently across all 5 findings in the write-up, this made the report easy to skim for a busy stakeholder while keeping the reasoning auditable underneath.

---
*Consolidated at the close of Project 1 (NovaMart), Dec 2025.*
