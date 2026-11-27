# 11. Risks and Technical Debts

## 11.1 Risks

These risks arise from decisions made to fill gaps the kata leaves unspecified.
Each risk identifies what was assumed, what could go wrong if the assumption is
wrong, and how to mitigate it.

| ID   | Risk | Assumption (ADR) | Impact if wrong | Mitigation |
|------|------|-----------------|-----------------|------------|
| R-1 | **Edge wrapping mismatches the intended behavior.** The kata does not specify grid-edge behavior. We chose toroidal wrapping. | ADR-002 | Runs that reach the grid boundary produce unexpected positions. All affected tests fail. | The assumption is stated explicitly in §1.2 (A-1) and ADR-002. If the kata owner specifies a different rule, supersede ADR-002 and update `Grid.wrap()` and the edge-wrapping tests only; no other module changes. |
| R-2 | **Skip-and-continue mismatches the intended obstacle behavior.** "Does nothing" could mean stop the entire run, not just skip one command. | ADR-003 | Runs with obstacles produce extra commands executed after the block, giving a different final position. | The interpretation is recorded in ADR-003 and §1.2 (R-4) and §2.3 (DC-3). If stop-on-first-obstacle is required, the change is confined to `Rover.execute()` and the BLOCKED output format. |
| R-3 | **BLOCKED output is not expected by the caller.** The kata does not mention reporting obstacles. A downstream tool parsing stdout may not handle `BLOCKED` lines. | ADR-003 | Automated consumers of the output break. | BLOCKED lines are prefixed with the keyword `BLOCKED`, so a consumer that only reads the last line gets the final position correctly. The risk is low for a CLI tool with no known automated consumers. |
| R-4 | **Input format is not accepted by the kata test harness.** The 3-or-4-group plain-text format is our invention; an external evaluator may expect a different format. | ADR-003 | The program cannot be driven by an external test suite without a format adapter. | The format is fully specified in §4.4. Adapting it requires changes only to `mission_control.py` (the parser); domain logic is unaffected. |

## 11.2 Technical Debts

| ID   | Debt | Reason incurred | Cost to pay |
|------|------|----------------|-------------|
| TD-1 | **No `PYTHONPATH` or package setup.** `src/` is not a Python package (no `__init__.py`, no `pyproject.toml` install). pytest and the simulator must be invoked with the right working directory or `PYTHONPATH=src` set in the Dockerfile. | Kept simple for the kata; full packaging is out of scope. | Add a `pyproject.toml` with `[tool.pytest.ini_options] pythonpath = ["src"]` or install the package in editable mode. One file change, no domain impact. |
| TD-2 | **No Docker smoke test.** QS-9 and QS-10 (Docker build and run) are not covered by the pytest suite; they require manual verification or a CI step. | pytest cannot easily test Docker invocations. | Add a shell-based smoke test in the CI pipeline (Module 4) that builds the image and asserts on the output of a known input file. |
| TD-3 | **Comment support in the input file adds parser complexity.** Stripping `#` lines before group-splitting is simple but untested edge cases exist (e.g. a file consisting only of comments). | Comments improve readability of example input files. | Cover comment-only and mixed-comment files in the Mission Control integration tests (Module 5). |
