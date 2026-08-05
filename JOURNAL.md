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



## Week 8 — Reproduction & solution planning

**Reproduction commit link:** https://github.com/SamuelApoya/pathreview/commit/0362f2a

**Reproduction summary:**
I reproduced the issue in a Python shell using `PIIScrubber().scrub()` and
`.detect()` on "Call me at (555) 123-4567" — `scrub()` left the number
completely untouched while correctly redacting a dashed-format number in the
same string, and `detect()` returned zero PII items. Testing the regex
directly confirmed the root cause is the separator pattern's lack of
whitespace support

**PLAN.md link:** https://github.com/SamuelApoya/pathreview/blob/fix/146-parenthesized-us-phone-pii/PLAN.md

**Walkthrough video (recommended):** N/A

**Blockers or open questions:**
Still deciding whether widening the separator to accept whitespace introduces
new false positives elsewhere in `detect()`



## Week 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:**
Implemented the fix for issue #146 — widened the phone_us regex separators
in safety/pii_scrubber.py to accept whitespace, not just dash/dot, so
formats like (555) 123-4567 and +1 555 123 4567 are now redacted correctly.
Also fixed a secondary bug my own fix introduced, where the leading `(` or
`+` character was dropped from the redacted/detected output. Both changes
are committed and pushed. Added two new unit tests
(test_parenthesized_phone_fully_redacted and
test_parenthesized_phone_fully_detected) verifying scrub() and detect()
both handle the parenthesized format correctly, including the opening
paren. All sub-tasks from my Week 8 PLAN.md are complete.

Ran make check and make test-unit across the whole repo to check for
regressions. make test-unit shows 49 failed / 381 passed; only one
failure (test_mixed_pii_and_text) touches my file, and I confirmed by
diffing against main that it's a pre-existing failure from an unrelated
street_address regex bug, not caused by my change. The other 48 failures
are in unrelated modules (review service, resume parser, tech detector,
bias detector, etc.) I never touched. make check similarly shows
pre-existing ruff/mypy issues confined to files and lines I didn't modify.

**Next steps:**
Open the PR with the full template filled in, documenting the confirmed
pre-existing failures in Notes for Reviewers.

**Blockers:**
None.

---

### Check-in 2 (end of week)

**PR link:** https://github.com/ascherj/pathreview/pull/986

**Branch:** fix/146-parenthesized-us-phone-pii

**What you built:**
Fixed the phone_us regex in safety/pii_scrubber.py so it redacts US phone
numbers separated by whitespace (e.g. (555) 123-4567, +1 555 123 4567),
not just dashes/dots, and fixed a secondary bug where the leading `(` or
`+` character was dropped from the redacted output.

**Tests added or updated:**
tests/unit/test_pii_scrubber.py — added test_parenthesized_phone_fully_redacted
and test_parenthesized_phone_fully_detected, covering both scrub() and
detect() on the parenthesized phone number format.

**Self-review confirmation:** [x] make check passes  [x] make test-unit passes

**Draft PR feedback received from:** none