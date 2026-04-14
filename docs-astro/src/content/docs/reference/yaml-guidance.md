---
title: YAML guidance
description: Practical guidance for choosing a `faraday.yaml` shape for host, Docker, and batch workflows.
---

Use `faraday.yaml` for stable defaults.

Use environment variables and CLI flags for secrets and one-off overrides.

## Start with the mental model

There are two knobs that matter:

| Knob | What it controls | Key |
|------|-----------------|-----|
| **App** | Where the Faraday process runs | `app.mode` |
| **Sandbox** | Where agent-generated code runs | `sandbox.backend` |

The three common combinations:

| Setup | `app.mode` | `sandbox.backend` |
|-------|-----------|-------------------|
| Local + Docker sandbox | `host` | `docker` |
| Docker app + sidecar sandbox | `docker` | `docker` |
| Local + Modal | `host` | `modal` |

If you only remember one thing: `app` = where Faraday lives, `sandbox` = where code runs.

## Recommended baseline

This is a good starting point for most users — local Faraday with a Docker sandbox:

```yaml
llm:
  provider: openai
  model: gpt-5
  api_key_env: OPENAI_API_KEY

backends:
  db: sqlite
  rag: in-memory

app:
  mode: host
  workspace:
    source_root: .

sandbox:
  backend: docker
  workspace:
    container_path: /workspace

outputs:
  root: ./run_outputs

storage:
  sqlite_path: ~/.faraday/faraday.db
  save_messages: true
  save_trajectory: true
```

Why this is a good default:

- `sqlite` works without extra infrastructure.
- `in-memory` gives you workspace-aware retrieval without extra services.
- `app.mode: host` keeps the app easy to debug.
- `sandbox.backend: docker` isolates code execution from your main Python environment.
- `outputs.root` makes run artifacts predictable.
- `storage` keeps everything about persistence in one place.

## Provider setup patterns

### OpenAI

```yaml
llm:
  provider: openai
  model: gpt-5
  api_key_env: OPENAI_API_KEY
```

### Azure OpenAI

```yaml
llm:
  provider: azure
  model: gpt-5
  api_key_env: AZURE_OPENAI_API_KEY
  base_url_env: AZURE_OPENAI_BASE_URL
  api_version: preview
```

### OpenRouter

```yaml
llm:
  provider: openrouter
  model: openai/gpt-4.1-mini
  api_key_env: OPENROUTER_API_KEY
```

Guidance:

- Prefer `api_key_env` over storing secrets in YAML.
- Keep provider-specific values inside the `llm` block.

## Switching between modes

Start from the recommended baseline above. Each mode only requires changing a small number of lines.

### Mode 1: Local + Docker sandbox (the default)

No changes needed — this is the baseline.

```yaml
app:
  mode: host
  workspace:
    source_root: .

sandbox:
  backend: docker
  workspace:
    container_path: /workspace
```

Best for: everyday development, repository work where code should be isolated.

### Mode 2: Docker app + sidecar sandbox

Starting from the baseline, change these lines:

```yaml
app:
  mode: docker              # ← was: host
  app_image: faraday-oss    # ← add: which image to run Faraday in
  workspace:
    source_root: /workspace # ← was: .  (absolute path inside the container)

outputs:
  root: /workspace/run_outputs  # ← add: so outputs land on the bind-mounted volume
```

The `sandbox` block stays the same.

Best for: containerized deployments, consistent runtime across machines, demo environments.

### Mode 3: Local + Modal

Starting from the baseline, change one line and add the modal config:

```yaml
sandbox:
  backend: modal            # ← was: docker
  modal:                    # ← add: Modal-specific settings
    cloud_storage_mode: optional
    bucket_name: my-faraday-bucket
```

The `app` block stays the same. Also set `features.modal: true`.

Best for: remote compute, workflows that already depend on Modal.

### Bonus: Local app, local execution (no Docker)

For the simplest possible setup when Docker is unavailable:

```yaml
app:
  mode: host

sandbox:
  backend: host             # ← was: docker
```

Trade-offs: installed packages and filesystem writes affect the host directly, results may be less reproducible across machines.

## Workspace guidance

### Bind the workspace directly

This is the default behavior and the right choice for most runs. The agent reads and writes files directly at `source_root`.

