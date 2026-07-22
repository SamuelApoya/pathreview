## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/146

**Issue title:** PII scrubber fails to redact parenthesized US phone numbers

**Tier:** [x] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**
The PII scrubber in `safety/pii_scrubber.py` is supposed to detect and redact
US phone numbers before review content is stored or returned, but its
regex only matches dashed/dotted formats like 555-123-4567. Parenthesized
formats like (555) 123-4567 pass through both `scrub()` and `detect()`
completely unredacted, because the pattern anchors on a `\b` word boundary
that doesn't match when the number starts with a non-word character like
`(`. This is a real gap since parenthesized formatting is one of the most
common ways US phone numbers are written, so a successful fix will update
the regex to recognize that format alongside the existing ones, verified
by the four currently-failing tests in `tests/unit/test_pii_scrubber.py`.

**Issue selection reasoning ("Is this right for me?" checklist):**
I can explain the bug without re-reading the issue: the phone_us regex in
safety/pii_scrubber.py anchors on \b, which fails to match when a number
opens with a non-word character like `(`, so parenthesized formats like
(555) 123-4567 pass through scrub() and detect() unredacted while dashed
formats are caught correctly. I opened safety/pii_scrubber.py and located
the phone_us pattern, and read through tests/unit/test_pii_scrubber.py,
including the four currently-failing tests this issue references. This is
Tier 1, so the tier is a good match
— the fix is scoped to a single regex in a single file. Several other
students are also working this issue, but claims are non-exclusive.
I estimate 2–4 hours of focused work,
comfortably within the Week 8–9 window, and the issue has no listed
blockers or dependencies.

**Branch name:** fix/146-parenthesized-us-phone-pii

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger