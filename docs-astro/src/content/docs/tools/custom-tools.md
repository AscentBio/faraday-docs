---
title: Adding a custom tool
description: Define a function tool and register it with Faraday via the Python SDK, CLI, or YAML config.
sidebar:
  order: 3
---

Every custom tool has two parts: a **tool definition** (the function schema the model uses to decide when and how to issue a tool call) and a **handler** (the Python function that runs when that tool call arrives). Once you have both, choose one of three ways to register them with Faraday.

## Define the tool and handler

The tool definition's `name` must exactly match the handler key used during registration — Faraday validates this at startup and raises an error on a mismatch.

The handler receives the function arguments from the tool call as keyword arguments and must return `{"markdown_output": <str>}`. That string is what the agent receives as the tool output.

```python
my_tool = {
    "type": "function",
    "name": "my_tool",
    "description": "Action-oriented description the model uses to decide when to call this tool.",
    "parameters": {
        "type": "object",
        "strict": True,
        "properties": {
            "query": {"type": "string", "description": "The input query."},
        },
        "required": ["query"],
    },
}


def my_tool_fn(query: str) -> dict:
    return {"markdown_output": f"Result for {query!r}: ..."}
```


> **Tip:** For helpful guidance on tool design, we recommend reading [Writing Effective Tools for Agents](https://www.anthropic.com/engineering/writing-effective-tools-for-agents).

## Register the tool

Pick the method that matches how you run Faraday. You only need one.

### Python SDK

Pass the tool definition and handler directly to `FaradayAgent`:

```python
from faraday.faraday_agent import FaradayAgent

agent = FaradayAgent(
    extra_tools=[my_tool],
    extra_tool_handlers={"my_tool": my_tool_fn},
)
```

### CLI (`--tool-module`)

Place the tool definition and handler in a `.py` file and pass it to `faraday query`. The flag accepts a `.py` file path or a dotted module name and can be repeated.

```python
# my_tools.py
EXTRA_TOOLS = [my_tool]
EXTRA_TOOL_HANDLERS = {"my_tool": my_tool_fn}
```

```bash
faraday query "..." --tool-module ./my_tools.py
faraday query "..." --tool-module my_package.tools
faraday query "..." --tool-module tools.scoring --tool-module tools.retrieval
```

File-path modules are automatically bind-mounted into the container when using `--use-docker`.

### YAML config (`tool_modules`)

To load tool modules on every run without repeating CLI flags, add `tool_modules` to `faraday.yaml`. Uses the same `EXTRA_TOOLS` / `EXTRA_TOOL_HANDLERS` module convention as the CLI.

```yaml
tool_modules:
  - my_package.tools
  - ./tools/scoring.py
```

CLI `--tool-module` flags are appended after config entries, so CLI modules take precedence for duplicate handler names.

## Advanced

### Multiple modules and override order

When multiple modules define a handler for the same tool name, the last one loaded wins. Tool definitions are deduplicated by name using the same last-wins rule.

### Overriding a built-in tool

To swap out a built-in tool's implementation, register a new handler under the built-in's name. No new tool definition is needed — the existing spec stays in the model's tool list.

```python
# Redirect general_web_search to a custom implementation for this agent only.
agent = FaradayAgent(
    extra_tool_handlers={"general_web_search": my_search_fn},
)
```

### Process-wide registration

To expose a tool to every `FaradayAgent` instance in the process, append the tool definition to `ALL_SEARCH_TOOLS` and the handler to `SEARCH_TOOL_DICT` in `faraday/search/__init__.py`. Prefer `extra_tools`, `--tool-module`, or `tool_modules` for project- or run-specific tools.
