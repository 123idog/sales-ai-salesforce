# 7. Analytics on CRM Data

Once AI and automation are writing to the CRM, management asks the CRM questions. The answers are only as good as the fields behind them.

---

## Check that the field holds data before analysing it

In one analysis:
- the **revenue** field was empty even on fully paid deals;
- a field whose name suggested a phone number was a boolean **consent** flag;
- the field that actually answered the question lived on a **different object**.

**Rule:** before any analysis, profile each field you'll use: fill rate, distinct values, a few sample records. Write down which object holds the truth.

---

## Cross-system matching

To join chat exports, call logs and CRM records, match on the **last 9 digits** of the phone number after stripping non-digits. This gave roughly a 98% match rate between a messaging export and the CRM, without fighting every country-code and formatting variant.

---

## Correlation is not a recommendation

Leads that already had a key prerequisite for purchase converted at roughly **4×** the rate of leads without it. That's a strong signal for prioritisation and for qualification questions, but it is **correlation**. It doesn't show that helping leads get the prerequisite would cause them to buy. Say that explicitly in the report, because the business will read it as a recommendation.

---

## State coverage with every number

Same rule as everywhere else: **"retrieved N of M"** before any figure, and the date window and record filter used. CRM data has hidden filters (converted records, record types, sharing rules for the integration user) that change the denominator.
