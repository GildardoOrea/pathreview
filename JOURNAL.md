# Module 3 Journal

## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/147

**Issue title:** Resume section detection fails on text with leading whitespace

**Tier:** [x] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**
The resume parser uses the `_detect_sections()` function in `ingestion/parsers/resume_parser.py` to identify sections such as Education, Skills, and Experience. Right now, the function expects each section title to appear directly at the beginning of a line. When text extracted from a resume contains spaces before a section title, the parser does not recognize it and reports that the resume has no sections. The fix would allow the parser to ignore leading whitespace and correctly identify section titles in indented resume text.

**Branch name:** `fix/147-resume-section-leading-whitespace`

**Setup confirmation:** [ ] App runs locally at localhost:5173

**Cohort ledger:** [ ] Issue added to cohort ledger

### Selection notes — “Is this issue right for me?”

I chose this issue because it is a Tier 1 problem with a clear scope and expected result. The issue appears to be limited to the resume section detection function and the regular expressions it uses to recognize section titles. It also includes examples of the failing input and identifies tests that can be used to confirm the solution. This makes the problem realistic for me to complete while still helping me practice reading an unfamiliar codebase, working with regular expressions, and testing a change.
