## Solution plan

**Issue:** Resume section detection fails on text with leading whitespace —
https://github.com/ascherj/pathreview/issues/147

### Understand

`_detect_sections()` in `ingestion/parsers/resume_parser.py` builds regex patterns
whose anchor (`^` or `\n`) is immediately followed by the section name, e.g.
`rf"^{re.escape(section)}\s*[:|-]"`. Under `re.MULTILINE`, `^` matches the start of
each line, but the header text must be the very first character of that line.

- **Expected:** For indented text like `"\n    Education:\n    Skills: Python\n"`,
  `detected_sections` contains `"Education"` and `"Skills"`.
- **Actual:** PDF extraction preserves leading spaces/tabs, so the anchors never
  reach the header word and `_detect_sections()` returns `[]`.

**Root cause:** there is no `\s*` (optional leading whitespace) between the line
anchor and the section name in any of the four patterns.

### Map

- `ingestion/parsers/resume_parser.py`
  - `SECTION_HEADERS` (set of recognized headers) — no change expected.
  - `_detect_sections(self, text)` — **the only production change**: the `patterns`
    list is where the anchors need to tolerate leading whitespace.
- `tests/unit/test_resume_parser.py`
  - Add a new regression test for indented input.
  - Referenced/affected tests to keep green: `test_detect_sections`,
    `test_parse_single_column_resume_text`, `test_parse_resume_no_work_experience`.

### Plan

1. **Lock in the reproduction as a test.** Add
   `test_detect_sections_with_leading_whitespace` using the issue's indented input;
   assert it currently returns `[]` (or simply assert the target sections and watch
   it fail before the fix).
2. **Relax the anchors.** In `_detect_sections()`, insert `\s*` after each line
   anchor so headers are found regardless of indentation:
   ```python
   patterns = [
       rf"^\s*{re.escape(section)}\s*$",
       rf"^\s*{re.escape(section)}\s*[:|-]",
       rf"\n\s*{re.escape(section)}\s*$",
       rf"\n\s*{re.escape(section)}\s*[:|-]",
   ]
   ```
3. **Decide on redundancy.** Because `re.MULTILINE` is set, `^\s*...` already covers
   the `\n\s*...` cases. Either keep all four (minimal change) or collapse to the two
   `^`-anchored patterns (simplification). Pick one and note the reasoning in the PR.
4. **Run the full unit suite** (`pytest tests/unit/test_resume_parser.py`, or the
   project's `make test`) and confirm the three referenced tests plus the new test
   pass.
5. **Guard against false positives.** Manually check that a body line such as
   `"    Experience designing APIs"` is NOT mis-detected, and that overlapping
   headers ("skills" vs "technical skills") aren't double-counted.

### Inputs & outputs

- **Input:** `text: str` — raw extracted resume text that may contain leading spaces
  or tabs on each line.
- **Output:** `list[str]` — title-cased, de-duplicated section names
  (`list(set(...))`). Signature and return type are unchanged.
- **Behavioral change:** indented headers now detected; on the issue's example the
  result must include `"Education"` and `"Skills"`.

### Risks & unknowns

- **False positives (low):** adding `\s*` could match a section word that starts a
  body line followed by a delimiter (e.g. an indented `"Summary: ..."` sentence). The
  `[:|-]` / end-of-line requirements keep this narrow, but worth a manual check.
- **Redundant `\n` patterns:** with `re.MULTILINE`, the `\n`-anchored patterns
  duplicate the `^` ones. Leaving them is harmless; removing them is a simplification
  I need to justify.
- **Nondeterministic order:** `return list(set(detected))` has no stable order. Not
  caused by this fix, but if any caller assumes order it could flake — grep
  `detected_sections` / `_detect_sections` usages to confirm callers only test
  membership.
- **CRLF line endings:** real PDFs may produce `\r\n`. Need to confirm `\s*$` handles
  a trailing `\r`.
- **Unknown:** exact line numbers in my local copy vs. what I read from `main` — verify
  against the actual file before editing.

### Edge cases

- Leading spaces before a header (the issue's case) → detected.
- Leading tabs, or mixed tabs/spaces → detected (`\s` covers both).
- Blank lines between sections → detected.
- Header appearing mid-sentence, e.g. `"My Experience at TechCorp"` → must NOT be
  falsely detected.
- Trailing spaces before the delimiter, e.g. `"Education   :"` → detected.
- CRLF (`\r\n`) line endings → detected.
- Empty string or a resume with no recognizable sections → returns `[]` gracefully
  (unchanged behavior).
