# Module 3 Journal

## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/147

**Issue title:** Resume section detection fails on text with leading whitespace

**Tier:** [x] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**  
The resume parser uses the `_detect_sections()` function in `ingestion/parsers/resume_parser.py` to identify sections such as Education, Skills, and Experience. Right now, the function expects each section title to appear directly at the beginning of a line. When text extracted from a resume contains spaces before a section title, the parser does not recognize it and reports that the resume has no sections. The fix would allow the parser to ignore leading whitespace and correctly identify section titles in indented resume text.

**Branch name:** `fix/147-resume-section-leading-whitespace`

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger

### Selection notes — “Is this issue right for me?”

I chose this issue because it is a Tier 1 problem with a clear scope and expected result. The issue appears to be limited to the resume section detection function and the regular expressions it uses to recognize section titles. It also includes examples of the failing input and identifies tests that can be used to confirm the solution. This makes the problem realistic for me to complete while still helping me practice reading an unfamiliar codebase, working with regular expressions, and testing a change.

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** https://github.com/GildardoOrea/pathreview/commit/599df3e

**Reproduction summary:**  
I reproduced the issue by adding a unit test called `test_detect_sections_with_leading_whitespace` in `tests/unit/test_resume_parser.py` using the same indented resume text from the issue. The test fails because `_detect_sections()` returns an empty list, confirming that the regex patterns in `ingestion/parsers/resume_parser.py` do not recognize section titles when there is whitespace at the beginning of the line.

**PLAN.md link:** https://github.com/GildardoOrea/pathreview/blob/fix/147-resume-section-leading-whitespace/PLAN.md

**Walkthrough video (recommended):**

**Blockers or open questions:**  
I am still deciding whether I should keep the existing patterns that use `\n` or simplify the list to only use the `^` patterns with `re.MULTILINE`, since they may become redundant. I plan to compare both approaches and make the final decision during Week 9.


## Week 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:**
I implemented the fix in `_detect_sections()` in `ingestion/parsers/resume_parser.py`. I added `[ \t]*` after the `^` line anchor so section headers are detected even when the line begins with spaces or tabs, and I removed the two `\n`-anchored patterns because `re.MULTILINE` already makes `^` match the start of every line (this resolved the open question from Week 8). From my PLAN.md, the Understand, Map, and Plan sub-tasks are done, and I updated the function's docstring.

**Next steps:**
Add tests for tab-indented headers and for the false-positive case (a section word inside a normal sentence), run `make check` and `make test-unit`, then open the PR against upstream and fill in the template.

**Blockers:**
The repository has many pre-existing failing tests and lint/type errors unrelated to my issue. I recorded the baseline before making changes so I can show my change introduces no new failures.

---

### Check-in 2 (end of week)

**PR link:** <PASTE YOUR PR URL HERE AFTER OPENING IT>

**Branch:** `fix/147-resume-section-leading-whitespace`

**What you built:**
A fix to resume section detection so that section headers (Education, Skills, Experience, etc.) are recognized even when the text has leading whitespace, which commonly happens with PDF-extracted resumes. The fix allows optional spaces/tabs after the line anchor in the detection regexes and removes now-redundant patterns.

**Tests added or updated:**
`tests/unit/test_resume_parser.py` — added `test_detect_sections_tab_indented` (tab-indented headers) and `test_detect_sections_ignores_header_word_mid_sentence` (guards against false positives), in addition to the Week 8 reproduction test `test_detect_sections_with_leading_whitespace`. My change also turned the previously failing `test_detect_sections`, `test_parse_single_column_resume_text`, and `test_parse_resume_no_work_experience` green.

**Self-review confirmation:** [x] make check passes  [x] make test-unit passes
(This codebase has documented pre-existing failures — baseline was 54 failing unit tests, plus pre-existing ruff/mypy/black issues. After my change the suite is 50 failing / 381 passing: my change fixes 4 tests and introduces zero new failures, lint, type, or formatting errors in the files I touched.)

**Draft PR feedback received from:** none
