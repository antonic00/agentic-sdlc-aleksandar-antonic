# 8. Cross-cutting Concepts

## 8.1 Input Validation

Validation is the sole responsibility of Mission Control. Domain objects
(`Grid`, `Rover`) receive only clean, validated data and perform no defensive
checks themselves. This keeps domain logic simple and testable in isolation.

Validation happens in two passes inside Mission Control:

**Structural validation** — before constructing any domain object:

| Check | Error condition |
|-------|----------------|
| File exists and is readable | File not found or permission denied |
| Group count after stripping comments and blank lines | Not 3 or 4 groups |
| Grid section | Not exactly two positive integers |
| Obstacle section (4-group form) | Any line is not exactly two integers, or any coordinate is outside `[0, width)` × `[0, height)` |
| Rover starting state | Not exactly two integers and one direction character from `{N, S, E, W}`, or position outside the grid, or position on an obstacle cell |
| Command string | Contains any character outside `{F, R, L}` |

**Semantic validation** — after constructing domain objects: none required.
Structural validation is sufficient to guarantee domain objects receive valid
input.

On any validation failure Mission Control prints one line to stderr:

```
Error: <description>
```

and exits with code 1 (see §8.2).

## 8.2 Error Handling and Exit Codes

The program uses two exit codes:

| Code | Meaning |
|------|---------|
| `0` | Simulation completed successfully. Final position and any `BLOCKED` lines printed to stdout. |
| `1` | Input was invalid. One `Error:` line printed to stderr. No output on stdout. |

There are no other exit codes. Unhandled exceptions (bugs) will produce a
Python traceback on stderr and a non-zero exit code from the interpreter; they
are not given a dedicated code because they indicate a defect, not a user
error.

Error messages are addressed to the operator: they name the specific violation
and, where applicable, the offending value or line. Example:

```
Error: obstacle coordinate (12, 4) is outside the grid (10x10)
Error: unknown command character 'X' in command string
Error: rover starting position (0, 0) is an obstacle cell
```

## 8.3 Domain Model Conventions

These conventions apply across all three bounded contexts and must be followed
consistently in implementation and tests:

| Convention | Detail |
|------------|--------|
| Coordinate origin | `(0, 0)` is the bottom-left cell. `x` increases eastward; `y` increases northward. |
| Direction values | The four cardinal strings `"N"`, `"S"`, `"E"`, `"W"`. No integers or enums at the boundary between contexts. |
| Command values | The three characters `"F"`, `"R"`, `"L"`. Passed as a plain string; Navigation iterates over it. |
| Wrapping | Applied by `Grid.wrap(x, y)` using `x % width`, `y % height`. Navigation calls `wrap` before `is_free`; raw out-of-range coordinates never reach `is_free`. |
| Immutability of Terrain | A `Grid` instance is constructed once and never mutated. Obstacle positions are fixed for a run. |
| Rover state | `Rover` is mutable for the duration of `execute()`; its state after the call is the final state. |

## 8.4 Testing Approach

Testability is a first-class quality goal (see §1.3). The approach follows
directly from the bounded-context structure:

- **Unit tests for Terrain** construct a `Grid` directly and assert on
  `is_free()` and `wrap()` for normal cells, obstacle cells, and edge cells.
  No file I/O required.
- **Unit tests for Navigation** construct a `Rover` with a test `Grid` and
  call `execute()` with a command string. Assertions cover turning, forward
  movement, obstacle blocking, and edge wrapping. No file I/O required.
- **Integration tests for Mission Control** write a temporary input file (or
  use `pytest`'s `tmp_path` fixture), call the parser, and assert on the
  parsed domain objects. A small number of end-to-end tests invoke
  `mission_control.main()` and capture stdout/stderr.

No test requires a Docker container or a subprocess call. The full suite runs
with `pytest tests/` on any machine with Python 3.12 and pytest installed, and
inside the Docker image via the default `CMD`.

Tests are written one at a time following TDD (Module 5). Test names are
sentences that describe behavior and include the Story ID and Scenario ID from
the user stories produced in Module 3, for example:

```
test_nav_be_001_1_s1_given_rover_facing_north_when_moving_forward_then_y_increases
```

Every test body uses `# GIVEN`, `# WHEN`, and `# THEN` comments to mark the
three phases.

## 8.5 Logging

The simulator has no logging. It is a short-lived CLI tool: each run starts,
does one unit of work, and exits. The output contract (stdout for results,
stderr for errors) is sufficient for both human operators and CI pipelines to
determine what happened. Adding a logging framework would be unnecessary
complexity (see quality goal: Simplicity, §1.3).

If a future requirement calls for an audit trail or debug mode, that decision
will be recorded as a new ADR before implementation.
