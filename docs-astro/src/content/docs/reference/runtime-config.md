---
title: YAML reference
description: Complete reference for `faraday.yaml`, including supported fields, defaults, and resolution rules.
---

Use this page as the schema-level reference for `faraday.yaml`.

All fields are optional unless you need a non-default behavior. CLI flags win over YAML when both are set.

For practical setup recipes, see **[YAML guidance](./yaml-guidance/)**.

## Syntax

```yaml
name: faraday-reference
query: "Optional default prompt when no CLI query is provided."

llm:
  provider: openai
  model: gpt-5
  api_key_env: OPENAI_API_KEY
  # base_url: OPENAI_BASE_URL
  # base_url_env: OPENAI_BASE_URL
  # api_version: preview
  # api_version_env: AZURE_OPENAI_API_VERSION
  # organization: OPENAI_ORG_ID
  # project: OPENAI_PROJECT_ID
  # timeout: 120
  # max_retries: 3
  # default_query: {}
  # default_headers: {}

backends:
  db: sqlite
  rag: in-memory

app:
  mode: host                # host | docker
  # app_image: faraday-oss
  workspace:
    source_root: .
    # init_mode: bind       # bind | copy
    # copy_root: ./.faraday_runtime/workspace-copies
    # keep_copy: false

sandbox:
  backend: docker           # docker | modal | host | disabled
  # docker_image: faraday-code-sandbox
  workspace:
    container_path: /workspace
  # advanced_config:
  #   allow_host_fallback: false
  #   allow_backend_fallback: true
  #   fallback_order: [docker, host]
  #   import_check: true
  #   import_check_warn_optional: false
  #   auto_install_missing_modules: false
  #   auto_install_missing_modules_timeout_sec: 300
  #   modal:
  #     cloud_storage_mode: disabled
  #     cloud_storage_root: ./.faraday_runtime/cloud-storage
  #     enable_bucket_mount: false
  #     bucket_name: ""
  #   policy:
  #     sandbox_ready_timeout_sec: 120
  #     python_exec_timeout_sec: 300
  #     bash_exec_timeout_sec: 60
  #     transient_retry_attempts: 1
  #     reinitialize_on_transient_failure: true
  #     max_reinitialize_attempts: 3
  #     stdout_max_bytes: 262144
  #     stderr_max_bytes: 32768
  #     heartbeat_interval_sec: 30
  #     docker_memory: 8g
  #     docker_cpus: 4
  #     docker_pids_limit: 1024
  #     docker_shm_size: 1g
  #     docker_network: bridge

outputs:
  root: ./run_outputs
  # artifacts:
  #   collect_dir: ""
  #   collect_to_harbor: false
  #   harbor_dir: /logs/artifacts

storage:
  sqlite_path: ~/.faraday/faraday.db
  save_messages: true
  save_trajectory: true
  # previous_context: ""

batch:
  enabled: false
  prompts: []
  # prompts_file: ""
  max_concurrency: 1
  max_retries: 2
  retry_base_delay_seconds: 2.0
  retry_max_delay_seconds: 30.0
  retry_jitter_seconds: 0.5
  continue_on_error: false

features:
  modal: false
  exa: false
  python_science_stack: true
  cheminformatics_stack: false

tool_modules:
  - my_project.faraday_tools
  - ./tools/custom_tools.py
```

## Resolution order

Faraday looks for config in this order:

1. `--config /path/to/file.yaml`
2. `FARADAY_CONFIG`
3. `/app/config/faraday.yaml`, `/app/config/faraday.yml`, `/etc/faraday/config.yaml`, `/etc/faraday/config.yml`
4. `./faraday.yaml`, `./faraday.yml`, `~/.faraday/config.yaml`, `~/.faraday/config.yml`

## Top-level fields

### `name`

Optional label for the config profile. It is descriptive only.

```yaml
name: faraday-docker
```

### `query`

Optional default query used when the CLI positional query is omitted.

```yaml
query: "Summarize the repository structure."
```

CLI wins over YAML:

```bash
faraday --config faraday.yaml "Use this prompt instead"
```

## `llm`

Configures the OpenAI-compatible client Faraday uses.

Supported providers:

- `openai`
- `azure`
- `openrouter`

### `llm.provider`

Provider name. Defaults to `openai` when omitted.

```yaml
llm:
  provider: openrouter
```

### `llm.model`

Default model passed to the agent and CLI unless `--model` overrides it.

```yaml
llm:
  model: gpt-5
```

### `llm.api_key_env`

Environment variable name that stores the API key. This is the preferred way to configure secrets.

Provider defaults:

