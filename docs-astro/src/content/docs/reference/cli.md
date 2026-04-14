---
title: CLI reference
description: Common Faraday CLI commands and flags.
---

## Basic commands

One-shot task:

```bash
faraday "Summarize the files in this workspace."
```

Interactive mode:

```bash
faraday
```

Explicit config path:

```bash
faraday --config /path/to/faraday.yaml "Your task here"
```

Verify tools and credentials:

```bash
faraday --check-tools
```

Batch from prompt file:

```bash
faraday --batch-file ./prompts.txt
```

## Useful flags

Common flags:

- `--config`
- `--model`
- `--max-steps`
- `--debug`
- `--check-tools`
- `--app-mode`
- `--sandbox-backend`
- `--workspace-source-root`
- `--workspace-container-path`
- `--previous-context`
- `--artifacts-dir`
- `--no-artifacts`
- `--collect-artifacts-dir`
- `--batch-file`
- `--batch-query`
- `--batch-continue-on-error`
- `--use-docker`
- `--docker-image`

> **Deprecated:** `--trajectory-path` is no longer needed. Trajectory output is
> written automatically to `run_artifacts/trajectory.json` inside the per-run
> directory. Use `outputs.root` in `faraday.yaml` to control the root.

Run `faraday --help` for full details.

## Environment variables

Common runtime variables:

- `FARADAY_CONFIG`
- model/provider credentials (for example `OPENAI_API_KEY`)

Keep secrets in environment variables, not in `faraday.yaml`.

For the full list, see **[Environment variables](./environment-variables/)**.
