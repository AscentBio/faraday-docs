---
title: Install
description: Install Faraday OSS locally and verify the CLI is available.
---

## Prerequisites

- Python 3.10+
- A model API key: `OPENAI_API_KEY` (default provider), or `OPENROUTER_API_KEY` if you use [OpenRouter](https://openrouter.ai/), or Azure credentials if you use Azure OpenAI
- Optional tool keys depending on your toolset (for example `EXA_API_KEY` for web/literature search)

## Install from source

```bash
pip install -e .
```

## Verify the CLI

```bash
faraday --help
```

## Set LLM Provider Credentials

### OpenAI (default)

```bash
export OPENAI_API_KEY=...
```

In `faraday.yaml`, the default matches the [example config](https://github.com/AscentBio/faraday-oss/blob/main/faraday.example.yaml) (`llm.provider: openai`, `api_key_env: OPENAI_API_KEY`).

### OpenRouter

[OpenRouter](https://openrouter.ai/) exposes an OpenAI-compatible API. Faraday uses the base URL `https://openrouter.ai/api/v1` when `llm.provider` is `openrouter`.

1. Create a key in the OpenRouter dashboard and export it:

   ```bash
   export OPENROUTER_API_KEY=...
   ```

2. Point `faraday.yaml` at OpenRouter and choose a [model id](https://openrouter.ai/models) (for example `openai/gpt-4o` or `anthropic/claude-3.5-sonnet`):

   ```yaml
   llm:
     provider: openrouter
     model: openai/gpt-4o
     api_key_env: OPENROUTER_API_KEY
   ```

3. Optional: OpenRouter recommends `HTTP-Referer` and `X-Title` headers for rankings and analytics. You can set them under `llm.default_headers` (see comments in `faraday.example.yaml`).

Do not put secret values in YAML; keep the key in the environment variable named by `api_key_env`.

## Check required API keys and tool credentials

Run:

```bash
faraday --check-tools
```

This command checks these keys:

- Agent / LLM: `faraday --check-tools` always lists `OPENAI_API_KEY` under agent keys. If you use OpenRouter only, configure `llm.provider: openrouter` and `OPENROUTER_API_KEY` as above; the main agent uses that key. Some optional tools still declare `OPENAI_API_KEY` in their own rows if they rely on it.
- Tool-dependent: `EXA_API_KEY`
- Modal backend only: `MODAL_TOKEN_ID`, `MODAL_TOKEN_SECRET`


## Setting Up Code Execution

### Docker-based Code Sandbox (recommended)



**Step 1: Build docker images for the Faraday app and code execution sandbox**

```bash
docker build -f Dockerfile.main -t faraday-oss .
docker build -f Dockerfile.sandbox -t faraday-code-sandbox .
```
**Step 2: Configure faraday.yaml to use docker backend**
```yaml
execution:
  backend: docker
  workspace_root: /path/to/agent_workspace/
```


**Step 3: Set workspace**

You have two options for setting the code sandbox workspace
- Using your local filesystem as the workspace
- Using a copy of your local filesystem as the workspace in a ctonainer.  


---

Option 1: Using your local filesystem as the workspace

If the `workspace_root` is not set, Faraday will use the current working directory, mounted into the code sandbox, in it's work. The specific path will set the agent's workspace root for the code sandbox.

Use this when you are fine with changes to your local filesystem.

```yaml
execution:
  backend: docker
  workspace_root: /path/to/agent_workspace/ # or leave blank to use the current working directory
```



--- 

Option 2: Use a copy of your local filesystem as the workspace (safer)

Combined with setting the Faraday agent to run in docker, this will create a copy of your local `workspace_root` directory as the workspace for the code sandbox. 
```yaml
runtime:
  backend: docker # host | docker

execution:
  backend: docker
  workspace_root: /path/to/agent_workspace/ # or leave blank to use the current working directory
```


### Modal-based Code Sandbox

Use the [Modal sandbox](https://modal.com/docs/guide/sandbox) feature for code execution.

- `modal`: runs code execution in Modal sandboxes; useful for remote isolated execution



Configure faraday.yaml to use modal backend:
(in `faraday.example.yaml`)
```yaml
execution:
  backend: modal
```


Set credentials and then validate:
```bash
export MODAL_TOKEN_ID=...
export MODAL_TOKEN_SECRET=...
faraday --check-tools
```

### Run code locally 

Run code using your local python / bash interpreter.

This is generally not recommended due to variability in the execution environment.

```yaml
execution:
  backend: host
  workspace_root: .
```

## Next step

Go to **[Quickstart](./getting-started/)** to run your first task.
