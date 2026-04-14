---
title: Batch runs
description: Run many prompts in one invocation; each prompt gets its own output folder and a summary JSON.
sidebar:
  order: 4
---

Run a list of prompts in one invocation. Each prompt gets its own `run_XXX/` directory with separate `agent_outputs/` and `run_artifacts/`. A `batch_summary.json` at the root indexes every run's directories when the batch finishes.

## Quickstart

The fastest way to run a batch is with a plain text file (one prompt per line):

```bash
faraday --batch-file ./prompts.txt
```

Or pass prompts directly on the command line:

```bash
faraday \
  --batch-query "Summarize the known binding mechanisms and resistance mutations associated with KRAS G12C inhibitors, and propose 2–3 structural hypotheses for next-generation inhibitor design" \
  --batch-query "Based on recent literature, what are plausible mechanisms by which PROTACs fail to degrade target proteins, and how could linker design mitigate these issues?" \
  --batch-query "Given typical kinase inhibitor scaffolds, what functional group modifications are most likely to improve selectivity over off-target kinases?"
```

By default, batch runs use `max_concurrency: 1`, so prompts run one at a time. To run prompts in parallel for a single invocation:

```bash
faraday --batch-file ./prompts.txt --batch-max-concurrency 4
```

## Defining a batch in `faraday.yaml`

For repeatable runs in CI or Docker, declare the batch in config instead:

```yaml
batch:
  enabled: true
  prompts:
    - "Summarize the known binding mechanisms and resistance mutations associated with KRAS G12C inhibitors, and propose 2–3 structural hypotheses for next-generation inhibitor design"
    - "Based on recent literature, what are plausible mechanisms by which PROTACs fail to degrade target proteins, and how could linker design mitigate these issues?"
  max_concurrency: 4
  continue_on_error: false
```

Then run as normal:

```bash
faraday --config /path/to/faraday.yaml
```

You can also point at a prompt file with `prompts_file: ./prompts.txt`. Inline `prompts` and `prompts_file` are merged. If you pass any `--batch-file` or `--batch-query` flags, they take precedence and config prompts are ignored for that invocation.

## Running prompts in parallel

Use parallel batch execution only when prompts are independent:

- they should not rely on files written by another prompt in the same batch
- they should not expect shared mutable workspace state
- they should fit within your model provider's rate limits

You can control concurrency either with CLI or YAML:

```bash
faraday --batch-file ./prompts.txt --batch-max-concurrency 4
```

```yaml
batch:
  max_concurrency: 4
```

If your prompts may read and write the workspace, consider enabling isolated workspace copies:

```yaml
app:
  workspace:
    source_root: .
    init_mode: copy
    copy_root: ./.faraday_runtime/workspace-copies
    keep_copy: false
```

> **Warning**
> If `app.workspace.init_mode` is not set to `copy`, concurrent runs still get separate output directories, but they share the same bound workspace. That means prompts can interfere with each other through file reads and writes.

## Prompt file formats

- **`.txt`** — one prompt per non-empty line; lines starting with `#` are skipped.
- **`.json`** — JSON array of strings.
- **`.jsonl`** — one prompt string per non-empty line.

## Output location

Batch output lands under `./run_outputs/` by default, following `outputs.root` in `faraday.yaml`.

The full layout:

```text
<batch_output_root>/
  batch_<timestamp>_<id>/
    batch_summary.json
    run_001/
      agent_outputs/
      run_artifacts/
        events.jsonl
        result.json
        trajectory.json
        metadata.json
    run_002/
      ...
```

`batch_summary.json` records each run's `status`, `exit_code`, `run_dir`, `run_artifacts_dir`, and `agent_outputs_dir`.

## Handling failures

By default, a failing prompt stops the batch. To keep going:

```bash
faraday --batch-file prompts.txt --batch-continue-on-error
```

or set `continue_on_error: true` in the `batch:` config block. Either one enables continuation.
