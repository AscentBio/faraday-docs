---
title: Follow-up conversations
description: Continue a conversation by reusing the same chat ID across multiple queries.
draft: true
---

To preserve context across turns, keep the same `chat_id`.

## Load context using conversation identifiers

```python
import asyncio
import uuid

from faraday.faraday_agent import FaradayAgent


async def run_turn(chat_id: str, prompt: str) -> None:
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

    try:
        async for event in agent.run(prompt):
            if event.get("type") == "solution":
                print(event["content"])
    finally:
        await agent.cleanup()


async def main():
    chat_id = uuid.uuid4().hex[:32]

    await run_turn(
        chat_id,
        "Use rdkit to calculate the logp, mw, and qed of warfarin, imatinib, and sotorasib.",
    )

    await run_turn(
        chat_id,
        "Now do the same analysis for aspirin and calculate its drug-likeness properties.",
    )


asyncio.run(main())
```

## Load context from a trajectory file

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
      "message": "<thought>The user\u2019s input is simply a greeting (\u201chello\u201d) with no scientific or task context. In this situation, the best next move is to acknowledge readiness and briefly frame what I can do within computational chemistry/biology drug-discovery workflows, without interrogating them for details. I should keep it lightweight and avoid over-scoping, while signaling capability to proceed once a concrete objective is provided.</thought>",
      "reasoning_content": "<thought>The user\u2019s input is simply a greeting (\u201chello\u201d) with no scientific or task context. In this situation, the best next move is to acknowledge readiness and briefly frame what I can do within computational chemistry/biology drug-discovery workflows, without interrogating them for details. I should keep it lightweight and avoid over-scoping, while signaling capability to proceed once a concrete objective is provided.</thought>"
    },
    {
      "step_id": 3,
      "timestamp": "2026-03-29T22:38:52.411000Z",
      "source": "agent",
      "message": "<feedback>I\u2019m here and ready to help. Within drug discovery, I can support things like target/background literature triage, hit/lead structure lookups (e.g., SMILES), binding/assay context synthesis, and computational strategy planning (docking/MD/QSAR/ADMET)\u2014once there\u2019s a specific objective to work on.</feedback>"
    },
    {
      "step_id": 4,
      "timestamp": "2026-03-29T22:38:56.057000Z",
      "source": "agent",
      "message": "<solution>Hi\u2014Faraday here. What would you like to work on today? \n- Option A: Identify or triage a target (biology and tractability)\n- Option B: Find or design molecules (with SMILES and visuals)\n- Option C: Build a quick ADMET/QSAR screen\n- Option D: Summarize recent papers or patents on a topic\nIf you already have a compound, target, or dataset, share it and I\u2019ll dive right in.</solution>"
    }
```

You can still override manually:

```bash
faraday --chat-id <existing_chat_id> \
  "Now compare those results to aspirin."
```

## Key ideas

- Keep `chat_id` stable for the conversation.
- Generate a fresh `query_id` for each turn.
- Use `--previous-context` when your handoff artifact is a trace file.
- You can recreate the agent per request. Context is keyed by IDs, not only by a long-lived Python object.
