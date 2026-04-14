---
title: Faraday-OSS tools
description: Tools that are available to all Faraday-OSS agents.
sidebar:
  order: 2
---

Faraday ships with **web search** tools (require `EXA_API_KEY`) and **code execution** tools (always available). Run `faraday --check-tools` to see what's active.

## Web search tools

| Tool | What it does | Input(s) |
|------|-------------|---------|
| `general_web_search` | Broad web search with highlights and per-result summaries. | `query` |
| `general_web_search_question_answering` | Returns a single synthesised answer via Exa's chat-completion endpoint. | `query` |
| `scientific_literature_search` | Searches research papers and preprints; returns titles, URLs, and AI summaries. | `literature_query` |
| `pharma_web_search` | Scoped to pharma/news/regulatory domains; results are LLM meta-summarised. | `query` |
| `read_webpage` | Fetches URL content, highlighting passages relevant to the query. | `user_query`, `url_list` |
| `name_to_smiles` | Looks up the canonical SMILES string for a molecule name or identifier. | `query` |

## Code execution tools

Both tools share a **persistent sandbox session** — variables, imports, and files created in one call carry into subsequent calls.

| Tool | What it does | Input |
|------|-------------|-------|
| `execute_python_code` | Runs Python; captures stdout/stderr | `python_code` |
| `execute_bash_code` | Runs shell commands; useful for file ops and CLI tools. | `bash_code` |

To add your own tools, see **[Adding a custom tool](./custom-tools/)**.
