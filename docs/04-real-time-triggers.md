# 4. Real-Time Triggers

Sending a WhatsApp message when a deal is paid, alerting a rep when a lead heats up, firing a conversion event to an ad platform: all of these start with "when something changes in Salesforce".

---

## Push, not poll

**Symptom:** a polling trigger ("records modified since last run") missed most status changes.

**Causes:**
- timestamp-cursor edge loss between runs
- polling sees the current state, not the **transition** (A → B → A looks like nothing happened)
- dedup collapses rapid successive edits

**Fix:** a record-triggered Flow with an **is-changed** condition calls your webhook (via a named credential with a secret header). It fires exactly once per transition and costs no API calls.

---

## Outbound Messages must be acknowledged

**Symptom:** a customer received the same WhatsApp message three or more times.

**Cause:** Outbound Messages expect a SOAP acknowledgement. Without `Ack=true`, Salesforce retries for up to 24 hours.

**Fix:** respond **immediately** with the SOAP ACK before doing any work. If a backlog has built up, deactivate the Outbound Message temporarily to drain it.

---

## Outbound Message delivery lag is variable

The same Outbound Message was delivered in one second on one test and in over four minutes on the next. It's a shared queue.

- Wait at least five minutes before concluding that it didn't fire.
- Never delete dedup rows while a message may still be queued, or the late delivery becomes a duplicate.

---

## Deploying a Flow doesn't activate it

**Symptom:** the deploy reports success (or "unchanged"), and the Flow stays inactive.

**Cause:** activation lives on the Flow *definition*, not the version.

**Fix:** set the active version number through the Tooling API, then verify with a query. Also beware: filtering Flows by the trigger object's **label** returns nothing and looks like "no flows exist"; filter by API name.

---

## Conversion-API flows: filter and structure

Flows that send conversion events to an ad platform must send **only** the right events:
- **Filter by brand or business line.** Without it, leads of one brand fed another brand's ad audiences and optimisation.
- **Use the structured is-changed operator**, not a formula. Formula-based conditions failed validation on new versions.
- **Recheck after the vendor's connector is reconnected.** Configuration can be regenerated without your filter.
