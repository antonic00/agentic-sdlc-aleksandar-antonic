# 4. Solution Strategy

## 4.1 Approach

The simulator is structured around three bounded contexts that map directly to
the problem's three concerns. Each context owns its data and exposes a narrow
interface; no context reaches into another's internals.

| Bounded Context | Responsibility | Key abstraction |
|-----------------|---------------|-----------------|
| **Mission Control** | Read the input file, parse it into domain objects, invoke the simulation, format and print the result. CLI entry point. | `InputParser`, `OutputFormatter`, `main()` |
| **Terrain** | Hold the grid dimensions and obstacle positions. Answer whether a given cell is free. Apply edge wrapping. | `Grid` |
| **Navigation** | Hold the rover's current position and heading. Execute F, R, L commands by calling Terrain to check the target cell before moving. | `Rover`, `Direction`, `Command` |

Dependency rule: **Mission Control → Navigation**, **Mission Control → Terrain**,
**Navigation → Terrain**. Terrain depends on nothing. Mission Control constructs
both a `Grid` (Terrain) and a `Rover` (Navigation), passes the `Grid` into the
`Rover`, then calls the `Rover` to execute commands. Navigation never imports
Mission Control; Terrain never imports Navigation or Mission Control.

## 4.2 Key Decisions

| Decision | Choice | Where recorded |
|----------|--------|----------------|
| Language and runtime | Python 3.12 stdlib, pytest, Docker | ADR-001 |
| Grid edge behaviour | Toroidal wrapping | ADR-002 |
| I/O format | Plain-text file in; stdout out | ADR-003 |
| Obstacle reporting | Print blocked position when a command is skipped | ADR-003 |
| Domain model | Three bounded contexts; pure functions for domain logic | This chapter |

## 4.3 Achieving the Quality Goals

| Quality Goal | How it is achieved |
|-------------|-------------------|
| **Correctness** | Domain logic (Rover, Grid) contains no I/O. It can be exercised directly by pytest with no subprocess or file setup. |
| **Testability** | Bounded context boundaries are Python module boundaries. Tests import `navigation` and `terrain` directly. |
| **Simplicity** | No framework, no database, no async. The entire runtime is `python main.py <file>` inside Docker. |
| **Portability** | A single-stage `Dockerfile` based on `python:3.12-slim` reproducibly builds and runs the program anywhere. |

## 4.4 Input File Format

The input file is a plain-text UTF-8 file. Lines starting with `#` are comments
and are ignored. Non-comment content is split into groups by blank lines. The
parser determines the structure by group count:

- **4 groups**: grid dimensions, obstacles, rover starting state, commands.
- **3 groups**: grid dimensions, rover starting state, commands (no obstacles).

The obstacles section is the only optional section. Its absence means an
empty obstacle set; the rover moves freely.

With obstacles:

```
# Section 1 — grid dimensions (width height)
10 10

# Section 2 — obstacles (one "x y" pair per line)
2 4
3 7

# Section 3 — rover starting state (x y direction)
0 0 N

# Section 4 — commands (single string of F, R, L characters)
FFRFFLF
```

Without obstacles:

```
# Section 1 — grid dimensions (width height)
10 10

# Section 2 — rover starting state (x y direction)
0 0 N

# Section 3 — commands (single string of F, R, L characters)
FFRFFLF
```

Rules:
- `width` and `height` are positive integers.
- Obstacle coordinates must be within the grid (0 ≤ x < width, 0 ≤ y < height).
- Starting position must be within the grid and must not be an obstacle cell.
- The command string contains only the characters `F`, `R`, `L` (uppercase).
- Any deviation from these rules causes the program to exit with code 1 and
  print a descriptive error to stderr.

## 4.5 Output Format

On success the program prints to stdout:

```
<x> <y> <direction>
```

If one or more commands were blocked by an obstacle, each blocked position is
printed before the final line:

```
BLOCKED <x> <y> <direction>
<x> <y> <direction>
```

Where each `BLOCKED` line shows the position the rover would have moved to and
the direction it was already facing at the time of the blocked command.

Example — rover starts at (0, 0) facing N on a 10×10 grid with an obstacle at
(0, 2), and receives commands `FFF`:

```
BLOCKED 0 2 N
BLOCKED 0 2 N
0 1 N
```

The first F moves the rover to (0, 1). The second F would move it to (0, 2),
which is an obstacle, so the command is skipped and a `BLOCKED 0 2 N` line is
printed. The rover stays at (0, 1). The third F would again move it to (0, 2),
is blocked again, and a second `BLOCKED 0 2 N` line is printed. The final
position is (0, 1) facing N.

On error the program exits with code 1 and prints one line to stderr:

```
Error: <description>
```
