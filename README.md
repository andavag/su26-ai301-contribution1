# Contribution 1: Add compatibility test for $minN array element

**Contribution Number:** 1  
**Student:** Andranik Avagyan  
**Issue:** https://github.com/documentdb/functional-tests/issues/201  
**Project Fork:** https://github.com/andavag/functional-tests  
**Working Branch:** https://github.com/andavag/functional-tests/tree/fix-issue-201  
**Status:** Phase IV Complete - PR Submitted and Local Verification Passed

---

## Why I Chose This Issue

I chose this issue because it is a good first open-source contribution: the scope is focused, the project has clear Python/pytest contribution documentation, and the work improves compatibility coverage for a specific MongoDB aggregation expression operator.

This issue also gives me practice navigating an established test framework, following existing test patterns, and writing tests that document expected database behavior.

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

### Starting Behavior

The repository already had this smoke test:

`documentdb_tests/compatibility/tests/core/operator/expressions/array/minN-array-element/test_smoke_expression_minN-array-element.py`

That test verifies a basic numeric array case:

```python
{"$project": {"minTwo": {"$minN": {"n": 2, "input": "$values"}}}}
```

The missing second-pass coverage included deeper behavior such as null handling, `n` larger than the number of usable values, and invalid `n` values.

### Affected Components

- `documentdb_tests/compatibility/tests/core/operator/expressions/array/minN-array-element/test_minN-array-element_null_filtering.py`
- `documentdb_tests/compatibility/tests/core/operator/expressions/array/minN-array-element/test_minN-array-element_n_validation.py`
- `documentdb_tests/framework/error_codes.py`
- `documentdb_tests/compatibility/tests/core/operator/expressions/array/minN-array-element/test_smoke_expression_minN-array-element.py` as the existing baseline smoke test, left unchanged
- `documentdb_tests/compatibility/tests/core/operator/expressions/array/maxN-array-element/test_smoke_expression_maxN-array-element.py` as the closest sibling pattern
- `documentdb_tests/framework/assertions.py` for `assertSuccess` and `assertFailureCode`
- `documentdb_tests/framework/executor.py` for `execute_command`
- `docs/testing/TEST_FORMAT.md`
- `docs/testing/FOLDER_STRUCTURE.md`
- `docs/testing/TEST_COVERAGE.md`

---

## Reproduction Process

### Environment Setup

Local environment used for Phase II and Phase III:

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

Important setup note: dependency installation is already done in the local `.venv`. Functional execution requires a running MongoDB or DocumentDB instance. MongoDB was installed locally but the `MongoDB` Windows service was initially stopped; after starting it, the focused tests passed against `localhost:27017`.

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

3. Inspect the existing `$minN` array expression smoke test:

```bash
Get-Content documentdb_tests/compatibility/tests/core/operator/expressions/array/minN-array-element/test_smoke_expression_minN-array-element.py
```

4. Collect the existing smoke test:

```bash
..\.venv\Scripts\python.exe -m pytest documentdb_tests\compatibility\tests\core\operator\expressions\array\minN-array-element\test_smoke_expression_minN-array-element.py --collect-only
```

5. Run the new focused tests against local MongoDB:

```bash
..\.venv\Scripts\python.exe -m pytest documentdb_tests\compatibility\tests\core\operator\expressions\array\minN-array-element\test_minN-array-element_null_filtering.py --connection-string "mongodb://localhost:27017/?serverSelectionTimeoutMS=2000" --engine-name mongodb

..\.venv\Scripts\python.exe -m pytest documentdb_tests\compatibility\tests\core\operator\expressions\array\minN-array-element\test_minN-array-element_n_validation.py --connection-string "mongodb://localhost:27017/?serverSelectionTimeoutMS=2000" --engine-name mongodb
```

### Reproduction Evidence

- Initial `pytest --collect-only` collected 1 test successfully from the existing `$minN` array expression smoke test file.
- Initial MongoDB execution failed during fixture setup because the local MongoDB service was not running:

