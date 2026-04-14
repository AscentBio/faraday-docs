---
title: First query
description: Run a single prompt with `FaradayAgent` and read the final `solution` event.
---

Smallest useful pattern: construct `FaradayAgent`, stream `agent.run()`, read the `solution` event, and always call `cleanup()`.

## Prerequisites

- `pip install -e .`
- `OPENAI_API_KEY` set in your environment

## Minimal example

```python
import asyncio
import uuid

from faraday.faraday_agent import FaradayAgent


async def main() -> None:
    agent = FaradayAgent(
        model="gpt-5",
        mode="dev",
        chat_id=uuid.uuid4().hex[:32],
        query_id=uuid.uuid4().hex[:8],
        max_total_steps=20,
        execution_backend="docker",
        verbose=False,
        debug_print=False,
    )

    try:
        async for event in agent.run(
            "What is the difference between a kinase inhibitor and a protease inhibitor? Keep your answer brief."
        ):
            if event.get("type") == "solution":
                print(event["content"])
    finally:
        await agent.cleanup()


asyncio.run(main())
```

## What this does

- `FaradayAgent(...)` applies YAML-backed defaults plus the explicit constructor overrides.
- `agent.run(prompt)` is an async generator of structured event dictionaries.
- The final answer normally arrives as `event["type"] == "solution"`.
- `agent.cleanup()` flushes artifacts and tears down runtime resources.

Run artifacts are written automatically to
`run_outputs/run_{timestamp}_{chat_id}_{query_id}/run_artifacts/`.
See **[Results and artifacts](../../run-queries/results-and-artifacts/)** for the full layout.

## Variations

### Run directly on the host

Use this when you want the simplest local setup:

```python
agent = FaradayAgent(execution_backend="host")
```

### Show tool progress while streaming

```python
async for event in agent.run("Inspect the repository structure."):
    event_type = event.get("type")
    if event_type == "tool_plan":
        print("tool:", event.get("display_tool_name") or event.get("tool_call_name"))
    elif event_type == "solution":
        print(event["content"])
```

## When to use this pattern

Use this for one-off prompts, quick local validation, and embedding Faraday inside
a larger async pipeline. For scripted one-shot tasks, `faraday "your query"` is
often simpler.
