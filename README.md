# Meshy Prompt Forge

> Turns weak 3D prompts into production-ready ones — and proves it with mesh quality scores.

## The Problem

Meshy has 6M users but most never convert to paid. The biggest activation killer: prompt sensitivity.

A user types `"a cool sword"`, gets a mediocre mesh, and leaves before the "aha moment."

Public evidence:
- G2 reviewer: *"small wording changes produce wildly different results"*
- Trustpilot: users reporting wasted credits on bad first outputs
- Meshy's own prompt tips doc exists — companies only write those when support tickets pile up

Text-to-3D is 10x harder to prompt than text-to-image. There's no established mental model for new users. This tool fills that gap.

## What It Does

1. Takes a plain-English prompt
2. Applies rules reverse-engineered from Meshy's failure patterns
3. Outputs an optimized prompt + explains every change
4. Scores both generated meshes on objective quality metrics
5. Quantifies the improvement

## Results

| Prompt | Naive Score | Optimized Score | Delta |
|---|---|---|---|
| "a cool sword" | — | — | — |
| "a warrior" | — | — | — |
| "a dragon" | — | — | — |
| "a nice vase" | — | — | — |
| "a futuristic car" | — | — | — |
| "a wooden barrel" | — | — | — |
| **Average** | | | |

*Results will be filled in after running the experiment.*

## Mesh Scoring Methodology

Each generated GLB is scored on 5 objective metrics from 3D printing literature:

| Metric | Weight | Why It Matters |
|---|---|---|
| Watertight | 35 pts | Printability, rigging |
| Is Volume | 25 pts | Manifold + consistent normals |
| Non-manifold edges | 25 pts | Topology quality |
| Degenerate faces | 15 pts | Mesh cleanliness |

Composite score: 0–100.

## Rewriter Rules

Rules are deterministic — every change is explainable:

1. Add `"3D model of"` prefix
2. Detect object type (character / weapon / vehicle / prop / creature)
3. Infer and add style tag if missing
4. Infer and add material hint if missing
5. Add `"clean quad topology"` hint
6. Add isolation context (`"isolated object, no background"`)
7. Add T-pose orientation for characters
8. Replace vague adjectives (`"cool"` → `"detailed"`)

## Setup

```bash
git clone https://github.com/YOUR_USERNAME/meshy-prompt-forge
cd meshy-prompt-forge
pip install -r requirements.txt
```

**Run the web app:**
```bash
streamlit run app.py
```

**Run from CLI:**
```bash
# Rewrite a prompt
python run.py rewrite "a cool sword"

# Score a mesh
python run.py score model.glb

# Compare naive vs optimized
python run.py compare naive.glb optimized.glb

# Full guided experiment
python run.py batch
```

## Stack

- Python, trimesh, numpy — mesh scoring
- Streamlit, Plotly — web UI
- Rule-based rewriter — no LLM, every change is transparent and explainable
- Meshy free tier — 3D generation

## Context

Built in one night as a research project to understand Meshy's activation funnel.

The claim: optimized prompts consistently produce higher-quality meshes than naive prompts, as measured by objective topology metrics.

The tool is fully external — no internal Meshy data required.

---

Built by [Sanmati Walwade](https://linkedin.com/in/sanmatiwalwade) — MS Information Systems, Northeastern University
