# Module 3 Journal

## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/147

**Issue title:** Resume section detection fails on text with leading whitespace

**Tier:** [x] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**
<!-- DRAFT — rewrite this in your own words before submitting; the grader looks for your own voice, not a paraphrase of mine. -->
The resume parser's `_detect_sections()` function in `ingestion/parsers/resume_parser.py`
is responsible for figuring out which sections a resume contains (Education, Skills,
Experience, etc.). It matches section headers with regex patterns that require the header
word to sit right at the start of a line. When text extracted from a PDF keeps its leading
indentation (for example, `\n    Education:`), the spaces before the word stop the patterns
from matching, so the function returns an empty list and reports zero detected sections.
A successful fix would let the detection tolerate leading whitespace so that normally
formatted, indented resumes are parsed correctly, and would make the three failing tests
(`test_parse_single_column_resume_text`, `test_parse_resume_no_work_experience`,
`test_detect_sections`) pass.

**Branch name:** fix/147-resume-section-leading-whitespace

**Setup confirmation:** [ ] App runs locally at localhost:5173

**Cohort ledger:** [ ] Issue added to cohort ledger

### Selection notes — "Is this issue right for me?"
<!-- Work through the course checklist and note your own reasoning here. Starter points: -->
- **Scope is contained:** the bug lives in a single function in one file
  (`_detect_sections()` in `ingestion/parsers/resume_parser.py`), so the blast radius is small.
- **Reproducible:** the issue includes a copy-paste reproduction snippet with observed vs.
  expected output.
- **Testable:** three existing tests already cover this behavior, so I can verify a fix
  objectively rather than guessing.
- **Right difficulty:** labeled `tier-1` / `good first issue` — appropriate for a first
  contribution to a large codebase.
- **Understood:** the root cause (regex anchored to line start, no allowance for leading
  whitespace) is clear to me, which is what this checklist asks for.
