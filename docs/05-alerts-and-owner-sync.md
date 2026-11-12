# 5. Alerts and Owner Sync

Automation that helps reps has to reach them where they work, and must never undo what they did by hand.

---

## Hot-lead pop-ups that reps actually see

**Goal:** when a lead's score crosses a threshold, the team lead sees it **now**, not in a notification bell they check twice a day.

**Design:**
- A record-triggered Flow fires when the computed score crosses the threshold (a handful of leads per day, so it stays meaningful).
- It publishes a **high-volume platform event**.
- A small listener component in the app's utility bar shows a pop-up with a link to the lead.
- Recipients come from **public groups** (e.g. one per language team), so managers can change who gets alerts without a deploy.

**Limits worth stating up front:** the score was a computed field, so it can't be written directly to test; and the pop-up only appears while the CRM is open in a tab.

---

## Syncing owner from a messaging platform

Conversations are assigned to agents in the messaging platform; the CRM owner should follow. But reps also reassign leads in the CRM by hand.

**The trap:** comparing messaging-side owner with the **CRM** owner and "fixing" differences undoes every manual reassignment.

**Fix: sync changes, not states.**
- Store the **last-seen** messaging-side owner per contact.
- Sync only when the messaging-side owner differs from the last-seen value, i.e. it actually changed there.
- Seed the stored state before the first run, or the first run reassigns everyone.
- Map owners by email to exactly **one active** CRM user; skip anything ambiguous.
- PATCH, then read back. On failure, keep the previous last-seen value so the next run retries.

**Known gaps to document:** an assignment made before the lead exists in the CRM isn't retried, and leads converted to opportunities need their own path.

---

## Click-to-dial from automation

A dial API triggered from the CRM's telephony integration works, but returns an error while the rep is in post-call wrap-up. Check availability before dialling, and remember that calls are already logged as activities by the integration, so don't create duplicates.
