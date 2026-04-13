# agent-trace

A notebook-first demo of an agentic loop using **Amazon Bedrock** (Claude Sonnet) with **OpenTelemetry** trajectory tracing. Run the agent fully offline with a built-in mock client, or point it at live Bedrock with your AWS credentials.

## Overview

The demo runs a security and network diagnostic agent with two tools:

- **`lookup_cve`** — looks up a CVE identifier and returns severity, CVSS score, affected versions, and remediation guidance from a local database.
- **`check_open_ports`** — probes TCP ports on loopback addresses only (`127.x.x.x` / `localhost`).

Every model call and tool invocation emits an **OpenTelemetry span**, producing a trace hierarchy like:

```
agent.run
  └─ bedrock.converse          (turn 1 — model requests lookup_cve)
       └─ tool.lookup_cve
  └─ bedrock.converse          (turn 2 — model requests check_open_ports)
       └─ tool.check_open_ports
  └─ bedrock.converse          (turn 3 — final answer)
```

Spans carry attributes for model ID, token usage, latency, tool inputs/outputs, and finish reason.

## Requirements

- Python 3.13+
- [`uv`](https://docs.astral.sh/uv/) for dependency and environment management

## Quickstart

```bash
# Install dependencies
uv sync

# Launch the notebook
uv run jupyter lab notebooks/agent-trace.ipynb
```

Then run the cells in order (Cell 2 → Cell 9). The default mode is **fully offline** — no AWS credentials needed.

## Running Modes

| Mode | Steps |
|------|-------|
| **Fully offline (mocked)** | Run all cells as-is. `USE_MOCK = True` is the default in Cell 2. |
| **Live Amazon Bedrock** | Set `USE_MOCK = False` in Cell 2 and export AWS credentials (see below). |
| **OTLP export** | Set `OTEL_EXPORTER_OTLP_ENDPOINT` before running Cell 3 (see below). |

### Live Bedrock

```bash
export AWS_REGION=us-east-1
export AWS_ACCESS_KEY_ID=<your-key-id>
export AWS_SECRET_ACCESS_KEY=<your-secret-key>
```

Then set `USE_MOCK = False` in Cell 2 and re-run from Cell 2.

### OTLP Export

Set the endpoint before running Cell 3 (or restart the kernel and set it in Cell 2):

```bash
export OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4318
```

To run a local OpenTelemetry Collector with Podman:

```bash
podman run --rm -p 4317:4317 -p 4318:4318 \
  otel/opentelemetry-collector-contrib:latest
```

## Running Tests

All tests run fully offline — no AWS calls required:

```bash
uv run pytest
```

Tests are also embedded in Cell 10 of the notebook and can be run directly in-kernel.

## Project Structure

```
notebooks/
  agent-trace.ipynb   # Main demo notebook
main.py               # Minimal entrypoint (reserved)
pyproject.toml        # Dependencies (boto3, opentelemetry-*, pytest)
README.md
```

## Scope

- **In scope:** Educational demo of OpenTelemetry trajectory tracing with an agentic loop.
- **Out of scope:** Production security scanning, live network probing beyond loopback, and distributed multi-agent orchestration.
