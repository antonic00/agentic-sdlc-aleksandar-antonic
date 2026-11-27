# 12. Glossary

Terms are grouped by bounded context where they belong to one. General terms
appear in a separate section. One term, one meaning — the same word is never
used for two different concepts across the codebase or documentation.

---

## Terrain

| Term | Definition |
|------|------------|
| **Grid** | The rectangular playing field on which the rover moves. Defined by a width and a height, both positive integers. The `Grid` class owns dimensions, obstacle positions, wrapping, and cell-availability queries. |
| **Width** | The number of cells in the horizontal (x) dimension of the grid. Valid x coordinates are `0` to `width − 1`. |
| **Height** | The number of cells in the vertical (y) dimension of the grid. Valid y coordinates are `0` to `height − 1`. |
| **Cell** | A single position on the grid, identified by an `(x, y)` coordinate pair. |
| **Obstacle** | A cell that the rover cannot enter. Obstacles are fixed for the duration of a run. Stored as a set of `(x, y)` pairs inside `Grid`. |
| **Free cell** | A cell that contains no obstacle. `Grid.is_free(x, y)` returns `True` for a free cell. |
| **Toroidal wrapping** | The edge behavior in which moving off one side of the grid places the rover on the opposite side. Implemented as `x % width`, `y % height` in `Grid.wrap(x, y)`. See ADR-002. |
| **Origin** | The cell `(0, 0)`, located at the bottom-left corner of the grid. |

---

## Navigation

| Term | Definition |
|------|------------|
| **Rover** | The agent being simulated. Represented by the `Rover` class, which owns the current position and heading and executes commands. |
| **Position** | The rover's current location on the grid, expressed as an `(x, y)` integer pair. |
| **Heading** (also **Direction**) | The cardinal direction the rover faces: one of `"N"`, `"S"`, `"E"`, `"W"`. |
| **Command** | A single instruction given to the rover: `"F"` (move forward one cell), `"R"` (turn right 90°), or `"L"` (turn left 90°). The kata defines these with arrow symbols; F, R, L are the text encoding used in the input file. |
| **Command string** | The sequence of commands for a run, represented as a string of `F`, `R`, and `L` characters (e.g. `"FFRFF"`). |
| **Forward move** | A command `F` that advances the rover one cell in its current heading direction, after applying toroidal wrapping. |
| **Turn right** | A command `R` that rotates the rover's heading 90° clockwise: N→E→S→W→N. |
| **Turn left** | A command `L` that rotates the rover's heading 90° counter-clockwise: N→W→S→E→N. |
| **Blocked move** | A forward command whose target cell is an obstacle. The command is skipped; the rover stays in its current position. See skip-and-continue. |
| **BLOCKED event** | The record produced when a forward move is blocked. Contains the target cell coordinates and the rover's heading at the time of the block. Formatted as `BLOCKED x y direction` in the output. |
| **Skip-and-continue** | The obstacle behavior: when a forward command is blocked, that command is skipped and execution continues with the next command. The rover does not stop. See ADR-003 and §2.3 (DC-3). |
| **Final state** | The rover's position and heading after all commands in the command string have been processed. |

---

## Mission Control

| Term | Definition |
|------|------------|
| **Input file** | The plain-text UTF-8 file passed as a CLI argument. Contains the grid dimensions, optionally obstacle coordinates, the rover's starting state, and the command string, separated into groups by blank lines. See §4.4. |
| **Group** | A block of non-blank, non-comment lines in the input file, separated from other groups by one or more blank lines. The parser identifies sections by group count (3 or 4). |
| **Starting state** | The rover's initial position `(x, y)` and heading, as given in the input file. |
| **Output** | The text written to stdout: zero or more `BLOCKED` lines followed by the final state line `x y direction`. |
| **Error message** | A line written to stderr in the form `Error: <description>` when the input file is invalid. The program exits with code 1. |
| **Exit code** | `0` for a successful run; `1` for invalid input or any error that prevents the simulation from completing. |

---

## General

| Term | Definition |
|------|------------|
| **Bounded context** | A part of the system with its own model and vocabulary, implemented as a single Python module. The three bounded contexts are Terrain (`terrain.py`), Navigation (`navigation.py`), and Mission Control (`mission_control.py`). |
| **Dependency rule** | The import constraint: `mission_control` may import `navigation` and `terrain`; `navigation` may import `terrain`; `terrain` imports neither. No reverse imports. See ADR-004. |
| **Kata** | A short, self-contained programming exercise used as a learning vehicle. This project implements the Mars Rover kata (codingdojo.org/kata/mars-rover). |
| **Run** | A single execution of the simulator: one input file processed, one output produced, process exits. No state persists between runs. |
| **Mission Operator** | The human actor who prepares the input file, invokes the simulator, and reads the output. The only external actor in the system. |
