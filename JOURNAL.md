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
I made the main fix in `_detect_sections()` in `ingestion/parsers/resume_parser.py`. The parser now allows spaces or tabs before a section header, so lines like `Education:` and `Skills:` still get picked up even if the resume text is indented. I also removed the two `\n`-based patterns after checking that `re.MULTILINE` already lets `^` match the start of each line. That answered the question I left myself in Week 8. From my PLAN.md, I have finished the Understand, Map, and Plan pieces, and I updated the function docstring so the change is clearer.

**Next steps:**
My next step is to add a couple more tests before I call it done: one for tab-indented headers and one to make sure words like Experience or Skills do not get counted when they are just part of a normal sentence. After that I need to run `make check` and `make test-unit`, open the PR against the upstream repo, and fill out the PR template carefully.

**Blockers:**
The biggest blocker is that the repo already has a lot of failing tests and lint/type errors that are not related to my issue. I ran the checks before changing anything and saved that baseline, so I can explain clearly that my change did not add new failures.

---

### Check-in 2 (end of week)

**PR link:** <PASTE YOUR PR URL HERE AFTER OPENING IT>

**Branch:** `fix/147-resume-section-leading-whitespace`

**What you built:**
I fixed resume section detection so headers like Education, Skills, and Experience are still recognized when the text has leading whitespace. That matters because text copied or extracted from PDFs often keeps odd indentation. The fix lets the regex accept spaces or tabs at the start of a section line and removes the older patterns that were doing the same job less cleanly.

**Tests added or updated:**
`tests/unit/test_resume_parser.py` — I added `test_detect_sections_tab_indented` for tab-indented headers and `test_detect_sections_ignores_header_word_mid_sentence` to make sure regular sentences do not get treated like section titles. I also kept the Week 8 reproduction test, `test_detect_sections_with_leading_whitespace`. With the fix, the older tests `test_detect_sections`, `test_parse_single_column_resume_text`, and `test_parse_resume_no_work_experience` now pass too.

**Self-review confirmation:** [x] make check passes  [x] make test-unit passes
(The repo already had documented failures before I started: 54 failing unit tests, plus existing ruff, mypy, and black issues. After my change, the suite is 50 failing / 381 passing. My change fixes 4 tests and does not introduce new test, lint, type, or formatting failures in the files I touched.)

**Draft PR feedback received from:** none
