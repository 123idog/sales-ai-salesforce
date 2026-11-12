# 1. The Org Fights Back

A mature Salesforce org is full of automation written by other people: assignment rules, before-save and after-save Flows, duplicate rules, record-type picklists. Your integration's write is only the first event in a transaction you don't control.

---

## Fields you set on create get overwritten

**Symptom:** the create succeeds, but the stored owner and lead source differ from what you sent. Field history shows no change, because it happened inside the insert.

**Cause:** assignment rules and Flows run in the same transaction as the insert.

**Fix:** create, then PATCH only the fields that were overwritten, with the header `Sforce-Auto-Assign: FALSE`. Putting assignment options in the request body is rejected as an invalid field.

**Detect:** read the record back after create and compare it with what you sent.

---

## The no-auto-assign header doesn't stop a Flow

**Symptom:** every update from the integration reassigns the lead to the API user.

**Cause:** a before-save Flow sets the owner to the running user on update. The header only controls assignment rules.

**Fix:** end every update path with an explicit owner PATCH as the **last** write. Audit each branch; in one case a disabled node meant one branch skipped the restore.

---

## Converted leads are frozen

**Symptom:** `CANNOT_UPDATE_CONVERTED_LEAD`. In one backfill, more than half the targets failed. A 5-record test run caught it first (4 of 5 failed).

**Cause:** a platform rule, not a permission.

**Fix:** add `IsConverted = false` to every query that feeds a write, and report updatable vs. frozen counts separately. When the data must be corrected, correct it on the **Opportunity**, which usually carries the same fields.

---

## Restricted picklists: labels, record types, objects

- **Labels are localised, API values aren't.** Sending the label you see in the UI gives `INVALID_OR_NULL_FOR_RESTRICTED_PICKLIST`. Send the API value. The list-view describe endpoint and Tooling queries show the real values.
- **The same field can accept a value on Lead and reject it on Opportunity.** Value sets are per object and per record type.
- **Diagnose before escalating:** write a *different* value to the same field. If that works, it's a value-set problem, not a permission problem.
- **Isolate the risky field.** Send it as its own subrequest in a composite call (`allOrNone=false`). The rest of the record saves now, and the field starts working with no code change once an admin adds the value.

---

## REST gaps

- **Lead conversion isn't exposed over REST.** Only SOAP works, and it needs a raw session ID. A direct Opportunity insert may get its stage and close date rewritten by automation (e.g. a close date far in the future). Create, then PATCH.
- **Required fields are revealed one at a time.** Each failed create names one missing field. Iterate with sensible defaults until the create succeeds, then document the full list.
- **Some standard fields are mirrors.** Writing to standard email or phone on a contact can return success and store null, because a Flow copies from custom "primary" fields. Write the source field.
- **Duplicate rules block integration inserts.** Send `Sforce-Duplicate-Rule-Header: allowSave=true` when the duplicate is intentional.
- **Long-text fields can't be filtered in SOQL.** Fetch and match in code, or reach the record through a relationship.

---

## Reuse Salesforce's own query

When the business says "the leads in *that* list view", call the list view's describe endpoint and use its `WHERE` clause verbatim. Confirm with `COUNT()`; expect a drift of a record or two as data changes.
