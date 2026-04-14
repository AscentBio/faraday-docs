---
title: Run a single query
description: Fast local query execution for development and debugging.
sidebar:
  order: 2
---
## Run a query

```bash
faraday "Generate a SAR using my interesting_molecules.csv dataset"
```

You can also point Faraday at a config file:

```bash
faraday --config /path/to/faraday.yaml "Generate a SAR using my interesting_molecules.csv dataset"
```

## Common config

### Model

```yaml
llm:
  provider: openai
  model: gpt-5
  api_key_env: OPENAI_API_KEY
```

### Workspace

By default, Faraday uses the current working directory as its workspace. To point it somewhere else, use `--workspace-source-root` or set `app.workspace.source_root` in `faraday.yaml`.

Path note: `app.workspace.source_root` is a local path in your config. `sandbox.workspace.container_path` is the path seen inside Docker sandboxes.

```yaml
app:
  mode: host
  workspace:
    source_root: ./workspace_directory/

sandbox:
  backend: docker
  workspace:
    container_path: /workspace
```

For the example above, `interesting_molecules.csv` should be present in `./workspace_directory/`.

If you want each run to start from a clean copy of the source files, use:

```yaml
app:
  workspace:
    source_root: ./workspace_directory/
    init_mode: copy
    copy_root: ./.faraday_runtime/workspace-copies
    keep_copy: false
```

### Outputs

Unless configured otherwise, Faraday writes run outputs to `./run_outputs`.

That `outputs.root` value is typically a local path in your config, even when execution happens in Docker.

```yaml
outputs:
  root: ./run_outputs
```

## Where outputs go

Each run gets its own directory:

```bash
run_<timestamp>_<chat_id>_<query_id>
```

Inside that directory, Faraday writes:

```bash
agent_outputs/
run_artifacts/
```

So a typical run layout looks like:

```bash
./run_outputs/run_<timestamp>_<chat_id>_<query_id>/agent_outputs
./run_outputs/run_<timestamp>_<chat_id>_<query_id>/run_artifacts
```

`agent_outputs/` contains files the agent generated during the run. `run_artifacts/` contains structured run metadata such as events, results, and trajectory output.

See **[Results and artifacts](./results-and-artifacts/)** for the full output layout.

