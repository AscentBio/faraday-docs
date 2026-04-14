---
title: Follow-up conversations
description: Continue a conversation by reusing the same `chat_id` across multiple runs.
---

To preserve context across turns, keep the same `chat_id` and generate a fresh `query_id` for each run.

## Reuse the same `chat_id`

```python
import asyncio
import uuid

from faraday.faraday_agent import FaradayAgent


async def run_turn(chat_id: str, prompt: str) -> str:
    agent = FaradayAgent(
        model="gpt-5",
        mode="dev",
        chat_id=chat_id,
        query_id=uuid.uuid4().hex[:8],
        max_total_steps=20,
        execution_backend="docker",
        verbose=False,
        debug_print=False,
    )

    solution = ""
    try:
        async for event in agent.run(prompt):
            if event.get("type") == "solution":
                solution = event["content"]
    finally:
        await agent.cleanup()

    return solution


async def main() -> None:
    chat_id = uuid.uuid4().hex[:32]

    first = await run_turn(
        chat_id,
        "Use rdkit to calculate the logp, mw, and qed of warfarin, imatinib, and sotorasib.",
    )
    print(first)

    second = await run_turn(
        chat_id,
        "Now do the same analysis for aspirin and calculate its drug-likeness properties.",
    )
    print(second)


asyncio.run(main())
```

Why this works:

- `chat_id` is the stable conversation key
- `query_id` changes on every turn
- `restore_history_from_storage` defaults to `True`, so Faraday can restore prior context associated with that conversation

## Seed history explicitly

If you already have conversation state in memory and do not want storage-backed restoration, pass it directly:

```python
agent = FaradayAgent(
    chat_id="existing-chat",
    query_id=uuid.uuid4().hex[:8],
    conversation_history=[
        {"role": "user", "content": "Summarize the main kinase inhibitor classes."},
        {"role": "assistant", "content": "Here is a summary..."},
    ],
    restore_history_from_storage=False,
)
```

Use this when:

- your application stores its own conversation state
- you are reconstructing a session from a database or API response
- you want exact control over what prior context is visible

## Start from a saved trajectory file

If you already have a prior run artifact at `trajectory.json`, use it as the source of truth for follow-up context:

```bash
faraday --previous-context run_outputs/run_<timestamp>_<chat_id>_<query_id>/run_artifacts/trajectory.json \
  "Now compare those results to aspirin."
```

This loads prior turns directly from the trajectory `steps` payload and seeds them into context for the new run (no storage lookup by `chat_id` required).

A minimal slice of what Faraday writes in `trajectory.json`. The CLI uses the `steps` array as the prior context source; `agent.extra` is metadata:

```json
    "extra": {
      "chat_id": "chat_20260329223841_016ea32c",
      "query_id": "query_20260329223841_4fa8975d",
      "mode": "dev"
    }
  },
  "steps": [
    {
      "step_id": 1,
      "timestamp": "2026-03-29T22:38:45.558000Z",
      "source": "user",
      "message": "hello"
    },
    {
      "step_id": 2,
      "timestamp": "2026-03-29T22:38:51.817000Z",
      "source": "agent",
      "message": "<thought>The user's input is simply a greeting...</thought>",
      "reasoning_content": "<thought>The user's input is simply a greeting...</thought>"
    }
```

You can still continue by ID instead:

```bash
faraday --chat-id <existing_chat_id> \
  "Now compare those results to aspirin."
```

## Key ideas

- Keep `chat_id` stable for the conversation.
- Generate a fresh `query_id` for each turn.
- Let storage restore history when you want the normal Faraday follow-up behavior.
- Use `conversation_history` plus `restore_history_from_storage=False` when your app owns the context explicitly.
- Use `--previous-context` when your handoff artifact is a saved trajectory rather than storage-backed chat history.
