# 2. Architecture Constraints

## 2.1 Technical Constraints

| ID   | Constraint | Source |
|------|------------|--------|
| TC-1 | Implementation language is Python 3.12. No third-party runtime packages. | Course tech stack (ADR-001) |
| TC-2 | Test framework is pytest. No other test runners or assertion libraries. | Course tech stack (ADR-001) |
| TC-3 | The application runs as a command-line program. No web server, GUI, or network interface. | ADR-001; kata scope |
| TC-4 | All state is held in memory for one run. Nothing is written to disk or a database during execution. | ADR-001 |
| TC-5 | The build and runtime environment is Docker. The project must produce a working image with `docker build` and run correctly with `docker run`. | Course runtime requirement (ADR-001) |
| TC-6 | Input is a plain-text file passed as a CLI argument. Output is plain text on stdout. | Assumption A-4 (ADR-003) |

## 2.2 Organisational Constraints

| ID   | Constraint | Source |
|------|------------|--------|
| OC-1 | The repository follows the course Git workflow. Every change is delivered via a pull request; the instructor (`momokrunic`) must be added as a reviewer before merging. | PR review steering rule |
| OC-2 | Architecture is documented using the arc42 template and C4 model, one chapter at a time, with approval before proceeding. | Module 2 assignment |
| OC-3 | The codebase must be ready to receive CI/CD configuration (Module 4) and TDD/BDD test suites (Module 5) without structural changes. | Course module dependencies |

## 2.3 Domain Constraints

| ID   | Constraint | Source |
|------|------------|--------|
| DC-1 | Valid directions are exactly N, S, E, W. | Kata specification |
| DC-2 | The kata defines three commands using arrow symbols (forward, turn right 90°, turn left 90°). F, R, L are our text encoding of those commands in the input file. | Kata specification + Assumption A-4 (input format) |
| DC-3 | The kata states "when the rover encounters an obstacle, it does nothing." Skipping only the blocked command and continuing with the next is our interpretation of that rule. | Interpretation of kata specification; see R-4 |
| DC-4 | The three bounded contexts — **Navigation**, **Terrain**, and **Mission Control** — keep separate models and vocabularies. Navigation asks Terrain whether a target cell is free; it does not inspect obstacle data directly. | Architectural principle; see Chapter 4 |

## 2.4 Conventions

| ID   | Constraint |
|------|------------|
| CV-1 | Coordinate origin (0, 0) is the bottom-left corner of the grid. x increases eastward; y increases northward. |
| CV-2 | Grid edges wrap (toroidal topology): moving north off y = grid_height−1 arrives at y = 0, and so on for all edges (ADR-002). |
| CV-3 | Architecture diagrams are PlantUML `.puml` files in `architecture/diagrams/`. Markdown references the generated `.svg`. The pre-commit hook renders SVGs on commit. |
