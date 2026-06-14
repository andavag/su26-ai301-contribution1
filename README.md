# Contribution 1: Add compatibility test for $minN array element

**Contribution Number:** 1  
**Student:** Andranik Avagyan  
**Issue:** https://github.com/documentdb/functional-tests/issues/201  
**Project Fork:** https://github.com/andavag/functional-tests  
**Working Branch:** https://github.com/andavag/functional-tests/tree/fix-issue-201  
**Status:** Phase II Complete

---

## Why I Chose This Issue

I chose this issue because it is a good first open-source contribution: the scope is focused, the project has clear Python/pytest contribution documentation, and the work improves compatibility coverage for a specific MongoDB aggregation expression operator.

This issue also gives me practice navigating an established test framework, following existing test patterns, and writing a test that documents expected database behavior.

---

## Understanding the Issue

### Problem Description

Issue #201 asks for additional compatibility test coverage for the `$minN` array expression operator in the `core / operator / expressions / array` category. This is a "second pass" issue. Upstream `main` already contains a basic smoke test for `$minN` array element behavior, so the contribution should add more specific coverage beyond the happy-path smoke case.

### Expected Behavior

`$minN` should accept syntax like:

```javascript
{ $minN: { n: <expression>, input: <array expression> } }
```

It should return the minimum `n` elements from the input array. MongoDB's documentation states that `n` resolves to a positive integer and `input` resolves to the array to read from.

### Current Behavior

The repository currently has this smoke test:

`documentdb_tests/compatibility/tests/core/operator/expressions/array/minN-array-element/test_smoke_expression_minN-array-element.py`

That test verifies a basic numeric array case:

```python
{"$project": {"minTwo": {"$minN": {"n": 2, "input": "$values"}}}}
```

The issue is still open, so the missing work is likely deeper second-pass coverage such as null handling, `n` larger than the number of usable values, duplicate values, or type/error behavior.

### Affected Components

- `documentdb_tests/compatibility/tests/core/operator/expressions/array/minN-array-element/test_smoke_expression_minN-array-element.py`
- `documentdb_tests/compatibility/tests/core/operator/expressions/array/maxN-array-element/test_smoke_expression_maxN-array-element.py` as the closest sibling pattern
- `documentdb_tests/framework/assertions.py` for `assertSuccess`
- `documentdb_tests/framework/executor.py` for `execute_command`
- `docs/testing/TEST_FORMAT.md`
- `docs/testing/FOLDER_STRUCTURE.md`

---

## Reproduction Process

### Environment Setup

Local environment used for Phase II:

- OS: Windows 11
- Repo path: `D:\Andranik\my coding tests\codepath\functional-tests`
- Fork remote: `https://github.com/andavag/functional-tests.git`
- Branch created and pushed: `fix-issue-201`
- Python in `.venv`: `Python 3.14.3`
- Installed project tools in `.venv`: `pytest`, `pymongo`, `pytest-xdist`, `pytest-json-report`, `pytest-timeout`, `pytest-cov`, `black`, `isort`, `flake8`, `mypy`, `pre-commit`
- Installed pre-commit hooks: `pre-commit`, `prepare-commit-msg`, and `pre-push`

Project prerequisites from `CONTRIBUTING.md`:

- Python 3.9 or higher
- Git
- Access to a DocumentDB or MongoDB instance for functional test execution
- Development dependencies from `requirements-dev.txt`
- Pre-commit hooks installed with:

```bash
pre-commit install -t pre-commit -t prepare-commit-msg -t pre-push
```

### Steps to Reproduce

1. Clone the fork and enter the project:

```bash
git clone https://github.com/andavag/functional-tests.git
cd functional-tests
```

2. Create and publish the working branch:

```bash
git checkout main
git pull origin main
git checkout -b fix-issue-201
git push -u origin fix-issue-201
```

3. Inspect the current `$minN` array expression test:

```bash
Get-Content documentdb_tests/compatibility/tests/core/operator/expressions/array/minN-array-element/test_smoke_expression_minN-array-element.py
```

4. Collect the focused test:

```bash
..\.venv\Scripts\python.exe -m pytest documentdb_tests\compatibility\tests\core\operator\expressions\array\minN-array-element\test_smoke_expression_minN-array-element.py --collect-only
```

5. Try to run the focused test against local MongoDB:

```bash
..\.venv\Scripts\python.exe -m pytest documentdb_tests\compatibility\tests\core\operator\expressions\array\minN-array-element\test_smoke_expression_minN-array-element.py --connection-string "mongodb://localhost:27017/?serverSelectionTimeoutMS=2000" --engine-name mongodb
```

### Reproduction Evidence

- `pytest --collect-only` collected 1 test successfully from the `$minN` array expression test file.

---

## Solution Approach

### Analysis

The root cause is missing second-pass compatibility coverage, not missing framework support. The test folder and basic smoke file already exist. The contribution should add a focused behavioral test in the existing `$minN` array expression test area.

The best candidate behavior to test is null handling with `n` larger than the number of non-null values. This is useful because it exercises behavior beyond the existing happy path while staying a positive result test that can use `assertSuccess`.

### Proposed Solution

Add one new test function to:

`documentdb_tests/compatibility/tests/core/operator/expressions/array/minN-array-element/test_smoke_expression_minN-array-element.py`

The new test should insert documents with arrays containing numbers and `None`, run an aggregation `$project` using `$minN`, and assert that the output contains the minimum non-null values in sorted order.

### Implementation Plan

Using the UMPIRE framework:

**Understand:** The issue asks for more compatibility coverage for `$minN` as an array expression operator. The existing smoke test only covers simple numeric arrays.

**Match:** The existing `$minN` smoke test and sibling `$maxN` array smoke test both use `collection.insert_many`, `execute_command`, an aggregation `runCommand`, and `assertSuccess`.

**Plan:**

1. Add a second test function in the existing `$minN` array expression file.
2. Use `collection.insert_many` with arrays such as `[None, 8, 2, None, 5]`.
3. Run:

```python
{"$project": {"minValues": {"$minN": {"n": 5, "input": "$values"}}}}
```

4. Assert expected output such as `[2, 5, 8]`, showing null values are excluded and `n` can be larger than the remaining values.
5. Keep one assertion in the test and use `assertSuccess`.

**Implement:** Phase III will implement this on branch `fix-issue-201`.

**Review:** Before submitting, check that the test:

- Has a descriptive `test_` function name
- Has a docstring
- Uses `execute_command`
- Uses framework assertion helpers instead of plain `assert`
- Keeps one assertion in the test function
- Stays in the correct feature folder

**Evaluate:** Verify with:

```bash
..\.venv\Scripts\python.exe -m pytest documentdb_tests\compatibility\tests\core\operator\expressions\array\minN-array-element\test_smoke_expression_minN-array-element.py --collect-only
```

Then run against a real MongoDB or DocumentDB instance:

```bash
..\.venv\Scripts\python.exe -m pytest documentdb_tests\compatibility\tests\core\operator\expressions\array\minN-array-element\test_smoke_expression_minN-array-element.py --connection-string "mongodb://localhost:27017" --engine-name mongodb
```

Finally run quality checks:

```bash
pre-commit run --all-files
```

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
