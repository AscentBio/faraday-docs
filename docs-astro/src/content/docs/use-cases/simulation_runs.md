---
title: Simulation runs
description: Run repeatable experiments from the same starting file state by using an isolated workspace copy per run.
---

Use this when you want to run Faraday many times against the same files without mutating the original source directory.

This is useful for simulation-style workflows, benchmarks, agent evaluations, and repeated scientific runs where each run should start from the same initial filesystem state.

## What this gives you

- A clean workspace copy for each run
- No persistent edits to the original source folder
- Comparable results across repeated runs
- Optional access to the copied workspace after the run for debugging
- Standard Faraday outputs in `run_outputs/`

## Why this matters

Many agent tasks read and write files as part of the workflow. That is often desirable for a normal interactive session, but it becomes a problem when you want to:

- run 50 prompts against the same starting directory
- compare model or prompt variants fairly
- test retry behavior without drift from earlier runs
- let the agent generate intermediate files without polluting the source workspace

`runtime.workspace.init_mode: copy` solves this by creating an isolated working copy before the run starts. The agent and code sandbox operate on the copy, not on the original files.

## Recommended config

Small path note:

- `./...` paths below are local machine paths in your config
- `/workspace` below is the path seen inside Docker runtimes

```yaml

# Faraday app environment
runtime:
  backend: docker
  workspace:
    source_root: ./ # point to directory to use as the working directory
    init_mode: copy # create an isolated copy of source before working
    copy_root: ./.faraday_runtime/workspace-copies # where to temporarily store the copy
    keep_copy: false # remove the copy after the run

# code sandbox environment
execution:
  backend: docker
  workspace:
    mount_path: /workspace # docker mounted path

outputs:
  root: ./run_outputs # where to store the run outputs after the run

backends:
  db: sqlite
  rag: in-memory

persistence:
  db_messages: true
  atif_trajectory: true
```

## How it works

1. Faraday reads files from `runtime.workspace.source_root`.
2. Because `runtime.workspace.init_mode` is set to `copy`, it creates a fresh working copy under `runtime.workspace.copy_root`.
3. The Docker code sandbox sees that copied workspace at `execution.workspace.mount_path`.
4. The agent runs normally, reading and writing inside the copied workspace.
5. Standard artifacts are still written under `outputs.root`.
6. At cleanup time, the copied workspace is removed unless `runtime.workspace.keep_copy: true`.

## Key settings

### `runtime.workspace.source_root`

The original directory Faraday should start from.

This is a local path, not an in-container path.

Examples:

- `./` for the current working directory
- `./inputs/simulation_case_01`
- `/Users/me/projects/kras-study`

Use this when you want to point the agent at a specific folder rather than wherever you happen to launch the CLI from.

### `execution.workspace.mount_path`

The path the copied workspace should appear at inside Docker runtimes.

This is an in-container path, not a local filesystem path.

For most setups, keep this as:

```yaml
workspace:
  mount_path: /workspace
```

This is the path agent-generated code and Docker sandbox tools will see.

### `runtime.workspace.init_mode`

Controls whether Faraday binds the workspace directly or creates an isolated copy first.

Options:

- `bind`: use the workspace directly
- `copy`: create an isolated workspace copy before the run

Use `copy` for repeatable experiments and `bind` for normal development.

### `runtime.workspace.copy_root`

The parent directory where isolated copies are created.

This is a local path on the machine running Faraday.

Example:

```yaml
copy_root: ./.faraday_runtime/workspace-copies
```

Faraday will create per-run subdirectories inside this location.

Pick a path that:

- has enough disk space
- is easy to clean up
- is separate from your canonical source files

### `runtime.workspace.keep_copy`

Controls whether the copied workspace is kept after the run.

Options:

- `false`: remove the copy during cleanup
- `true`: keep the copy so you can inspect intermediate files

Use `false` for large repeated runs and `true` when debugging agent behavior.

### `outputs.root`

Where normal Faraday run outputs go.

This is usually a local path in your config, even if code execution happens in Docker.

Example:

```yaml
outputs:
  root: ./run_outputs
```

This is separate from the copied workspace itself. It holds the run directory with:

- `agent_outputs/`
- `run_artifacts/`

That separation is helpful because it lets you keep reproducible run outputs even when temporary workspace copies are deleted.

## Typical patterns

### Pattern 1: Repeatable benchmark runs

Use this when comparing prompts, tools, or models on the same inputs.

```yaml
runtime:
  workspace:
    source_root: ./benchmark_case
    init_mode: copy
    copy_root: ./.faraday_runtime/workspace-copies
    keep_copy: false

execution:
  workspace:
    mount_path: /workspace
```

Each run starts from the same `./benchmark_case` contents.

### Pattern 2: Debugging file mutations

Use this when you want to see exactly what the agent changed during a run.

```yaml
runtime:
  workspace:
    source_root: ./simulation_case
    init_mode: copy
    copy_root: ./.faraday_runtime/workspace-copies
    keep_copy: true

execution:
  workspace:
    mount_path: /workspace
```

After the run, inspect the copied workspace under `./.faraday_runtime/workspace-copies/`.

### Pattern 3: Normal development

Use this when you do want the agent to operate directly on the workspace.

```yaml
runtime:
  workspace:
    source_root: ./
    init_mode: bind

execution:
  workspace:
    mount_path: /workspace
```

This is the simplest mode, but it can leave persistent file changes in the source directory.

## Example commands

Run a single repeatable simulation:

```bash
faraday --config examples/configs/batch-literature-queries.yaml \
  "Run the analysis and save intermediate files if helpful."
```

Run a batch:

```bash
faraday --config examples/configs/batch-literature-queries.yaml
```

Because the workspace is copied per run, each batch item starts from the same original file state.

## Where files go

With this config:

```yaml
runtime:
  workspace:
    source_root: ./
    init_mode: copy
    copy_root: ./.faraday_runtime/workspace-copies
    keep_copy: false
execution:
  workspace:
    mount_path: /workspace
outputs:
  root: ./run_outputs
```

you should expect:

- original files stay in `./`
- temporary per-run workspace copies are created under `./.faraday_runtime/workspace-copies/`
- normal run outputs land under `./run_outputs/`

If `runtime.workspace.keep_copy: false`, the temporary copy is removed at the end of the run.

If `runtime.workspace.keep_copy: true`, the temporary copy remains on disk for inspection.

## When to use this

- Running the same experiment many times with different prompts
- Evaluating model behavior on identical starting inputs
- Letting the agent generate or modify files without touching the source folder
- Replaying scientific or engineering workflows from the same initial state
- Batch runs where filesystem drift would invalidate comparisons

## Related pages

- **[YAML guidance](../reference/yaml-guidance/)** for the broader config model
- **[Results and artifacts](../run-queries/results-and-artifacts/)** for output directory layout
- **[Run a single query](../run-queries/run-a-single-query/)** for basic CLI usage