- `openai`: `OPENAI_API_KEY`
- `azure`: `AZURE_OPENAI_API_KEY`
- `openrouter`: `OPENROUTER_API_KEY`

```yaml
llm:
  api_key_env: OPENAI_API_KEY
```

### `llm.base_url`

Base URL for OpenAI-compatible deployments. The value can be a literal URL or the name of an environment variable whose value is the URL.

```yaml
llm:
  base_url: AZURE_OPENAI_BASE_URL
```

### `llm.base_url_env`

Explicit environment variable name for the base URL. This is only consulted when `llm.base_url` is unset.

```yaml
llm:
  base_url_env: AZURE_OPENAI_BASE_URL
```

### `llm.api_version`

API version string. Used most often with Azure. If set, Faraday sends it as the `api-version` default query parameter.

```yaml
llm:
  api_version: preview
```

### `llm.api_version_env`

Environment variable name that stores the API version. Used only when `llm.api_version` is unset.

```yaml
llm:
  api_version_env: AZURE_OPENAI_API_VERSION
```

### `llm.default_query`

Additional query parameters sent with every request.

```yaml
llm:
  default_query:
    api-version: preview
```

### `llm.default_headers`

Additional headers sent with every request.

```yaml
llm:
  default_headers:
    HTTP-Referer: https://your-app.example
    X-Title: Faraday
```

### `llm.organization`

Optional OpenAI organization value. Like `base_url`, this can be a literal string or an env var name.

```yaml
llm:
  organization: OPENAI_ORG_ID
```

### `llm.project`

Optional OpenAI project value. This can be a literal string or an env var name.

```yaml
llm:
  project: OPENAI_PROJECT_ID
```

### `llm.timeout`

Client timeout in seconds.

```yaml
llm:
  timeout: 120
```

### `llm.max_retries`

OpenAI client retry count.

```yaml
llm:
  max_retries: 3
```

## `backends`

Controls persistence and retrieval backends.

### `backends.db`

Supported values:

- `sqlite`
- `auto`

Faraday currently resolves message persistence through SQLite-oriented configuration. `sqlite` is the default practical choice for local and Docker setups.

```yaml
backends:
  db: sqlite
```

### `backends.rag`

Supported values:

- `in-memory`
- `none`
- `auto`

Notes:

- `auto` currently behaves like in-process retrieval.
- `in-memory` is the normal workspace-local default.
- `none` disables retrieval-oriented memory/file search behavior.

```yaml
backends:
  rag: in-memory
```

## `app`

`app` controls where the Faraday process itself runs and how the workspace is prepared before a run starts.

### `app.mode`

Where the Faraday app runs.

Supported values:

- `host`
- `docker`

Default: `host`

Behavior:

- `host` means the Faraday app runs directly in the current Python environment.
- `docker` means the CLI relaunches the app inside a Docker container.

```yaml
app:
  mode: docker
```

### `app.app_image`

Docker image used for the Faraday app when `app.mode: docker` or when `--use-docker` is used.

Default: `faraday-oss`

```yaml
app:
  app_image: faraday-oss
```

### `app.workspace.source_root`

Directory treated as the workspace source for the run. If omitted, Faraday falls back to the current working directory in most host-side flows.

```yaml
app:
  workspace:
    source_root: .
```

### `app.workspace.init_mode`

How Faraday prepares the workspace before the run starts.

Supported values:

- `bind` — use the original workspace directly
- `copy` — create an isolated copy for the run (useful for reproducible benchmark-style runs)

Default: `bind`

```yaml
app:
  workspace:
    init_mode: copy
```

### `app.workspace.copy_root`

Parent directory used for isolated workspace copies when `init_mode: copy`.

Default: `./.faraday_runtime/workspace-copies`

```yaml
app:
  workspace:
    init_mode: copy
    copy_root: ./.faraday_runtime/workspace-copies
```

### `app.workspace.keep_copy`

Whether to keep the copied workspace after cleanup when `init_mode: copy`.

Default: `false`

```yaml
app:
  workspace:
    init_mode: copy
    keep_copy: true
```

## `sandbox`

`sandbox` controls where agent-generated code runs — the execution environment for Python and bash tools.

### `sandbox.backend`

Which execution backend to use for code tools.

Supported values:

- `docker` — run code in an isolated Docker container
- `modal` — run code on Modal
- `host` — run code directly in the host Python environment
- `disabled` — code tools are unavailable

Default: `docker`

```yaml
sandbox:
  backend: host
```

### `sandbox.workspace.container_path`

Path where the workspace is mounted inside the sandbox container.

Default: `/workspace`

