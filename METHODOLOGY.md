# Methodology

## Analytical Approach

Every objective in this project followed the same three steps, in order:

1. **Hypothesis first.** Before opening a pivot table, write down what I expect to find and why, in plain business language. This forces the analytical question to be answered before the data question is even asked.
2. **Design before building.** Sketch the pivot/table structure on paper (or in notes), including what goes in rows, columns, values, and what percentage type (row/column/grand total) actually answers the hypothesis, before touching Excel.
3. **Audit before trusting.** Every finding was checked against a 5-point audit before it was allowed into the write-up: Does the sample size support the claim? Is the percentage type correct for the question being asked? Does the result hold up if I slice it a different way? Is this correlation being mistaken for sequence or causation? Would a skeptical reader find a hole in this in 30 seconds?

This discipline caught real errors before they became false conclusions. See the bugs below.

## Objective-by-Objective Method

**Obj 1: Revenue and Return Rate.** Established baseline revenue and return rate by category and average price, to understand the shape of the business before asking retention questions.

**Obj 2: Delivery, Payment, and Repeat Behavior.** Analyzed return rate by delivery method and payment method, average delivery days by city, monthly order volume by city, and, the key link in the report, return-vs-repeat behavior on a customer's first order.

**Obj 3: RFM Segmentation.** Built Recency-Frequency-Monetary scoring from a base pivot, then mapped customers into segments (Champions, Loyal, Promising New, Developing, Lost/One-Time) to quantify the retention problem's size.

**Obj 4: Discount Behavior and First-Order Timing.** Examined discount frequency and depth by customer type, and built a helper table + final pivot tracking whether discounts actually converted one-time buyers into repeat buyers (they largely didn't).

**Obj 5: City x Category Return Patterns.** Layered category mix by delivery bucket, return rate by city × category, and return rate by city alone, to isolate whether elevated returns in certain cities were category-driven or structural.

## Bugs & Fixes

### 1. Pivot cache bleed
Several PivotTables were sharing the same underlying pivot cache. Because of this, a field grouping change made for one tile (e.g. grouping dates into months) silently applied to other pivots built from the same cache, producing unexpected groupings on tiles I hadn't touched. **Fix:** gave each analytical tile its own dedicated pivot cache rather than reusing one across multiple tables, and renamed each PivotTable object manually (renaming via script/macro was found to trigger cache corruption).

### 2. "% of Row Total" can't be sorted directly
Excel's "% of Row Total" pivot value doesn't sort reliably when applied directly. The sort quietly breaks or reorders against the wrong reference. **Fix:** kept a raw count/flag field alongside the percentage field, sorted on an AVERAGE-based helper calculation instead of the percentage display value directly, and used the percentage field for display only.

### 3. Same-day order tiebreaker
When identifying a customer's "first order" for the return-vs-repeat analysis, some customers had multiple orders logged on the same date, so a simple MIN(date) lookup couldn't reliably identify a single first order. **Fix:** used FILTER + SORT with a secondary tiebreaker (Order_ID) to deterministically resolve which order counted as "first" when dates were tied.

### 4. MINIFS failure on text-formatted Order_IDs
MINIFS returned errors/blank results when the Order_ID field was stored as text rather than a true numeric ID. MINIFS requires numeric comparison ranges. **Fix:** switched to XLOOKUP with a combined key (Customer_ID + Date) for cases where Order_ID alone wasn't a safe join field, avoiding the need for numeric MINIFS logic altogether.

### 5. Mid-project file corruption
The workbook corrupted mid-session after heavy pivot/cache manipulation. **Fix:** rebuilt the affected tabs from the last stable save rather than attempting an in-place repair, which also served as a natural checkpoint to re-verify earlier objectives were still intact.

## A Note on Write-Up Discipline

Each finding in the write-up is deliberately scoped to its own evidence. The recommendation for Finding 3 (category-level returns) doesn't borrow support from Finding 5 (city-level returns), even though they're related. Where the data couldn't establish causation (e.g., *why* Faisalabad and Peshawar show structurally elevated returns), the write-up says so explicitly and recommends further investigation rather than guessing at a cause the pivot tables can't support.