```yaml
app:
  workspace:
    source_root: .
```

### Use isolated workspace copies

Use this when each run should start from the same clean workspace snapshot.

```yaml
app:
  workspace:
    source_root: .
    init_mode: copy
    copy_root: ./.faraday_runtime/workspace-copies
    keep_copy: false
```

Best for:

- repeated benchmark runs
- long-running experiments
- comparisons where mutations to the workspace should not leak between runs

## Output and storage guidance

Faraday writes per-run artifacts under `outputs.root`.

Typical layout:

```text
run_outputs/
  run_{timestamp}_{chat_id}_{query_id}/
    agent_outputs/
    run_artifacts/
```

Use these defaults unless you have a clear reason not to:

```yaml
outputs:
  root: ./run_outputs

storage:
  sqlite_path: ~/.faraday/faraday.db
  save_messages: true
  save_trajectory: true
```

What each setting is for:

- `save_messages`: persist conversation history so it can be restored across runs
- `save_trajectory`: write `trajectory.json` for replay, debugging, and handoff
- `previous_context`: path to a prior `trajectory.json`; use when a follow-up run should resume from a known trace

## Retrieval and storage guidance

### Simple local retrieval

```yaml
backends:
  db: sqlite
  rag: in-memory

storage:
  sqlite_path: ~/.faraday/faraday.db
```

Use this when:

- you want easy local setup
- you want conversation history
- you want retrieval over the current workspace

### Disable retrieval

```yaml
backends:
  db: sqlite
  rag: none
```

Use this when:

- you want simpler behavior
- you do not want workspace retrieval influencing answers

If you need retrieval over an external corpus instead of the current workspace, see **[Bring your own document store](../tools/bring-your-own-document-store/)**.

## Batch-run guidance

If you regularly run the same prompt set, define it in YAML:

```yaml
batch:
  enabled: true
  prompts:
    - "Summarize the main modules in this repository."
    - "Identify the biggest operational risks."
  max_concurrency: 1
  max_retries: 2
  continue_on_error: false
```

Use `prompts_file` when the list is large:

```yaml
batch:
  enabled: true
  prompts_file: ./prompts.txt
```

Guidance:

- keep `max_concurrency: 1` until you know your provider limits
- raise concurrency only when prompts are independent
- enable `continue_on_error` when you care more about batch completion than fail-fast behavior

## Feature flags

Most users can leave `features` unset.

Reach for it when you need to enable or disable optional runtime dependencies:

```yaml
features:
  modal: false
  exa: false
  python_science_stack: true
  cheminformatics_stack: false
```

Rules of thumb:

- enable Modal only when `sandbox.backend: modal`
- enable the science stack for common plotting and analysis workflows
- enable the cheminformatics stack only when you actually need RDKit or Datamol

## Tool extension guidance

Use `tool_modules` when you want to load custom tools before each run.

```yaml
tool_modules:
  - my_project.faraday_tools
  - ./tools/custom_tools.py
```

This is the right approach when:

- you want reusable project-specific tools
- the same custom tools should be available from both the CLI and SDK

## Keep YAML small

The best Faraday YAML is usually shorter than you expect.

Add a key only when:

- you want behavior different from the default
- a runtime boundary needs an explicit path
- the environment is shared and you want predictable outputs
- a workflow will be reused enough that it deserves a checked-in config

## Common mistakes

### Mixing up `app.mode` and `sandbox.backend`

Symptom:

- Faraday launches in the wrong place, or code executes somewhere unexpected.

Fix:

- `app.mode` = where Faraday runs. `sandbox.backend` = where code runs.
- set both blocks explicitly when you care about predictability

### Storing secrets in YAML

Symptom:

- API keys end up in version control

Fix:

- use `llm.api_key_env`
- export secrets in the shell or `.env`

### Using a relative `source_root` with `app.mode: docker`

Symptom:

- The agent cannot find files, or workspace reads return empty results.

Fix:

- use an absolute path for `source_root` when running inside a container, for example `source_root: /workspace`

## Related pages

- **[YAML reference](./runtime-config/)** for the full field-by-field schema
- **[Environment variables](./environment-variables/)** for overrides and secrets
- **[Docker](../integrations/docker/)** for a container example
- **[Python SDK](../sdk/)** for programmatic usage
