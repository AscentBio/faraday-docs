---
title: Batch runs and traces
description: Run many prompts with the CLI and collect per-run artifacts automatically.
draft: true
---

After single-query runs work, switch to CLI-managed batch runs.

## Recommended flow

Use one of these entry points:

- CLI flags (`--batch-file` / `--batch-query`) for ad hoc batches.
- `batch` section in `faraday.yaml` for repeatable runs in CI or orchestrators.

Each prompt runs as an isolated query with its own artifact directory.

## Batch via CLI flags

Run from a file:

```bash
faraday --config /path/to/faraday.yaml \
  --batch-file ./prompts.txt
```

Run from repeated flags:

```bash
faraday \
  --batch-query "Summarize the key claims in this project." \
  --batch-query "List the main experimental risks in this repository." \
  --batch-query "Identify files most relevant for onboarding."
```

Useful options:

- `--batch-output-root /path/to/traces`
- `--batch-continue-on-error`

Supported batch file formats:

- `.txt` (one prompt per line; blank lines and `#` comments ignored)
- `.json` (array of prompt strings)
- `.jsonl` (one prompt per line)

## Batch via YAML config

Add a `batch` block:

```yaml
batch:
  enabled: true
  prompts:
    - "Summarize the key claims in this project."
    - "List the main experimental risks in this repository."
  prompts_file: ""
  output_root: /workspace/run_outputs
  continue_on_error: false
```

Then run:

```bash
faraday --config /path/to/faraday.yaml
```

If CLI batch flags are provided, they take precedence over `batch.prompts` and `batch.prompts_file`.

## Artifact layout

```text
run_outputs/
  batch_20260329224510_ab12cd34/
    batch_summary.json
    run_001/
      agent_outputs/
      run_artifacts/
        events.jsonl
        result.json
        trajectory.json
        metadata.json
    run_002/
      agent_outputs/
      run_artifacts/
        events.jsonl
        result.json
        trajectory.json
        metadata.json
```

`batch_summary.json` records `run_dir`, `run_artifacts_dir`, and `agent_outputs_dir`
for each prompt so downstream tools can locate outputs without path reconstruction.

## Why this is simpler

- No custom Python driver required for normal batch runs.
- Same `faraday` command works for one query or many prompts.
- Config-driven batches are easy to automate in Docker/CI.
