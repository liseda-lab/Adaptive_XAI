# Adaptive-XAI

**Agentic Personas for Adaptive Scientific Explanations with Knowledge Graphs**  

> ### For new experiments, use the PyTorch implementation
> A maintained PyTorch implementation lives at **[liseda-lab/REx_PyTorch](https://github.com/liseda-lab/REx_PyTorch)**. It is significantly faster, supports both OpenAI and local LLMs (e.g. Qwen3.5-9B for training, Qwen3-4B for scoring), produces a clean per-pair JSON of correct paths with lowest common ancestors, and adds a **decoupled rerank pipeline** (`--no_llm_rerank` + `--external_rerank`) for slow datasets like oregano DTI where per-batch LLM calls during test would otherwise dominate runtime.
>
> **This TensorFlow repository is preserved for paper's reproducibility. New work should use the PyTorch implementation.**

## Overview
This repository accompanies our paper, which presents **adaptive explainability**: an approach to designing AI explanations that adapt to experts’ epistemic stances. 

Most explainable AI (XAI) systems assume a generic user model, overlooking that explanation needs vary by background, task, and interpretive strategy. Our work presents:  

1. **Agentic personas** – synthetic profiles derived from expert input and large language models, capturing diverse explanatory needs without requiring individual-level data.  
2. **A reinforcement learning framework** – using a persona-aligned reward function to generate explanations that better match expert reasoning.

We validate our approach through formative studies and a comparative user evaluation in the biomedical domain, showing that adaptive explanations are preferred and align more closely with expert assessments than uniform alternatives.

---

## Expert Evaluation 
Before the comparative evaluation, we conducted an expert evaluation to understand expert needs and build our personas:

   - **Tasks/Datasets**: DR and DTI tasks, with domain experts providing qualitative feedback on explanations.  
   - **Focus**: Clustering interpretive strategies into *agentic personas* that capture distinct explanatory stances.  

Insights from this study guided the synthesis of agentic personas and the design of adaptive explanations.

---

## Designing Agentic Personas and Adaptive Explanations

### Persona Synthesis
We synthesized coherent epistemic stances by combining clustered expert feedback with LLM-based narrative generation, producing personas that encode distinct explanatory preferences without relying on individual-level tracking.  

### Adaptive Explanation Generation
We operationalized our personas through a reinforcement learning framework: candidate explanations are filtered by relevance and scored by agentic personas, guiding the generation of explanations that adapt to diverse expert stances.

---

## Evaluation Criteria
We introduce **three evaluation criteria** grounded in philosophy of science:  
- **Relevance** – does the explanation provide meaningful, mechanistically informative content?  
- **Completeness** – does it offer sufficient causal depth without overwhelming complexity?  
- **Validity** – is the explanation biologically plausible and consistent with current knowledge?


## Repository Structure

```text
Adaptive-XAI/
│
├── evaluation/                 # Comparative evaluation and analyses
│   ├── credibility check        # Materials for the credibility check 
│   └── user study results       # Materials for the 22-participant comparative study
│
├── personas/                   # Agentic persona synthesis
│   ├── agentic_personas/       # Generate personas
│   ├── clustering/             # Clusters of interpretive strategies
│   ├── final_personas/         # Final personas (e.g., Elena, Leo)
│   └── verbalization/          # LLM-generated verbalizations 
│
├── adaptive_approach/         # Modified REx extension with adaptive explanainability
│   ├── configs/                # Training configs for RL with persona rewards
│   ├── code/                   # Core code for the adaptive approach
│   └── datasets/               # Dataset files for training and evaluation
│
└── README.md                   # ← You are here


```


---

## Prerequisites
- Docker installed on your machine

## Building the Docker Image
To build the Docker image, use the provided `Dockerfile`. Run the following command in the root directory of the project:

```sh
docker build -t reflex-image .
```
Start a container from the built image:

```sh
docker run --gpus all -d --name reflex_space -v $(pwd):/REx reflex-image tail -f /dev/null

```

Create an interactive shell in the container to run commands:

```sh
docker exec -it reflex_space bash
```

## Running Adaptive Approach
Once inside the container, to run it, you will need to run the following command:

```sh
uv run bash run.sh configs/{dataset}
```

Where `{dataset}` is the name of the dataset you would like to run the approach on.

### Skipping LLM scoring during test (for slow runs)

For slow runs — particularly **oregano DTI**, where the test set is large and per-batch GPT-4o-mini calls dominate wall time — set `no_llm_rerank=1` in the dataset config to skip in-loop LLM scoring during `test()`. The test loop then runs at neutral pace (no API calls) and `paths.json` is written with `final_score = ic_mean`. You still get all the standard metrics (Hits@k, MRR), you just don't get the persona-shaped LLM blend.

This repo does **not** ship a post-hoc reranker. If you want the full agentic pipeline (fast test + a separate batched rerank pass with a possibly lighter LLM and a data-driven failure fallback), use the PyTorch implementation referenced at the top of this README — that's where the `--external_rerank` mode lives. 

## Datasets 
Datasets should have the following files:
```
dataset
    ├── graph.txt
    ├── dev.txt
    ├── test.txt
    ├── train.txt
    └── clustered_IC_classes_edgeType.json
    └── vocab
        └── entity_vocab.json
        └── relation_vocab.json
```

Where:
- `graph.txt` contains all triples of the KG except for `dev.txt`, `test.txt`.
- `dev.txt` contains all validation triples.
- `test.txt` contains all test triples.
- `train.txt` contains all train triples.
- `clustered_IC_classes_edgeType.json` contains the IC scores for each edge types of the graph. It is a dictionary where the keys are the edge types and the values are dictionaries with the IC scores for each class.
- `vocab/entity_vocab.json` contains the vocabulary for the entities.
- `vocab/relation_vocab.json` contains the vocabulary for the relations.
- The vocab files are created by using the `create_vocab.py` file.


**Note**: The existing dataset has the graph.txt file divided into one or more files. Just run the following command in the dataset directory:

```sh
cat graph_part*.txt > graph.txt

```


## Authors
- __Susana Nunes__
- __Tiago Guerreiro__
- __Catia Pesquita__


For any comments or help needed, please send an email to: to be added.

## Acknowledgments
We build on [REx](https://github.com/liseda-lab/REx) for the reinforcement learning path discovery.  

---
