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

**PR link:** https://github.com/ascherj/pathreview/pull/921

**Branch:** `fix/147-resume-section-leading-whitespace`

**What you built:**
I fixed resume section detection so headers like Education, Skills, and Experience are still recognized when the text has leading whitespace. That matters because text copied or extracted from PDFs often keeps odd indentation. The fix lets the regex accept spaces or tabs at the start of a section line and removes the older patterns that were doing the same job less cleanly.

**Tests added or updated:**
`tests/unit/test_resume_parser.py` — I added `test_detect_sections_tab_indented` for tab-indented headers and `test_detect_sections_ignores_header_word_mid_sentence` to make sure regular sentences do not get treated like section titles. I also kept the Week 8 reproduction test, `test_detect_sections_with_leading_whitespace`. With the fix, the older tests `test_detect_sections`, `test_parse_single_column_resume_text`, and `test_parse_resume_no_work_experience` now pass too.

**Self-review confirmation:** [x] make check passes  [x] make test-unit passes
(The repo already had documented failures before I started: 54 failing unit tests, plus existing ruff, mypy, and black issues. After my change, the suite is 50 failing / 381 passing. My change fixes 4 tests and does not introduce new test, lint, type, or formatting failures in the files I touched.)

**Draft PR feedback received from:** none


## Week 10 — Iteration & reflection

### Reviewer feedback

**Feedback received:** [ ] Yes  [x] No — still awaiting review

**Summary of feedback:**
No reviewer or maintainer feedback came in on PR #921. (Reviewer feedback is not
a feature this term, so this is expected.)

**How you responded:**
No changes were required. I kept the PR open and my branch up to date.

---

### Reflection

**What was harder than you expected?**
The actual code change was tiny — a few characters added to one regex in
`_detect_sections()` — but almost everything around it took longer. The hardest
part was telling apart the failures I caused from the ones that were already in the
repo. When I first ran the tests there were 54 failures, and my instinct was that I
had broken something. Learning to record a baseline first, and to reason about
"did my change make it worse?" instead of "is everything green?", was harder than
writing the fix.

**What did you learn about working in a large codebase?**
You read a lot more code than you write. My change was about three lines, but I
spent most of the time understanding how the section-detection regex used `^` with
`re.MULTILINE`, and making sure I matched the project's conventions — Conventional
Commit messages, the pull request template, and the existing test style in
`tests/unit/`. On my own projects I can do whatever works; on someone else's
production code the fix also has to fit their patterns and not regress the rest of
the suite. The fork → branch → pull-request-against-upstream workflow, and getting
the base repository right, was also new to me.

**How did AI tools help — and where did they fall short?**
AI was most useful for orienting quickly: finding where sections were detected,
explaining what the regex was doing, and drafting the plan, tests, and PR
description. Where it fell short: at one point it produced a test that was indented
at the wrong level (a loose function instead of a class method), which would have
broken test collection — I caught that while reviewing before committing. It also
made claims about the code that I had to verify against the actual file, and
"it works" only counted once I ran `make test-unit` myself. AI sped up the typing,
but I still had to read the diff like a reviewer and run everything.

**What would you do differently if you started over?**
I would set up the environment and run the baseline checks on day one instead of
close to the deadline, and get comfortable with the git and PR workflow earlier so
the final submission was not rushed. I would also read generated code more carefully
before committing it — the wrong-indentation test would have cost me time if I had
not caught it.

**What are you most proud of from this module?**
Shipping a real, well-scoped pull request into an unfamiliar production codebase —
with tests and a clear description — and being able to prove my change fixed four
previously-failing tests without adding any new failures, in a repo that was already
full of pre-existing breakage. Keeping the change small and focused instead of
trying to "fix everything" is the part I'm happiest with.
