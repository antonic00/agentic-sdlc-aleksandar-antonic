# 10. Quality Requirements

## 10.1 Quality Tree

```
Quality
├── Correctness (priority 1)
│   ├── Movement accuracy
│   ├── Obstacle handling
│   └── Edge wrapping
├── Testability (priority 2)
│   ├── Domain logic free of I/O
│   └── Each behavior exercisable in isolation
├── Simplicity (priority 3)
│   └── No code beyond what the kata requires
└── Portability (priority 4)
    └── Identical behavior inside and outside Docker
```

## 10.2 Quality Scenarios

Each scenario is phrased as a concrete, testable statement tied to a quality
goal and a chapter that specifies the behavior.

### Correctness

| ID   | Scenario |
|------|----------|
| QS-1 | Given a rover at (0,0,N) and commands `FFRFF` on an obstacle-free 10×10 grid, the simulator outputs `2 2 E`. |
| QS-2 | Given a rover at (0,0,N) and commands `FFF` with an obstacle at (0,2) on a 10×10 grid, the simulator outputs `BLOCKED 0 2 N`, `BLOCKED 0 2 N`, `0 1 N`. |
| QS-3 | Given a rover at (0,9,N) on a 10×10 grid with no obstacles and command `F`, the simulator outputs `0 0 N` (toroidal wrap, north edge → y=0). |
| QS-4 | Given a rover at (0,0,N) and commands `RRR`, the simulator outputs `0 0 W` (three right turns). |

### Testability

| ID   | Scenario |
|------|----------|
| QS-5 | A pytest test for `Rover.execute()` can be written with zero file-system setup: constructing `Grid` and `Rover` directly in the test body is sufficient. |
| QS-6 | A pytest test for `Grid.is_free()` can be written without importing `mission_control` or `navigation`. |

### Simplicity

| ID   | Scenario |
|------|----------|
| QS-7 | `terrain.py` and `navigation.py` contain no `import sys`, no `open()` calls, and no `print()` calls. Verifiable by static inspection. |
| QS-8 | `requirements.txt` lists only `pytest`. The application itself has no third-party runtime dependencies. |

### Portability

| ID   | Scenario |
|------|----------|
| QS-9 | `docker build -t rover .` completes without error on any machine with Docker and internet access. |
| QS-10 | `docker run --rm rover` (default CMD) runs the full pytest suite and exits with code 0 when all tests pass. |