```text
ConnectionError: Cannot connect to mongodb at mongodb://localhost:27017/?serverSelectionTimeoutMS=2000
```

- After running `Start-Service MongoDB`, the local MongoDB server was reachable on `localhost:27017`.
- The null-filtering test collected successfully and passed against local MongoDB.
- The `n=0` validation test collected successfully and passed against local MongoDB.
- The `n=0` validation error code `5787908` was confirmed by running the invalid `$minN` array expression against local MongoDB and reading the returned server error code.
- Upstream `main` already contains the basic `$minN` array smoke test.
- GitHub issue #201 is still open as of June 14, 2026 and is titled "Add compatibility test for $minN (array element) (second pass)".

---

## Solution Approach

### Analysis

The root cause is missing second-pass compatibility coverage, not missing framework support. The test folder and basic smoke file already exist. The contribution should add focused behavioral tests in the existing `$minN` array expression test area.

The implemented coverage adds two second-pass behaviors:

1. Null filtering with `n` larger than the number of returned non-null values.
2. Validation that `n=0` is rejected because MongoDB requires `n` to resolve to a positive integer.

### Implemented Solution

Added two new focused test files instead of modifying the existing smoke test:

- `documentdb_tests/compatibility/tests/core/operator/expressions/array/minN-array-element/test_minN-array-element_null_filtering.py`
- `documentdb_tests/compatibility/tests/core/operator/expressions/array/minN-array-element/test_minN-array-element_n_validation.py`

The null-filtering test inserts documents with arrays containing numbers and `None`, runs an aggregation `$project` using `$minN`, and asserts that the output contains the minimum non-null values in sorted order.

The `n` validation test inserts one document, runs `$minN` with `n: 0`, and uses `assertFailureCode` to verify MongoDB returns error code `5787908`.

### Implementation Plan

Using the UMPIRE framework:

**Understand:** The issue asks for more compatibility coverage for `$minN` as an array expression operator. The existing smoke test only covers simple numeric arrays.

**Match:** The existing `$minN` smoke test and sibling `$maxN` array smoke test both use `collection.insert_many`, `execute_command`, an aggregation `runCommand`, and `assertSuccess`.

**Plan:**

1. Keep the existing `$minN` smoke test unchanged.
2. Add a separate null-filtering test file.
3. Use `collection.insert_many` with arrays containing `None` and numbers.
4. Run:

```python
{"$project": {"minValues": {"$minN": {"n": 5, "input": "$values"}}}}
```

5. Assert expected output such as `[2, 5, 8]`, showing null values are excluded and `n` can be larger than the remaining values.
6. Add a separate `n` validation test file.
7. Add `N_ARRAY_ELEMENT_NON_POSITIVE_N_ERROR = 5787908` to `documentdb_tests/framework/error_codes.py`.
8. Use `assertFailureCode` to verify `$minN` rejects `n: 0`.

**Implement:** Phase III implemented these changes locally on branch `fix-issue-201`.

**Review:** Before submitting, check that the tests:

- Have descriptive `test_` function names
- Have docstrings
- Use `execute_command`
- Use framework assertion helpers instead of plain `assert`
- Keep one assertion in each test function
- Stay in the correct feature folder
- Keep the existing smoke test unchanged

**Evaluate:** Verify with:

```bash
..\.venv\Scripts\python.exe -m pytest documentdb_tests\compatibility\tests\core\operator\expressions\array\minN-array-element\test_minN-array-element_null_filtering.py --collect-only

..\.venv\Scripts\python.exe -m pytest documentdb_tests\compatibility\tests\core\operator\expressions\array\minN-array-element\test_minN-array-element_n_validation.py --collect-only
```

Then run against a real MongoDB or DocumentDB instance:

