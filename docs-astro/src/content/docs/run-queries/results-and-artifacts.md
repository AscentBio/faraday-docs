---
title: Results and artifacts
description: Where run outputs go, how to change paths, and how to collect or sync them.
sidebar:
  order: 3
---

## What each run produces

Each run writes a single directory under `run_outputs/` (or another root you configure). Inside it you get agent files plus a fixed set of run metadata:

```text
run_outputs/
  run_{timestamp}_{chat_id}_{query_id}/
    agent_outputs/       # files the agent created while running
    run_artifacts/       # events, final result, trajectory, metadata
      events.jsonl
      result.json
      trajectory.json
      metadata.json
```

The default root is `./run_outputs`. Set `outputs.root` in `faraday.yaml` to use a different base path.

## Trajectory (ATIF) export

`trajectory.json` is written when the run finishes, at:

```text
<outputs.root>/run_{timestamp}_{chat_id}_{query_id}/run_artifacts/trajectory.json
```

To turn trajectories off, set `storage.save_trajectory: false` in config. The run output root still follows `outputs.root` as above.

## CLI: where files land

`faraday "<query>"` writes `events.jsonl`, `result.json`, `metadata.json`, and `trajectory.json` under that run’s `run_artifacts/` folder.

**One-off output location**

```bash
faraday --artifacts-dir /path/to/output "Your task here"
```

**Copy into a second collection directory** (useful for Harbor-style workflows)

- Flag: `--collect-artifacts-dir /path/to/collection`
- Config: `outputs.artifacts.collect_dir: /path/to/collection` in `faraday.yaml`

If `/logs/artifacts` exists (typical in Harbor-style sandboxes), Faraday also collects there and writes a `manifest.json`.

## Using paths from Python

After a run, `FaradayAgent` exposes `agent.run_artifacts_dir` and `agent.agent_outputs_dir`. See **[First query](../sdk/first-query/)** for a minimal example.

## Batch runs

For many prompts with one artifact tree per run, see **[Batch runs](./batch-runs/)**.

## Practical storage

1. Let Faraday write under the working directory (default `run_outputs/`).
2. After the run succeeds, sync the whole `run_{timestamp}_{chat_id}_{query_id}/` folder to durable storage if you need it elsewhere.

This keeps run configuration simple and avoids losing partial uploads if a run fails midway.
