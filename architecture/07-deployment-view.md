# 7. Deployment View

## 7.1 Deployment Environment

There is one deployment environment: a developer machine or CI runner that has
Docker installed. There is no staging environment, no cloud target, and no
orchestration layer. The Docker image is the only deployable artefact.

## 7.2 Docker Image

The image is built from a single-stage `Dockerfile` based on `python:3.12-slim`.

| Layer | Content |
|-------|---------|
| Base image | `python:3.12-slim` |
| `requirements.txt` | Copied and installed first (only `pytest`; no runtime dependencies) |
| `src/` | Application source: `mission_control.py`, `navigation.py`, `terrain.py` |
| `tests/` | pytest test modules (executed by the default CMD) |
| Default `CMD` | `pytest tests/` — runs the full test suite |

The default command runs tests, not the simulator. This matches the Module 4
CI/CD pattern where `docker run` in the pipeline means "run the tests".

## 7.3 Running the Simulator

To run the simulator the operator overrides the default command and mounts the
input file into the container:

```bash
# Build the image
docker build -t rover .

# Run the test suite (default CMD)
docker run --rm rover

# Run the simulator with an input file
docker run --rm \
  -v "$(pwd)/mission.txt:/mission.txt:ro" \
  rover \
  python src/mission_control.py /mission.txt
```

The input file is bind-mounted read-only into the container. The result is
printed to stdout, which Docker forwards to the terminal.

## 7.4 Dockerfile Structure

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY src/ src/
COPY tests/ tests/

CMD ["pytest", "tests/"]
```

`requirements.txt` contains only `pytest`. There are no other runtime or
build-time dependencies.

## 7.5 Why No Deployment Diagram

A C4 deployment diagram shows nodes (servers, containers, devices) and how
artefacts are deployed onto them. With a single Docker image running on a
single machine and no network topology, the diagram would show one node
containing one container — adding no information beyond what this chapter
states in prose. It is omitted.
