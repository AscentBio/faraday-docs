---
title: Python SDK
description: Reference for using `FaradayAgent` directly from Python.
---

Use the SDK when you want to drive Faraday from Python instead of the CLI.

It is the right interface when you need to:

- embed Faraday in an async application or pipeline
- stream structured events instead of terminal output
- control `chat_id`, `query_id`, and runtime overrides in code
- load custom tool modules or custom conversation state programmatically

For one-off usage, the CLI is usually simpler.

## Import path

Use the current import path:

```python
from faraday.faraday_agent import FaradayAgent
```

## Quickstart

```python
import asyncio
import uuid

from faraday.faraday_agent import FaradayAgent


async def main() -> None:
    agent = FaradayAgent(
        model="gpt-5",
        chat_id=uuid.uuid4().hex[:32],
        query_id=uuid.uuid4().hex[:8],
        max_total_steps=20,
        execution_backend="docker",
        verbose=False,
        debug_print=False,
    )

    try:
        async for event in agent.run("Summarize the current repository."):
            if event.get("type") == "solution":
                print(event["content"])
    finally:
        await agent.cleanup()


asyncio.run(main())
```

## API shape

The Python surface area is intentionally small:

- `FaradayAgent(...)` constructs an agent instance
- `agent.run(prompt)` returns an async event stream
- `await agent.cleanup()` flushes artifacts, persistence, and runtime cleanup

The SDK reads the same `faraday.yaml` config used by the CLI. Constructor arguments act as per-run overrides.

## `FaradayAgent`

### Constructor

Commonly used parameters:

```python
FaradayAgent(
    model="gpt-5",
    mode="dev",
    chat_id=None,
    query_id=None,
    conversation_history=None,
    restore_history_from_storage=True,
    execution_backend=None,
    app_runtime=None,
    workspace_source_root=None,
    workspace_mount_path=None,
    max_total_steps=50,
    verbose=True,
    debug_print=False,
    extra_tools=None,
    extra_tool_handlers=None,
)
```

### Core identity parameters

#### `model`

Model name for this run. If omitted, Faraday uses the model resolved from YAML.

```python
agent = FaradayAgent(model="gpt-5")
```

#### `mode`

Agent mode, typically `dev` or `prod`.

```python
agent = FaradayAgent(mode="dev")
```

#### `chat_id`

Stable conversation identifier. Reuse it across turns when you want a follow-up conversation.

```python
agent = FaradayAgent(chat_id="screening-session-001")
```

#### `query_id`

Unique run identifier for a single turn. You usually want a fresh value on each run.

```python
agent = FaradayAgent(query_id=uuid.uuid4().hex[:8])
```

### Conversation parameters

#### `conversation_history`

Optional pre-seeded conversation payload. Use this when you want to start from already-constructed history rather than restoring from storage.

```python
agent = FaradayAgent(
    conversation_history=[
        {"role": "user", "content": "Summarize KRAS G12C resistance mechanisms."},
        {"role": "assistant", "content": "Here is a summary..."},
    ],
    restore_history_from_storage=False,
)
```

#### `restore_history_from_storage`

Whether Faraday should restore prior context associated with the current identifiers from storage-backed persistence.

Default: `True`

Set this to `False` when you are explicitly seeding `conversation_history` and do not want storage lookup to add more history.

### Runtime override parameters

#### `execution_backend`

Per-run override for `execution.backend`.

Supported normalized values:

- `docker`
- `host`
- `modal`
- `disabled`

Aliases:

- `local` -> `host`
- `harbor` -> `docker`

```python
agent = FaradayAgent(execution_backend="host")
```

#### `app_runtime`

Per-run override for where the Faraday app runs.

Supported values:

- `host`
- `docker`

```python
agent = FaradayAgent(app_runtime="host")
```

#### `workspace_source_root`

Per-run workspace source root. Useful when embedding Faraday from a different current working directory than the target workspace.

```python
agent = FaradayAgent(workspace_source_root="/path/to/project")
```

#### `workspace_mount_path`

Per-run mount path for Docker-based runtimes.