```bash
..\.venv\Scripts\python.exe -m pytest documentdb_tests\compatibility\tests\core\operator\expressions\array\minN-array-element\test_minN-array-element_null_filtering.py --connection-string "mongodb://localhost:27017/?serverSelectionTimeoutMS=2000" --engine-name mongodb

..\.venv\Scripts\python.exe -m pytest documentdb_tests\compatibility\tests\core\operator\expressions\array\minN-array-element\test_minN-array-element_n_validation.py --connection-string "mongodb://localhost:27017/?serverSelectionTimeoutMS=2000" --engine-name mongodb
```

Finally run quality checks through the project virtual environment:

```bash
..\.venv\Scripts\python.exe -m black --check documentdb_tests\compatibility\tests\core\operator\expressions\array\minN-array-element\test_minN-array-element_null_filtering.py documentdb_tests\compatibility\tests\core\operator\expressions\array\minN-array-element\test_minN-array-element_n_validation.py documentdb_tests\framework\error_codes.py
..\.venv\Scripts\python.exe -m isort --check-only documentdb_tests\compatibility\tests\core\operator\expressions\array\minN-array-element\test_minN-array-element_null_filtering.py documentdb_tests\compatibility\tests\core\operator\expressions\array\minN-array-element\test_minN-array-element_n_validation.py documentdb_tests\framework\error_codes.py
..\.venv\Scripts\python.exe -m flake8 documentdb_tests\compatibility\tests\core\operator\expressions\array\minN-array-element\test_minN-array-element_null_filtering.py documentdb_tests\compatibility\tests\core\operator\expressions\array\minN-array-element\test_minN-array-element_n_validation.py documentdb_tests\framework\error_codes.py
```

---

## Testing Strategy

### Focused Tests

- [x] Add `$minN` array expression test for arrays containing `null` values.
- [x] Verify the result excludes `null` and returns the minimum non-null values.
- [x] Verify the test still behaves correctly when `n` is greater than the number of returned values.
- [x] Add `$minN` array expression validation test for `n=0`.
- [x] Verify invalid `n=0` fails with error code `5787908`.

### Manual / Local Validation

- [x] Confirmed the existing test file collects successfully.
- [x] Confirmed local execution needs a running MongoDB or DocumentDB instance.
- [x] Started the local MongoDB Windows service.
- [x] Ran the null-filtering `$minN` test file against local MongoDB.
- [x] Ran the `n` validation `$minN` test file against local MongoDB.
- [x] Ran focused `black`, `isort`, and `flake8` checks through `.venv`.
- [x] Ran full `mypy documentdb_tests/ --no-site-packages` through `.venv`.
- [ ] Run `pre-commit run --all-files` successfully through the system hook environment.

---

## Implementation Notes

### Phase II Progress

- Read `CONTRIBUTING.md`, `README.md`, `requirements.txt`, `requirements-dev.txt`, `docs/testing/TEST_FORMAT.md`, `docs/testing/FOLDER_STRUCTURE.md`, and `docs/testing/TEST_COVERAGE.md`.
- Confirmed the correct feature location for `$minN` array expression tests.
- Created and pushed branch `fix-issue-201`.
- Installed the project's pre-commit hooks.
- Confirmed upstream `main` already has a basic `$minN` smoke test.
- Confirmed issue #201 remains open and is a second-pass coverage issue.
- Identified MongoDB/DocumentDB availability as the remaining local execution dependency.

### Phase III Progress

- Added a new null-filtering test file without modifying the existing smoke test.
- Added a new `n` validation test file that uses `assertFailureCode`.
- Added `N_ARRAY_ELEMENT_NON_POSITIVE_N_ERROR = 5787908` to `documentdb_tests/framework/error_codes.py`.
- Confirmed MongoDB was installed locally but the Windows service was initially stopped.
- Started the local MongoDB service and verified focused test execution against `localhost:27017`.
- Verified the new files follow the repository's test-format and folder-structure rules.
- Created the local signed-off commit `bd0b3e5 Add minN array element compatibility tests`.
- Pushed the code commit to the personal fork branch and submitted the pull request.

