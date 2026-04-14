---
title: Environment variables
description: Environment variables used by Faraday for config discovery and provider credentials.
---

Use environment variables for secrets and one-off runtime bootstrapping.

Prefer `faraday.yaml` for runtime behavior (model, execution, storage, trajectories, artifacts).
Prefer env vars for API keys and provider credentials.

## Most common

- `OPENAI_API_KEY`: common API key for `llm.provider: openai`
- `FARADAY_CONFIG`: path to a specific `faraday.yaml`.
- `AZURE_OPENAI_BASE_URL`: common Azure base URL when `llm.provider: azure`
- `OPENROUTER_API_KEY`: API key when `llm.provider: openrouter`

## LLM provider setup

Use `faraday.yaml` to choose the provider, then use environment variables for secrets.

OpenAI example:

```yaml
llm:
  provider: openai
  model: gpt-5
  api_key_env: OPENAI_API_KEY
```

Azure example:

```yaml
llm:
  provider: azure
  model: gpt-5
  api_key: OPENAI_API_KEY
  base_url: AZURE_OPENAI_BASE_URL
  api_version: preview
```

OpenRouter example:

```yaml
llm:
  provider: openrouter
  model: openai/gpt-4.1-mini
  api_key_env: OPENROUTER_API_KEY
```

In the `llm` block:

- `api_key_env` means "read the API key from this environment variable"
- `model` is the default model name for the agent and CLI
- `api_key` can be either the literal key or an env var name like `OPENAI_API_KEY`
- `base_url` can be either the literal URL or an env var name like `AZURE_OPENAI_BASE_URL`
- `base_url_env` is the explicit version if you prefer to separate the env var name from the value

Common provider-related env vars:

- `OPENAI_API_KEY`
- `AZURE_OPENAI_API_KEY`
- `AZURE_OPENAI_BASE_URL`
- `AZURE_OPENAI_API_VERSION`
- `OPENROUTER_API_KEY`

## Config discovery

- `FARADAY_CONFIG`: load this exact config file.

Faraday checks config in this order:

1. `--config`
2. `FARADAY_CONFIG`
3. Container defaults such as `/app/config/faraday.yaml`
4. Local defaults such as `./faraday.yaml` and `~/.faraday/config.yaml`

## Storage and persistence

- `ENABLE_DB_MESSAGE_STORE`: enable DB-backed message persistence. Use `1` or `0`.

## Retrieval and RAG

- `ENABLE_IN_MEMORY_RAG`: force-enable or disable in-memory RAG with `1` or `0`.
- `ENABLE_TURBOPUFFER_STORE`: enable or disable Turbopuffer-backed storage with `1` or `0`.
- `TURBOPUFFER_KEY`: required for Turbopuffer-backed features.

Use these only if you are changing the default retrieval behavior. The common built-in document-store examples are:

- `in-memory`: simplest local retrieval
- `turbopuffer`: managed retrieval for normal Faraday workspace/file search

## Trajectories

- `ENABLE_ATIF_TRAJECTORY`: enable or disable trajectory export with `1` or `0`.

## Modal cloud bucket mounts (optional)

If you set `sandbox.backend: modal` and `sandbox.advanced_config.modal.cloud_storage_mode` to `optional` or `required`, Faraday may mount a cloud bucket inside the Modal sandbox. In that case you must supply the credential environment variables expected for that mount (for typical S3-compatible setups this includes `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`, and optionally `AWS_SESSION_TOKEN` or `AWS_DEFAULT_REGION` as needed). With `cloud_storage_mode: disabled` (the default), these are not required.

## Tool-specific provider keys

Some tools need extra credentials beyond the base agent setup:

- `EXA_API_KEY`: web and literature search tools.

Memory, summarization, and some web-search tool paths use the same active `llm` provider config as the agent.

If you are unsure which keys are missing, run:

```bash
faraday --check-tools
```

## Advanced

- `DB_FLUSH_BATCH_SIZE`: flush size for DB persistence. Advanced tuning only.
- `DB_FLUSH_INTERVAL`: flush interval in seconds for DB persistence. Advanced tuning only.
- `ENV_SETTING_TYPE`: internal mode flag set by Faraday. Do not set this manually unless you know why.

## Example

```bash
export OPENAI_API_KEY=...
export FARADAY_CONFIG=/path/to/faraday.yaml

faraday "Summarize the files in this workspace."
```