```python
agent = FaradayAgent(workspace_mount_path="/workspace")
```

### Execution and verbosity parameters

#### `max_total_steps`

Upper bound on total agent steps before the run stops.

Default: `50`

```python
agent = FaradayAgent(max_total_steps=20)
```

#### `verbose`

Controls internal logging verbosity.

```python
agent = FaradayAgent(verbose=False)
```

#### `debug_print`

Enables additional debugging output.

```python
agent = FaradayAgent(debug_print=True)
```

### Extension parameters

#### `extra_tools`

Optional list of tool specs to register for this agent instance.

#### `extra_tool_handlers`

Mapping of tool name to Python callable for the corresponding `extra_tools`.

Use these when you want SDK-only tool extensions instead of YAML-based `tool_modules`.

## `FaradayAgent.run()`

### Signature

```python
async for event in agent.run(task_string, user_added_files=[]):
    ...
```

### Parameters

#### `task_string`

The user prompt or task instruction.

```python
async for event in agent.run("Review the main modules in this repository."):
    ...
```

#### `user_added_files`

Optional list of user-supplied files to associate with the run.

Most SDK users can ignore this parameter.

### Return value

`run()` is an async generator of event dictionaries. The final answer is typically the event where `event["type"] == "solution"`.

Example:

```python
solution = ""

async for event in agent.run("Explain the current Docker integration."):
    if event.get("type") == "solution":
        solution = event.get("content", "")
```

## Event stream

The event stream is the main SDK contract.

Common event types you should expect:

- `thought`: model reasoning or progress-style thought output
- `feedback`: user-facing interim commentary
- `tool_plan`: a tool invocation is starting
- `tool_output`: a tool finished and returned output
- `update`: progress update
- `warning`: non-fatal issue
- `error`: run or tool error
- `solution`: final answer content

Minimal consumer:

```python
async for event in agent.run(prompt):
    event_type = event.get("type")
    if event_type == "tool_plan":
        print("tool:", event.get("display_tool_name") or event.get("tool_call_name"))
    elif event_type == "solution":
        print("answer:", event.get("content"))
```

Guidance:

- key off `event["type"]`, not terminal formatting assumptions
- treat unknown event types as forward-compatible extras
- use `solution` as the normal completion signal for downstream systems

## `FaradayAgent.cleanup()`

Always call `await agent.cleanup()` in a `finally` block.

Cleanup is responsible for:

- saving message history
- saving `trajectory.json` when enabled
- syncing `agent_outputs/` into the run directory
- shutting down sandbox/runtime resources

Pattern:

```python
agent = FaradayAgent(...)

try:
    async for event in agent.run(prompt):
        ...
finally:
    await agent.cleanup()
```

## Artifact behavior

SDK runs write into the same artifact layout as the CLI.

Typical layout:

```text
run_outputs/
  run_{timestamp}_{chat_id}_{query_id}/
    agent_outputs/
    run_artifacts/
```

The exact root is controlled by `outputs.root` in YAML unless you override the underlying runtime settings elsewhere.

See **[Results and artifacts](../run-queries/results-and-artifacts/)** for artifact details.

## Conversation patterns

### One-off prompt

Create a fresh `chat_id` and `query_id`.

### Follow-up conversation

Reuse `chat_id`, generate a fresh `query_id`, and let storage-backed history restore the prior context.

### Explicit seeded history

Pass `conversation_history` and set `restore_history_from_storage=False`.

See **[Follow-up conversations](./follow-up-conversations/)** for concrete patterns.

## CLI vs SDK

Prefer the CLI when:

- you want terminal-first usage
- YAML config is enough
- you want built-in artifact handling without writing Python

Prefer the SDK when:

- you need structured events in code
- Faraday is one step in a larger application workflow
- you want to manage IDs, prompts, and custom tools programmatically

## Related pages

- **[First query](./first-query/)** for the smallest useful example
- **[Follow-up conversations](./follow-up-conversations/)** for multi-turn usage
- **[YAML reference](../reference/runtime-config/)** for config keys shared with the CLI