```yaml
sandbox:
  workspace:
    container_path: /workspace
```

### `sandbox.docker_image`

Docker image used for code-execution sandboxes.

```yaml
sandbox:
  docker_image: faraday-code-sandbox
```

### `sandbox.advanced_config`

Advanced sandbox settings. Most users do not need to touch these — the defaults work for standard Docker and Modal setups. Reach for `advanced_config` when you need to tune fallback behavior, import checking, auto-install, Modal storage, or resource limits.

#### Fallback behavior

#### `sandbox.advanced_config.allow_host_fallback`

Whether the sandbox may fall back to the host Python environment when the Docker backend encounters edge-case failures.

Default: `false`

#### `sandbox.advanced_config.allow_backend_fallback`

Whether Faraday may fall back to another sandbox backend if the selected one cannot be initialized.

Default: `true`

#### `sandbox.advanced_config.fallback_order`

Ordered list of permitted fallback backends.

Allowed entries: `docker`, `modal`, `host`

Default: `[docker, host]`

#### Import checking

#### `sandbox.advanced_config.import_check`

Whether Faraday checks package availability before code execution.

Default: `true` for `host` and `docker`, `false` otherwise

#### `sandbox.advanced_config.import_check_warn_optional`

When enabled, optional imports may emit warnings instead of staying silent.

Default: `false`

#### Auto-install

#### `sandbox.advanced_config.auto_install_missing_modules`

Whether Faraday may attempt to install missing Python modules automatically.

Default: `false`

#### `sandbox.advanced_config.auto_install_missing_modules_timeout_sec`

Timeout for auto-install attempts.

Default: `300`

#### Modal storage

Only relevant when `sandbox.backend: modal`.

#### `sandbox.advanced_config.modal.cloud_storage_mode`

Controls Modal cloud bucket mounting behavior.

Supported values: `disabled`, `optional`, `required`

Default: `disabled`

#### `sandbox.advanced_config.modal.cloud_storage_root`

Local root used for `/cloud-storage` style path mapping.

Default: `./.faraday_runtime/cloud-storage`

#### `sandbox.advanced_config.modal.enable_bucket_mount`

Whether to mount a cloud bucket into the Modal sandbox.

Default: `false`

#### `sandbox.advanced_config.modal.bucket_name`

Bucket name used with Modal cloud storage mounts.

#### Policy: timeouts

#### `sandbox.advanced_config.policy.sandbox_ready_timeout_sec`

How long to wait for the sandbox to become ready.

Default: `120`

#### `sandbox.advanced_config.policy.python_exec_timeout_sec`

Timeout for Python tool execution.

Default: `300`

#### `sandbox.advanced_config.policy.bash_exec_timeout_sec`

Timeout for bash tool execution.

Default: `60`

#### `sandbox.advanced_config.policy.heartbeat_interval_sec`

Heartbeat interval for sandbox liveness checks.

Default: `30`

#### Policy: retries

#### `sandbox.advanced_config.policy.transient_retry_attempts`

Retries for transient execution failures.

Default: `1`

#### `sandbox.advanced_config.policy.reinitialize_on_transient_failure`

Whether Faraday should reinitialize the sandbox after transient failures.

Default: `true`

#### `sandbox.advanced_config.policy.max_reinitialize_attempts`

Maximum sandbox reinitializations before giving up.

Default: `3`

#### Policy: output limits

#### `sandbox.advanced_config.policy.stdout_max_bytes`

Maximum captured stdout size per tool call.

Default: `262144`

#### `sandbox.advanced_config.policy.stderr_max_bytes`

Maximum captured stderr size per tool call.

Default: `32768`

#### Policy: Docker resource limits

#### `sandbox.advanced_config.policy.docker_memory`

Docker memory limit, for example `8g`.

#### `sandbox.advanced_config.policy.docker_cpus`

Docker CPU limit.

#### `sandbox.advanced_config.policy.docker_pids_limit`

Docker PID limit.

#### `sandbox.advanced_config.policy.docker_shm_size`

Docker shared-memory size, for example `1g`.

#### `sandbox.advanced_config.policy.docker_network`

Docker network name.

Example:

```yaml
sandbox:
  backend: docker
  advanced_config:
    allow_backend_fallback: true
    fallback_order: [docker, host]
    auto_install_missing_modules: true
    policy:
      python_exec_timeout_sec: 600
      bash_exec_timeout_sec: 120
      docker_memory: 8g
      docker_cpus: 4
```

## `outputs`

### `outputs.root`

Root directory for per-run outputs.

Default behavior:

