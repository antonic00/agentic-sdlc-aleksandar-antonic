# 9. Architecture Decisions

## ADR-001: Use Python 3.12 with the Standard Library, pytest, In-Memory State, and Docker

**Status:** Accepted

**Context:**
The Mars Rover Simulator takes a starting position and direction, a map with
obstacles, and a list of commands (move forward, turn left, turn right). It
outputs the rover's final position and direction. When the rover meets an
obstacle it does nothing. We need to choose the language, framework, test
tooling, data storage, and runtime environment. The course tech stack is
Python 3.12 with pytest. Later modules build and test the project in Docker
and implement it with strict TDD.

**Decision:**
- Language: Python 3.12.
- Framework: none. The simulator is a command-line application that uses only
  the Python standard library.
- Testing: pytest.
- Data storage: none. The map, rover state, and commands are held in memory
  for a single run. Nothing is persisted between runs.
- Runtime: a Docker image based on Python 3.12, built with `docker build` and
  run with `docker run`.

**Rationale:**
- Python 3.12 and pytest are the course tech stack, so the later agents
  (CI/CD and TDD/BDD) work with them directly.
- The problem is pure logic on a small grid. A web or application framework
  would add complexity without adding value.
- pytest gives fast, readable tests, which suits one-test-at-a-time TDD and
  GIVEN-WHEN-THEN test structure.
- The kata has no requirement to store data between runs, so a database would
  be unnecessary infrastructure.
- Docker gives the same environment on every machine and in CI, which avoids
  "works on my machine" problems.

**Consequences:**
- No third-party runtime dependencies. Only pytest is needed for testing.
- Input and output are plain text on the command line. Their exact format
  must be defined, because the kata does not specify it.
- No state survives between runs. Saving or replaying missions would need a
  new decision.
- Docker must be installed to build and run the project the same way as in CI.

---

## ADR-002: Toroidal Edge Wrapping

**Status:** Accepted

**Context:**
The kata does not specify what happens when the rover moves off the edge of
the grid. Three options exist: treat the boundary as a hard wall (block the
move), raise an error, or wrap to the opposite edge (toroidal topology). The
kata requires the program to produce a final position for any valid input, so
silently stopping at the wall or raising an error on an edge move would
surprise operators and complicate test cases.

**Decision:**
The grid has toroidal topology. Moving off one edge places the rover on the
opposite edge. Wrapping is applied using modular arithmetic:
`new_x = new_x % width`, `new_y = new_y % height`. The `Grid` class owns
this calculation via `Grid.wrap(x, y)`. Navigation calls `wrap` to compute
the target cell before calling `is_free`.

**Rationale:**
- Toroidal wrapping is a common convention in Mars Rover kata variants and
  produces intuitive behavior for operators.
- It eliminates a class of boundary-condition errors without adding any
  conditional logic beyond the modulo operation.
- A hard wall could also be implemented inside Terrain (`is_free` returns
  `False` for coordinates outside the grid), but every edge move would then
  be a blocked move and produce a `BLOCKED` line in the output. Wrapping keeps
  edge moves as normal moves with no special output.

**Consequences:**
- Every forward move produces a valid cell coordinate regardless of starting
  position or heading. No move can raise a bounds error.
- Tests must include edge-wrapping cases (e.g. rover at the north edge moves
  north and arrives at y = 0).
- Operators may be surprised if they expect a wall. The assumption is stated
  in §1.2 (A-1) and in the input format documentation (§4.4).

---

## ADR-003: Plain-Text Input File, stdout Output, BLOCKED Reporting, and Skip-and-Continue Obstacle Behavior

**Status:** Accepted

**Context:**
The kata does not specify the input/output format, whether obstacles are
reported, or the exact behavior when the rover meets an obstacle ("it does
nothing" leaves open whether the whole run stops or only the blocked command
is skipped). These four concerns are decided together because they define the
complete I/O contract, and the obstacle behavior directly affects what the
output must contain.

