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
