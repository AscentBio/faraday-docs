---
title: Harbor
description: Run Faraday as a Harbor agent for benchmarking workflows.
---

Faraday works as a [Harbor](https://harborframework.com/docs/agents) installed agent. Your normal `faraday.yaml` controls everything -- no Harbor-specific config is needed beyond the launch command.

## Quick start

```bash
harbor run -d "<dataset@version>" \
  --agent-import-path faraday.integrations.harbor.agent:FaradayHarborAgent
```


```
faraday.integrations.harbor.agent:FaradayHarborAgent
```


That's it. Harbor handl
es installing Faraday from your repo and routing task instructions to the agent.

## Configuration

Place a `faraday.yaml` at your **repository root** (start from `faraday.example.yaml`). This is the same config file used for local and Docker runs. The most relevant settings:

```yaml
app:
  mode: host              # runs inside the Harbor task container
  workspace:
    source_root: /workspace

sandbox:
  backend: docker         # code sandbox (default for Harbor)
  workspace:
    container_path: /workspace
```

These are the defaults -- you only need a `faraday.yaml` if you want to change something.

## Output artifacts

After each run, artifacts are written to `/logs/artifacts`:

| File | What's in it |
|------|-------------|
| `result.json` | The agent's final answer and run summary |
| `events.jsonl` | Timestamped stream of every agent event |
| `metadata.json` | Run parameters (model, steps, config) |
| `trajectory.json` | Full trajectory in ATIF format for replay |

Override the output location with `--artifacts-dir`:

```bash
harbor run -d "<dataset@version>" \
  --agent-import-path faraday.integrations.harbor.agent:FaradayHarborAgent \
  --artifacts-dir /custom/output
```