**Decision:**

*Input:* A plain-text UTF-8 file passed as a CLI argument. The file contains
3 or 4 blank-line-separated groups (comment lines starting with `#` are
ignored): grid dimensions; optionally, obstacle coordinates (one `x y` pair
per line); rover starting state (`x y direction`); command string (`F`, `R`,
`L` characters). The parser determines whether obstacles are present by group
count: 4 groups means obstacles are included, 3 means they are absent.

*Output:* Plain text on stdout. The final line is `x y direction`. Each
command that was blocked by an obstacle produces a preceding
`BLOCKED x y direction` line showing the cell the rover could not enter and
the heading at the time of the block.

*Obstacle behavior:* "Does nothing" is interpreted as: skip the blocked
command and continue executing the remaining commands. The rover does not
stop. One `BLOCKED` line is emitted per blocked command.

*Errors:* Any invalid input causes the program to exit with code 1 and print
one `Error: <description>` line to stderr.

**Rationale:**
- A file argument is simpler to test and to mount in Docker than stdin piping,
  and it separates the input data from the command invocation.
- The 3-vs-4-group rule makes the obstacles section unambiguously optional
  without requiring a sentinel value or a header keyword.
- Reporting `BLOCKED` lines gives the operator actionable feedback at no cost
  to the domain model: Navigation already needs to detect the block to skip
  the command.
- Skip-and-continue is the most useful interpretation for a simulator: the
  operator sees the full effect of the command sequence, not just the first
  obstacle hit.

**Consequences:**
- The input format is fully specified and stable. The TDD/BDD agent (Module 5)
  and the CI/CD agent (Module 4) can rely on it without further negotiation.
- A run with no obstacles and no blocked commands produces exactly one line of
  output, making automated assertion simple.
- If the kata is later interpreted as "stop on first obstacle", this ADR must
  be superseded and the output format and test suite updated accordingly.

---

## ADR-004: Three Bounded Contexts as Python Modules with an Explicit Dependency Rule

**Status:** Accepted

**Context:**
The problem has three distinct concerns: navigating the rover (position,
heading, movement logic), describing the terrain (grid size, obstacles, edge
wrapping), and orchestrating the run (I/O, parsing, formatting). These could
be implemented in a single module or split in various ways. The choice affects
testability, the clarity of ownership, and how well the code maps to the
problem domain.

**Decision:**
The simulator is structured as three bounded contexts, each implemented as one
Python module:

- `terrain.py` — owns `Grid`: dimensions, obstacles, `wrap()`, `is_free()`.
- `navigation.py` — owns `Rover` and `Direction`: position, heading, command
  execution. Depends on `terrain`.
- `mission_control.py` — owns the entry point, parser, and formatter. Depends
  on both `navigation` and `terrain`.

Dependency rule: `mission_control` → `navigation` → `terrain`. `terrain`
depends on nothing. No module imports in the reverse direction.

**Rationale:**
- Each module has a single, named responsibility matching a bounded context
  from the domain. Future readers can locate any behavior by context.
- The dependency rule makes `terrain` and `navigation` independently
  instantiable in tests without any I/O setup.
- Obstacle checking belongs to Terrain (`is_free`), not to Navigation. This
  keeps the question "is this cell occupied?" separate from "how does the
  rover move?", which are genuinely different concerns.
- Two modules instead of three (merging Terrain into Navigation) would couple
  grid knowledge to movement logic and make both harder to test in isolation.

**Consequences:**
- `terrain.py` and `navigation.py` contain no `import sys`, no file reads,
  and no print calls. This is verifiable by inspection and enforced by the
  test strategy in §8.4.
- Module 5 (TDD/BDD) can write unit tests for `terrain` and `navigation`
  without any test fixtures that touch the file system.
- Adding a second rover or a second grid type in the future would require
  extending `terrain` and `navigation` only, leaving `mission_control`
  unchanged.
