---
title: First query
description: Run a single prompt with FaradayAgent and print the final answer.
draft: true
---

Smallest useful pattern: create an agent, stream `agent.run()`, read the `solution` event.

## Prerequisites

- `pip install -e .`
- `OPENAI_API_KEY` set in your environment

## Minimal example

```python
import asyncio
import uuid

from faraday.faraday_agent import FaradayAgent


async def main():
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
- `agent.run(prompt)` streams structured events.
- Final answer arrives as `type == "solution"`.
- `agent.cleanup()` flushes artifacts and releases runtime resources.

## When to use this pattern

Use this for one-off prompts and quick local validation before adding artifact capture or batching.
