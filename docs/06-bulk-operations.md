# 6. Bulk Operations Without Regret

Backfills and retro fixes are where a CRM integration can destroy months of sales data in seconds.

---

## Target exact IDs, never a heuristic

**Symptom:** a filter meant to match about a thousand records matched about nine times that, and overwrote several hundred correct values.

**Fix:**
- Target the **exact IDs** you logged at write time, not a filter that "should" select the same records.
- Run `COUNT()` on the target set before writing. If it differs from what you expect, stop.
- Test on **five** records, read them back, then run the rest.
- Make scripts **dry-run by default**, with an explicit confirm token for live writes.
- Turn on **field history tracking** for fields you bulk-update, so a mistake can be undone.

---

## Bulk updates have side effects

**Symptom:** a retro attribution fix through the REST API moved some leads to the API user (a few taken from real agents) and recalculated the lead score on almost all of them.

**Cause:** assignment rules and score recalculation fire on update.

**Fix:** expect side effects from any bulk lead update and check owners afterwards. In this org, the Bulk API 2.0 did **not** fire the assignment rule (zero owner changes on the same kind of job), but verify in yours.

---

## Composite requests for speed

Composite PATCH in chunks of 200 with `allOrNone=false` updated several hundred records in about ten seconds, and reports success or failure per record, so partial failures are visible.

---

## Silent caps

**Symptom:** a CLI query over about 90k records returned exactly 50,000, with exit code 0.

**Fix:** always compare **returned** vs **totalSize**, and chunk by date range. A round number like 50,000, 2,000 or 200 is a cap, not a count.

The same applies inside automation: a search step that runs **once per input item** returned a "total" of almost 20k rows that was really the same rows repeated. Check whether a node runs per item before trusting its count.
