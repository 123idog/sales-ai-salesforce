# 3. AI Writing to the CRM

An AI agent that only talks is a toy. The value comes when it writes back: summaries, statuses, interests, call scores. That's also where it can do the most damage.

---

## Finding the record: phone matching

**Symptom:** "the AI didn't document this call." About 1.5% of open opportunities were unreachable by the call-analysis pipeline.

**Cause:** a `LIKE '%<9 digits>%'` match fails when the stored number contains separators (spaces, dashes).

**Fix:** narrow in SOQL with an end-anchored `LIKE '%<last 4 digits>'` plus a `LIMIT`, then compare the last 9 digits in code after stripping non-digits. Almost all the missing records were recovered.

**Watch the limit:** an unanchored match returned a candidate count close to the limit, which is one step from silent truncation.

---

## A bot that changes status needs guards

**Symptom:** the bot marked leads *Unqualified* on the first turn, while the customer's timing was still unknown, and the status stayed after the conversation was handed to a rep. Over a hundred leads were affected before it was caught.

**Cause:** a "not soon enough" rule evaluated true when timing was **unknown**, and handoff had no exemption.

**Fix:**
- Disqualify only on **explicit** signals, never on missing information.
- Never disqualify once a rep is involved, or for high-value segments (e.g. large groups) that deserve a human look.
- On handoff, revert a status the **bot** set; never touch a status a **human** set.
- Write the **reason** into a notes field so a rep can see why.
- Put the rules in front of the sales team for approval before going live. It's their pipeline.

---

## Record who conducted the conversation

Sales managers want to know which deals the bot handled, which a human handled, and which both handled. A backfill classified historical leads from a marker in the AI summary into bot / bot + human / human.

- Be honest about what the label means: "human" really meant "no bot summary found".
- Fill only empty fields, so manual tags are never overwritten.

---

## Filter before you call the model

A scheduled job classified abandoned conversations with an LLM and marked the matching leads for follow-up. Moving the CRM match (open, unmarked leads only) **before** the LLM call cut model calls by about 70%. Cheap deterministic filters go first; the LLM only sees what can actually be acted on.

Related safety habits:
- Marks are added, never removed, by the automation.
- Live writes sit behind a single flag, off until the dry-run output has been reviewed.

---

## Merge, don't append

When an LLM extracts structured data from each call or chat into a CRM field, appending a dated block per interaction repeats the same facts over and over. Give the extractor the field's **current value** and ask for one merged, deduplicated state. Write only when non-empty, so a failed extraction never wipes data. Make sure the read query actually selects the field you're about to update, or it will look empty and get overwritten.