- host runs default to `<cwd>/run_outputs`
- app-container runs default to `<container_path>/run_outputs`

Each run creates:

```text
run_outputs/
  run_{timestamp}_{chat_id}_{query_id}/
    agent_outputs/
    run_artifacts/
```

```yaml
outputs:
  root: ./run_outputs
```

### `outputs.artifacts`

Optional artifact collection settings. Most users do not need this — run artifacts are already written under `outputs.root`. Use `outputs.artifacts` only when you need to copy artifacts to a second location or collect them for Harbor.

#### `outputs.artifacts.collect_dir`

Directory where Faraday copies generated artifacts and writes `manifest.json`.

```yaml
outputs:
  artifacts:
    collect_dir: /tmp/faraday-artifacts
```

#### `outputs.artifacts.collect_to_harbor`

Whether to collect artifacts automatically into the Harbor artifact path.

Default: `false`

#### `outputs.artifacts.harbor_dir`

Harbor artifact collection path.

Default: `/logs/artifacts`

```yaml
outputs:
  artifacts:
    collect_to_harbor: true
    harbor_dir: /logs/artifacts
```

## `storage`

`storage` controls persistent state: where the database lives and what gets saved between runs.

### `storage.sqlite_path`

SQLite database path used when `backends.db: sqlite`.

```yaml
storage:
  sqlite_path: ~/.faraday/faraday.db
```

### `storage.save_messages`

Whether to persist messages to the configured DB backend.

Default: `true`

```yaml
storage:
  save_messages: true
```

### `storage.save_trajectory`

Whether to write `trajectory.json` into `run_artifacts/`.

Default: `true`

```yaml
storage:
  save_trajectory: true
```

### `storage.previous_context`

Default path to a prior `trajectory.json` used for follow-up runs when you do not want to pass `--previous-context` each time.

```yaml
storage:
  previous_context: ./run_outputs/previous/run_artifacts/trajectory.json
```

## `batch`

Batch settings are used when you run `faraday` without explicit batch flags and want the YAML to define the batch behavior.

### `batch.enabled`

Whether the config should run as a batch by default.

Default: `false`

```yaml
batch:
  enabled: true
```

### `batch.prompts`

Inline list of prompt strings.

```yaml
batch:
  prompts:
    - "Summarize the README."
    - "List major risks."
```

### `batch.prompts_file`

Path to a `.txt`, `.json`, or `.jsonl` file of prompts.

```yaml
batch:
  prompts_file: ./prompts.txt
```

### `batch.max_concurrency`

Maximum number of prompts run at once.

Default: `1`

```yaml
batch:
  max_concurrency: 4
```

### `batch.max_retries`

Retry count for retryable batch failures.

Default: `2`

```yaml
batch:
  max_retries: 2
```

### `batch.retry_base_delay_seconds`

Base backoff delay for retries.

Default: `2.0`

### `batch.retry_max_delay_seconds`

Maximum backoff delay.

Default: `30.0`

### `batch.retry_jitter_seconds`

Random jitter added to retry delays.

Default: `0.5`

### `batch.continue_on_error`

Whether the batch continues after a failed prompt.

Default: `false`

```yaml
batch:
  continue_on_error: true
```

## `features`

Feature flags that control which optional dependency sets are included.

### `features.modal`

Include Modal support dependencies.

Default: `true` when `sandbox.backend: modal`, otherwise `false`

### `features.exa`

Include Exa-based tooling dependencies.

Default: `false`

### `features.python_science_stack`

Include common scientific Python packages such as `requests`, `matplotlib`, `seaborn`, and `mygene`.

Default: `true` for `docker` and `host` sandbox backends

### `features.cheminformatics_stack`

Include heavier cheminformatics packages such as `rdkit` and `datamol`.

Default: `false`

Example:

```yaml
features:
  exa: true
  python_science_stack: true
  cheminformatics_stack: true
```

## `tool_modules`

List of extra Python modules loaded before the run.

Each item may be:

- a dotted module path such as `my_project.faraday_tools`
- a filesystem path to a `.py` file such as `./tools/custom_tools.py`

Loaded modules may expose:

- `EXTRA_TOOLS`
- `EXTRA_TOOL_HANDLERS`

Later modules override earlier handlers with the same tool name.

```yaml
tool_modules:
  - my_project.faraday_tools
  - ./tools/custom_tools.py
```

## Related pages

- **[YAML guidance](./yaml-guidance/)** for recommended config patterns
- **[CLI](./cli/)** for flag-level behavior and overrides
- **[Environment variables](./environment-variables/)** for secret management and overrides
- **[Python SDK](../sdk/)** for programmatic usage
