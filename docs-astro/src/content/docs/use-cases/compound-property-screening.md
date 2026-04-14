---
title: Compound property screening
description: Use Faraday to calculate physicochemical properties for a set of drug candidates and assess drug-likeness.
sidebar:
  order: 1
---

Use this when you have a list of candidate molecules and want a fast property screen — molecular weight, logP, QED — before committing to more expensive assays or synthesis.

## What you get

- Physicochemical properties calculated via `rdkit` for each compound
- A rule-based drug-likeness assessment against Lipinski and Veber-style criteria
- A ranked comparison table across all compounds in a single run
- A short recommendation on which compounds look best for follow-up
- A reusable trajectory you can extend with additional compounds or analyses

## How it works

You give Faraday a list of compound names and a clear screening brief. The agent fetches structures from PubChem, writes Python code using `rdkit`, executes it inside a Docker sandbox, collects the output, and assembles a structured comparison. If a compound fails a heuristic check, the agent flags it and explains why.

This fits early-stage triage workflows where you want a fast, reproducible pass over a compound series before a deeper SAR or assay planning discussion.

## Setup

This example assumes:

- Faraday runs locally on your machine
- Agent-generated code runs in the Docker sandbox
- You built the provided sandbox image with `Dockerfile.sandbox`
- The sandbox can reach PubChem over the network
- `OPENAI_API_KEY` is set

Build the sandbox image once:

```bash
docker build -f Dockerfile.sandbox -t faraday-code-sandbox .
```

If you use the provided `Dockerfile.sandbox`, both `rdkit` and `pubchempy` are already included.

Minimal `faraday.yaml`:

```yaml
launch:
  mode: host

execution:
  backend: docker
  workspace_path: /workspace

runtime:
  workspace:
    source: ./
```

Run `faraday --check-tools` to verify credentials and backend selection before running a longer analysis.

## Typical workflow

**1. Run the screen:**

```bash
faraday "Use PubChem to retrieve structures for the following compounds, then use rdkit to analyze them for early-stage medicinal chemistry triage.

For each compound, calculate:
- molecular weight
- logP
- QED
- TPSA
- H-bond donors
- H-bond acceptors
- rotatable bonds

Then:
- assess each compound against Lipinski's Rule of Five
- note any obvious permeability or flexibility concerns using TPSA and rotatable bonds
- rank the compounds from most promising to least promising for oral drug-like follow-up
- explain the ranking in plain English
- include the PubChem CID used for each compound
- save a CSV called compound_screen.csv with the calculated properties
- present a final summary table in the answer

Compounds:
- warfarin
- imatinib
- aspirin
- sotorasib"
```

Faraday fetches the structures, writes and executes a Python script using `rdkit.Chem` and `rdkit.Chem.Descriptors` inside the Docker sandbox, saves the CSV into `agent_outputs/`, and returns a ranked summary table.

**2. Extend with additional compounds using the trajectory as context:**

```bash
faraday --previous-context run_outputs/run_<timestamp>_<chat_id>_<run_id>/run_artifacts/trajectory.json \
  "Add gefitinib and erlotinib to the existing screen, recompute the ranking, and highlight which compounds look most balanced for oral follow-up."
```

**3. Request a plot:**

```bash
faraday --previous-context run_outputs/run_<timestamp>_<chat_id>_<run_id>/run_artifacts/trajectory.json \
  "Create a scatter plot of MW vs logP for all compounds, label each point, color by QED, and save it as compound_screen.png."
```

The figure lands in `agent_outputs/` inside the run directory.

## What the agent does step by step

1. Looks up each compound in PubChem and retrieves a structure identifier
2. Converts the PubChem structure into an `rdkit` molecule
3. Calculates MW, logP, QED, TPSA, HBD, HBA, and rotatable bonds
4. Checks each compound against Lipinski's Rule of Five and simple permeability or flexibility heuristics
5. Writes a CSV or figure into `agent_outputs/` when requested
6. Formats results into a summary table and ranks the compounds for follow-up

## When to use this

- Early hit triage before purchasing or synthesizing compounds
- Comparing a series of analogs against drug-likeness criteria
- Generating a property baseline for a SAR analysis
- Producing a reproducible screening record with a full calculation trace

## Related pages

- **[Results and artifacts](../run-queries/results-and-artifacts/)** for the output directory layout
- **[Run a single query](../run-queries/run-a-single-query/)** for interactive mode and useful CLI flags
- **[Docker integration](../integrations/docker/)** for sandbox setup and image configuration
