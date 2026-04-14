---
title: Quickstart
description: Run your first Faraday task in a few minutes.
---

## Before you start

- Install Faraday: **[Install](./install/)**
- Set a model API key such as `OPENAI_API_KEY`

## Single Query Mode

```bash
faraday "Summarize structure–activity relationships across KRAS inhibitors, focusing on motifs that improve binding affinity and selectivity"
```

Faraday will run the task while logging updates to the console for real-time feedback.


### Accessing the results

While the solution will be printed to the console, its often preferable to have a persistant record of the task and it's execution.

By default, Faraday will store the results to the `./run_outputs` directory.

```bash
# in the ./run_outputs directory
 run_20260406045042_chat_20260406045041_b524f0c9_query_20260406045041_304a95d8
│   ├── agent_outputs
│   │   ├── beta_lactam_5_molecules.png
│   │   └── plots
│   │       ├── beta_lactam_molecules_grid.png
│   │       ├── binomial_vs_normal.png
│   │       ├── binomial_vs_normal_data.csv
│   │       ├── binomial_vs_normal_data.txt
│   │       ├── binomial_vs_normal_n50_p0.5.svg
│   │       └── normal_vs_binomial_n50_p0.5.png
│   └── run_artifacts
│       ├── events.jsonl
│       ├── metadata.json
│       ├── result.json
│       └── trajectory.json
```

See more information about the results directory in the **[Results and artifacts](./reference/results-and-artifacts/)** section.

## Config-based Run

The Faraday agent can be configured using a YAML file. An example config is provided in `faraday.example.yaml`.
To run with a config, use the `--config` flag:

```bash
faraday --config faraday.example.yaml "Summarize structure–activity relationships across KRAS inhibitors, focusing on motifs that improve binding affinity and selectivity"
```

For practical examples, see **[YAML guidance](./reference/yaml-guidance/)**.


## Interactive Mode

```bash
faraday
```