### Phase IV Pre-Submission Check

Current check from June 27, 2026:

- `functional-tests` is on branch `fix-issue-201` and is ahead of `origin/fix-issue-201` by one local commit.
- `git diff origin/main...HEAD` is limited to the two new `$minN-array-element` test files and `documentdb_tests/framework/error_codes.py`.
- `git diff --check origin/main...HEAD` passed with no whitespace errors.
- No debug statements such as `print`, `pdb`, `TODO`, `FIXME`, or `breakpoint` were found in the changed files.
- MongoDB was running locally and reachable on `localhost:27017`.
- The focused `$minN-array-element` tests passed against local MongoDB.

### Code Changes

Implemented locally:

- `documentdb_tests/compatibility/tests/core/operator/expressions/array/minN-array-element/test_minN-array-element_null_filtering.py`
  - Adds `test_minN_array_element_filters_null_values_when_n_exceeds_available_values`.
  - Uses `assertSuccess`.
  - Verifies `$minN` filters `null` values and returns all available non-null values when `n` exceeds the available values.
- `documentdb_tests/compatibility/tests/core/operator/expressions/array/minN-array-element/test_minN-array-element_n_validation.py`
  - Adds `test_minN_array_element_rejects_zero_n`.
  - Uses `assertFailureCode`.
  - Verifies `$minN` rejects `n: 0`.
- `documentdb_tests/framework/error_codes.py`
  - Adds `N_ARRAY_ELEMENT_NON_POSITIVE_N_ERROR = 5787908`.

The existing smoke test file was intentionally left unchanged.

### Local Validation Results

Passed in the latest Phase IV check:

```bash
& "D:\Andranik\my coding tests\codepath\.venv\Scripts\python.exe" -m pytest "documentdb_tests\compatibility\tests\core\operator\expressions\array\minN-array-element\test_minN-array-element_null_filtering.py" "documentdb_tests\compatibility\tests\core\operator\expressions\array\minN-array-element\test_minN-array-element_n_validation.py" --collect-only
```

Result: 2 tests collected successfully.

Focused MongoDB execution passed in the latest Phase IV check:

```bash
& "D:\Andranik\my coding tests\codepath\.venv\Scripts\python.exe" -m pytest "documentdb_tests\compatibility\tests\core\operator\expressions\array\minN-array-element\test_minN-array-element_null_filtering.py" "documentdb_tests\compatibility\tests\core\operator\expressions\array\minN-array-element\test_minN-array-element_n_validation.py" --connection-string "mongodb://localhost:27017/?serverSelectionTimeoutMS=2000" --engine-name mongodb
```

Result: 2 passed.

Focused quality checks passed through `.venv` in the latest Phase IV check:

```bash
& "D:\Andranik\my coding tests\codepath\.venv\Scripts\python.exe" -m black --check "documentdb_tests\compatibility\tests\core\operator\expressions\array\minN-array-element\test_minN-array-element_null_filtering.py" "documentdb_tests\compatibility\tests\core\operator\expressions\array\minN-array-element\test_minN-array-element_n_validation.py" "documentdb_tests\framework\error_codes.py"
& "D:\Andranik\my coding tests\codepath\.venv\Scripts\python.exe" -m isort --check-only "documentdb_tests\compatibility\tests\core\operator\expressions\array\minN-array-element\test_minN-array-element_null_filtering.py" "documentdb_tests\compatibility\tests\core\operator\expressions\array\minN-array-element\test_minN-array-element_n_validation.py" "documentdb_tests\framework\error_codes.py"
& "D:\Andranik\my coding tests\codepath\.venv\Scripts\python.exe" -m flake8 "documentdb_tests\compatibility\tests\core\operator\expressions\array\minN-array-element\test_minN-array-element_null_filtering.py" "documentdb_tests\compatibility\tests\core\operator\expressions\array\minN-array-element\test_minN-array-element_n_validation.py" "documentdb_tests\framework\error_codes.py"
```

