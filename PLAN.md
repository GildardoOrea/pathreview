## Solution plan

**Issue:** Resume section detection fails on text with leading whitespace  
https://github.com/ascherj/pathreview/issues/147

### Understand

The `_detect_sections()` function in `ingestion/parsers/resume_parser.py` uses regular expressions to identify resume sections such as Education, Skills, and Experience. The current patterns expect the section name to appear directly at the beginning of a line.

The expected behavior is for the function to recognize section titles even when spaces or tabs appear before them. For example, resume text containing indented `Education:` and `Skills:` lines should return both sections.

The actual behavior is that `_detect_sections()` returns an empty list because the spaces before the section names prevent the regular expressions from matching them.

The root cause is that the current patterns do not allow optional whitespace between the beginning of the line and the section name.

### Map

The main files involved are:

- `ingestion/parsers/resume_parser.py`
  - `_detect_sections()` contains the regular expressions that need to be updated.
  - `SECTION_HEADERS` contains the recognized section names, but I do not expect to change it.

- `tests/unit/test_resume_parser.py`
  - This file contains the existing tests for resume section detection.
  - I added `test_detect_sections_with_leading_whitespace` to reproduce the issue.
  - I will use the existing tests to make sure the fix does not break the current behavior.

### Plan

1. Keep the new test that uses resume text with spaces before `Education:` and `Skills:`. The test should expect both sections to be detected and should fail before the fix is applied.

2. Update the regex patterns inside `_detect_sections()` so they allow optional spaces or tabs before a section name.

3. Review whether the patterns that begin with `\n` are still necessary. Since the function uses `re.MULTILINE`, the patterns beginning with `^` may already cover section titles at the beginning of every line.

4. Run the tests in `tests/unit/test_resume_parser.py` and confirm that the new test and the existing resume parser tests pass.

5. Test additional examples to make sure normal sentences containing words such as Experience or Skills are not incorrectly detected as section titles.

### Inputs & outputs

The input is a string containing text extracted from a resume. Some lines may begin with spaces or tabs because of the resume formatting or the way text was extracted from a PDF.

The output is a list containing the section names found in the resume.

The function signature and return type will not change. The behavior change is that indented section titles will now be recognized. For the example from the issue, the result should include `Education` and `Skills`.

### Risks & unknowns

One possible risk is creating false matches. Allowing whitespace before a section title could cause a normal body line to be detected as a section if it begins with a recognized section name. The rest of the regular expression should reduce this risk by requiring a delimiter or the end of the line, but I still need to test it.

I also need to decide whether to keep all four existing patterns or remove the two patterns that begin with `\n`. Keeping them would be a smaller change, while removing them could make the code simpler because `re.MULTILINE` allows `^` to match the beginning of each line.

Another question is whether to use `\s*` or a pattern that only allows spaces and tabs. Since `\s` can also match line breaks, a more specific pattern such as `[ \t]*` may be safer. I will compare both options before making the final change.

### Edge cases

The fix should correctly handle:

- Section titles with leading spaces
- Section titles with leading tabs
- Section titles with both spaces and tabs
- Section titles with trailing spaces
- Section titles followed by a colon, vertical bar, or hyphen
- Blank lines between resume sections
- Windows line endings using `\r\n`
- Empty resume text
- Resume text with no recognized section titles

The fix should not detect a section when the section word appears in the middle of a normal sentence, such as `My Experience at TechCorp`.
