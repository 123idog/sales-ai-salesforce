# 2. Lead Intake and Attribution

Every downstream system (bots, dialers, ad platforms, reports) depends on the lead being created right. Intake bugs are invisible until someone asks "where did our leads come from?"

---

## Normalise phones in *every* intake path

**Symptom:** malformed phone numbers kept arriving after "all" intake workflows were fixed: a leading `+`, a trunk `0` after the country code, a prefix in a separate field.

**Cause:** one lead-creating workflow had been missed, and reps also typed numbers by hand.

**Fix:**
- Scan the whole automation instance for **every** workflow that creates leads. Don't trust a list from memory.
- Use one market-aware normaliser: digits only, strip international `00`, fix the "country code + 0" leak, convert a local `0` to the country code.
- For human-entered leads, run a scheduled normaliser every few minutes.

Phone format matters because the messaging platform, the dialer and record matching all key on it (see [chapter 3](03-ai-writing-to-crm.md)).

---

## Hidden UTM fields carry stale defaults

**Symptom:** hundreds of leads were attributed to a campaign unrelated to the page they came from, and organic visits looked paid.

**Cause:** hidden `utm_*` fields on the form had a default value baked in.

**Fix:** read attribution **only from the page URL**, never from form defaults.

---

## Keep the entry point separate from the channel

Two different questions:
- **Channel:** how did they find us? (paid search, social, organic, referral)
- **Entry point:** which page or form did they use?

Mixing them in one field means a landing-page change rewrites your channel reports. Paid-search leads always get the paid-search source, whatever the page; the page or form identity goes in its own field. Write both in the **same request**, because the old source value may be the only evidence of the entry point.

When fixing history, update only the records you can attribute with certainty. In one retro fix, several hundred leads and opportunities were corrected, and a similar number of older leads were deliberately left empty rather than guessed.

---

## Store the search keyword only when it's real

**Symptom:** the keyword field contained static labels, and real keywords landed in the ad-group field.

**Fix:** save the keyword only when a match-type parameter is in the URL (that's how you know the ad platform filled it), and store the ad-group ID in its own field. Don't backfill records from before the tracking template changed; the data isn't there.

---

## Personalised first messages need a fallback chain

**Symptom:** a template greeting by first name had no first name for roughly 30% of leads. Separately, a name containing a quote character broke the JSON request body, so the opening message, and the rest of that run, never happened.

**Fix:**
- Fall back: first name → full name → last name → a generic greeting.
- Build request bodies with `JSON.stringify`, never by string concatenation of user-supplied values.