Full type check passed through `.venv` in the latest Phase IV check:

```bash
& "D:\Andranik\my coding tests\codepath\.venv\Scripts\python.exe" -m mypy documentdb_tests\ --no-site-packages
```

Known local environment issue:

- If MongoDB is stopped in a future shell, the focused functional tests require starting the local `MongoDB` Windows service before rerunning.
- `pre-commit run --all-files` currently invokes system `C:\Python314\python.EXE`, which does not have `black`, `isort`, `flake8`, or `mypy` installed. The equivalent checks passed when run through the project `.venv`.

---

## Pull Request

**PR Link:** Submitted on GitHub; direct URL not recorded in this local README yet.

**PR Description:**

Submitted PR description:

```markdown
## What does this PR do?

Adds second-pass compatibility coverage for the `$minN` array expression operator.

This PR adds tests for:
- filtering `null` values from `$minN` array input
- returning all available non-null values when `n` is larger than the number of usable values
- rejecting invalid `n: 0` input with the expected MongoDB error code

## Why was this PR needed?

Issue #201 asks for additional `$minN` array expression compatibility coverage beyond the existing smoke test. The existing test confirms the basic happy path, but it does not cover edge behavior such as null handling or validation of invalid `n` values.

These tests help document expected MongoDB-compatible behavior and protect against future regressions.

## How did you test it?

Focused tests passed against local MongoDB:

```bash
python -m pytest documentdb_tests/compatibility/tests/core/operator/expressions/array/minN-array-element/test_minN-array-element_null_filtering.py documentdb_tests/compatibility/tests/core/operator/expressions/array/minN-array-element/test_minN-array-element_n_validation.py --connection-string "mongodb://localhost:27017/?serverSelectionTimeoutMS=2000" --engine-name mongodb
```

**Maintainer Feedback:** None received yet.

**Status:** PR submitted; awaiting maintainer review.

---

## Learnings & Reflections

### Technical Skills Gained

- Learned how the DocumentDB functional test framework organizes tests by feature tree.
- Learned that functional tests use `execute_command` and framework assertion helpers.
- Learned that running these tests requires a live MongoDB or DocumentDB instance.
- Learned when to use `assertSuccess` for successful aggregation output and `assertFailureCode` for error-code validation.
- Learned that some aggregation expression validation is runtime-based and requires at least one input document to trigger the error.

### Challenges Overcome

- Git initially reported a safe-directory ownership warning in the sandbox, so commands were run with a per-command `safe.directory` option.
- The test environment had dependencies installed in `.venv`, but the system Python did not.
- Pre-commit initially failed because Git did not trust the sandbox-owned checkout and tried to write its cache outside the workspace; rerunning it with temporary safe-directory and `PRE_COMMIT_HOME` environment variables fixed the setup.
- Functional execution initially failed because the local MongoDB service was stopped.
- The first proposed null-filtering filename failed collection validation because files in feature folders must include the exact parent folder name. The file was renamed to include `minN-array-element`.
- `pre-commit run --all-files` still fails in the local shell because the system Python used by the hooks is missing the required tools, even though the `.venv` checks pass.

### What I'd Do Differently Next Time

Next time, I would spend more time researching the issue before starting implementation so I can better understand the exact work needed, the existing test coverage, and the project’s expectations. I would also compare a few issue types before choosing one, because a different kind of issue may have been easier to scope and validate.

## Resources Used

- Project issue: https://github.com/documentdb/functional-tests/issues/201
- Project contribution guide: https://github.com/documentdb/functional-tests/blob/main/CONTRIBUTING.md
- MongoDB `$minN` array expression docs: https://www.mongodb.com/docs/manual/reference/operator/aggregation/minn-array-element/
- Working branch: https://github.com/andavag/functional-tests/tree/fix-issue-201
