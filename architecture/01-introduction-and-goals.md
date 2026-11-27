# 1. Introduction and Goals

## 1.1 Purpose

The Mars Rover Simulator is a command-line program that accepts a rover's
starting state, a map of obstacles, and a sequence of movement commands, then
reports where the rover ends up. It is built as the kata for Module 2 of the
Agentic SDLC course. Later modules add CI/CD, TDD/BDD tests, and deployment
automation to the same codebase.

## 1.2 Requirements Overview

| ID  | Requirement |
|-----|-------------|
| R-1 | Accept the rover's starting position (x, y) and facing direction (N, S, E, W). |
| R-2 | Accept a map that describes the locations of obstacles on a grid. |
| R-3 | Accept a list of commands: **F** (move forward one cell), **R** (turn right 90°), **L** (turn left 90°). |
| R-4 | Execute commands in order. When a move-forward command would land the rover on an obstacle, skip that command and continue with the next. |
| R-5 | Report the rover's final position and direction after all commands have been processed. |
| R-6 | Run as a command-line application. Input is provided via a file argument; output goes to stdout. |
| R-7 | Build and run inside Docker (`docker build` / `docker run`). |

### Assumptions (kata leaves these unspecified)

| ID  | Assumption | Rationale |
|-----|------------|-----------|
| A-1 | The grid wraps at its edges (toroidal topology). A rover moving off one edge appears on the opposite side. | Avoids a hard boundary error; common in Mars Rover kata variants. Recorded as ADR-002. |
| A-2 | Grid size is provided as part of the map input. The default for tests is 10 × 10. | A fixed size would make edge-wrapping tests fragile. |
| A-3 | When a command is blocked by an obstacle, the blocked position is included in the output alongside the final position. | Gives the operator actionable feedback without over-specifying normal output. |
| A-4 | Input is read from a plain-text file whose path is passed as a CLI argument. Output is written to stdout. | Simplest I/O contract that keeps domain logic free of stdin parsing. Recorded as ADR-003. |
| A-5 | Invalid commands or malformed input cause the program to exit with a non-zero code and a descriptive error message. | Prevents silent wrong output. |

## 1.3 Quality Goals

Top quality goals, in priority order:

| Priority | Quality Goal | Motivation |
|----------|-------------|------------|
| 1 | **Correctness** — the simulator produces the right final position and direction for any valid input. | The kata's sole success criterion. |
| 2 | **Testability** — every behavior is exercisable by automated pytest tests without I/O. | Required for the TDD/BDD module that follows. |
| 3 | **Simplicity** — no code exists that the kata does not require. | Keeps the codebase easy to reason about and extend in later modules. |
| 4 | **Portability** — the program runs identically on any machine with Docker installed. | Required by the course runtime constraint. |

## 1.4 Stakeholders

| Stakeholder | Expectation |
|-------------|-------------|
| **Mission Operator** (user) | Provide rover state and commands; receive a correct, readable result. |
| **Course Instructor** | Reviewable architecture following arc42 + C4; clean ADRs; working Docker build. |
| **TDD/BDD Agent** (Module 5) | Pure domain functions testable without I/O setup. |
| **CI/CD Agent** (Module 4) | A `Dockerfile` and a `pytest`-based test suite that run in a standard pipeline step. |
