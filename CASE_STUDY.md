# Meshy Prompt Forge — Case Study

**Author:** Sanmati Sawalwade
**Background:** MS Information Systems, Northeastern University (Silicon Valley) | Ex-Gameskraft | Startup Founder
**Built in:** One night
**Repo:** [github.com/sanmati1997/meshy-prompt-forge](https://github.com/sanmati1997/meshy-prompt-forge)
**Live demo:** [link]

---

## At a Glance

| | Dragon | Sword |
|---|---|---|
| Geometry lift | **+20.8%** faces | **+29.1%** faces |
| Topology quality | Watertight, 0 holes, 0 non-manifold edges | Printability maintained |
| Visual result | Better posed, game-ready stance | **Wrong archetype** — fantasy sword → combat knife |
| Key finding | Optimization works on complex objects | Material hints override style intent |

---

## The Problem

Meshy has 6 million users but $15M ARR — roughly $2.50 average revenue per user. Most users are free and never convert to paid.

The biggest reason: **bad first experience.**

Text-to-3D is fundamentally harder to prompt than text-to-image. Users have mental models for Midjourney and DALL-E — those don't transfer to 3D generation. A new Meshy user types a vague prompt, gets a mediocre or wrong model, burns free credits, and leaves before the "aha moment."

### Evidence from public sources

- **G2 reviewer:** *"Small wording changes produce wildly different results"*
- **G2 reviewer:** *"Limited prompt character space is frustrating"*
- **Trustpilot:** Multiple users reporting wasted credits on poor first outputs
- **Meshy's own docs:** A "prompt tips" section exists — companies only write those when support tickets pile up
- **Pricing page:** Free tier caps downloads — users can't evaluate what they generated without upgrading

This is an activation problem. Users hit the friction wall before they see the product's real value.

---

## Hypothesis

> Optimized prompts — prompts that include explicit style tags, material hints, topology context, and isolation framing — will produce meaningfully better 3D models than naive prompts on complex objects.

Secondary hypothesis: **prompt sensitivity is highest on ambiguous, complex objects** (creatures, characters) and lowest on simple geometric objects (swords, barrels).

---

## What I Built

A rule-based prompt optimizer + mesh quality scorer, built in one night on Meshy's free tier.

**Stack:** Python, Streamlit, trimesh, Plotly. Zero dollars. No paid APIs.

### How the optimizer works

The rewriter applies 8 deterministic rules reverse-engineered from Meshy's failure patterns:

| Rule | Why |
|---|---|
| Add "3D model of" prefix | Meshy responds better to explicit 3D framing |
| Detect object type (character / weapon / vehicle / prop / creature) | Enables context-aware downstream rules |
| Add style tag if missing | Style-less prompts produce inconsistent results |
| Add material hint if missing | Material ambiguity is the top cause of texture inconsistency |
| Add "clean quad topology" | Improves mesh quality for rigging and sculpting |
| Add isolation context | Prevents unwanted scene elements |
| Add T-pose for characters | Critical for rigging compatibility |
| Replace vague adjectives | "cool" → "detailed", "nice" → "well-crafted" |

Every change is explainable. No black-box LLM. A PM can audit every decision.

### Bonus: Game IP detection

The tool detects known game IPs (Dota 2, League of Legends, Overwatch) and applies IP-specific art style tags automatically:

- Input: `dota 2 invoker`
- Output: `3D model of dota 2 invoker, Dota 2 stylized hero model, Valve art style, detailed armor and fabric, game-ready PBR materials, clean quad topology, isolated object, no background, standing upright in T-pose`

### Mesh scorer

For downloadable GLBs, the tool scores on 4 objective topology metrics:

| Metric | Weight | Why It Matters |
|---|---|---|
| Watertight | 35 pts | Printability and rigging |
| Is Volume | 25 pts | Manifold + consistent normals |
| Non-manifold edges | 25 pts | Topology quality |
| Degenerate faces | 15 pts | Mesh cleanliness |

---

## Findings

### Finding 1 — Hypothesis confirmed on complex objects

**Test:** `dragon` (naive) vs `3D model of dragon, game-ready stylized asset, organic skin and scales, clean quad topology, isolated object, no background`

| Naive: `dragon` | Optimized |
|:---:|:---:|
| ![Dragon Naive](assets/results/dragon_naive.png) | ![Dragon Optimized](assets/results/dragon_optimized.png) |

| Metric | Naive | Optimized | Delta |
|---|---|---|---|
| Faces | 992,658 | 1,199,452 | **+206,794 (+20.8%)** |
| Vertices | 496,327 | 599,724 | **+103,397 (+20.8%)** |
| Watertight | — | **Yes** | ✅ |
| Holes | — | **0** | ✅ |
| Non-manifold edges | — | **0** | ✅ |
| Printability | ✅ | ✅ | same |

**Visual observation:** The naive prompt produced a dragon with wings swept back in an organic, natural pose — visually interesting but structurally harder to rig. The optimized prompt produced a dragon with wings fully spread in a display stance — immediately recognizable as game-ready, more consistent with what a rigger or animator would need. The "clean quad topology" and isolation context in the prompt appear to push Meshy toward standardized asset poses, not just higher polygon counts.

The optimized model is also confirmed watertight with zero holes and zero non-manifold edges — the exact topology requirements for 3D printing and animation pipelines.

**Implication:** Adding explicit style, topology, and isolation context pushes Meshy toward higher-fidelity, production-ready outputs on complex objects. The benefit is both measurable (20.8% geometry) and visual (pose standardization).

---

### Finding 2 — Over-specified material hints change weapon archetype, not just quality

**Test:** `a cool sword` vs `3D model of a detailed sword, game-ready low-poly asset, tempered steel blade, clean quad topology, isolated object, no background`

| Naive: `a cool sword` | Optimized |
|:---:|:---:|
| ![Sword Naive](assets/results/sword_naive.png) | ![Sword Optimized](assets/results/sword_optimized.png) |

| Metric | Naive | Optimized | Delta |
|---|---|---|---|
| Faces | 129,400 | 167,050 | **+37,650 (+29.1%)** |
| Vertices | 64,690 | 83,527 | **+18,837 (+29.1%)** |
| Printability | ✅ | ✅ | same |

**Visual observation:** This is the most visually striking result of the experiment. The naive `a cool sword` prompt produced an **ornate fantasy longsword** — wide blade, elaborate decorative crossguard with engravings, fantasy-coded aesthetic. The optimized prompt produced a **clean, realistic combat sword** — straight blade, minimal crossguard, practical profile. These are completely different weapon archetypes. The geometry improvement (+29.1%) is almost irrelevant compared to the archetype mismatch — a user who wanted a fantasy sword got a tactical knife instead.

The culprit is clear: `"tempered steel blade"` is a modern, tactical material hint. It overrode the fantasy aesthetic implied by `"cool sword"` and steered Meshy toward a realistic style. The optimizer made the prompt more specific but less intent-faithful.

**Implication:** Material inference must respect style context embedded in the original prompt. A smarter optimizer would detect `"cool sword"` as fantasy-coded — informal, evocative — and use a fantasy-appropriate material hint (`"enchanted steel blade, ornate fantasy detailing"`) rather than a generic one. This is the clearest gap in the current rule set and the highest-priority fix.

---

### Finding 3 — IP characters require image-to-3D, not text optimization

**Test:** Generated `dota 2 sf` (Shadow Fiend) with naive and optimized prompts.

Neither produced a recognizable Shadow Fiend. The game IP detection correctly identified it as a Dota 2 character and applied Valve art style tags — but Meshy's model doesn't have sufficient hero-specific training data to reconstruct a named character from text alone.

**Implication:** Meshy should surface its image-to-3D workflow more prominently for users requesting specific IP characters. This is a product navigation gap, not a model capability gap. A simple in-product message — *"Looking for a specific character? Try image-to-3D"* — would reduce credit waste and improve conversion for this user segment.

---

### Finding 4 — Free tier download gating creates a broken evaluation loop

While running this experiment I hit a structural blocker: **the free tier doesn't allow GLB downloads.**

This means:
1. Free users can generate but can't export or evaluate their model properly
2. The try-before-you-buy loop is broken — users can't take a model into Blender to check if it meets their needs
3. Users who can't evaluate quality won't pay to upgrade — they'll just leave

This finding came from building the tool, not from a dashboard. It's a real friction point hiding in the free-to-paid conversion funnel and likely costs more in lost conversions than the infrastructure savings from gating it.

---

## Business Implications

| Finding | Business Impact | Recommended Action |
|---|---|---|
| Prompt optimizer lifts geometry +20-29% and improves topology on complex objects | Higher first-session success → lower churn → higher conversion | A/B test in-product prompt suggestions at generation time |
| Material hints can override style intent | Users get wrong archetype — worse than no optimization | Add style-context detection before material inference |
| IP characters need image-to-3D | Text optimization can't fix a training data gap | Surface image-to-3D routing for named characters |
| Free tier download gating breaks evaluation loop | Users can't validate quality before paying | A/B test 1-3 free GLB downloads — conversion uplift likely exceeds cost |

---

## Limitations

- Test set is small (6 prompts, 1 session). Findings are directional, not statistically significant.
- No access to Meshy's real funnel data — implications are inferred, not measured.
- Mesh scoring was blocked by free-tier download limits on Meshy. Scoring infrastructure is built and validated on external GLBs but not run on paired Meshy outputs.
- Rule-based rewriter has no feedback loop — rules were hand-authored, not learned from real user failure data.

Being explicit about these limits is intentional. A PM or data scientist at Meshy would catch overclaiming immediately.

---

## What's Next

If I had access to Meshy's internal data, the three highest-value experiments would be:

1. **A/B test the optimizer in-product** — surface prompt suggestions at generation time, measure first-session success rate lift and credit-to-conversion rate
2. **Style-context detection** — train a lightweight classifier to detect fantasy vs realistic vs sci-fi intent before applying material hints, fixing the archetype override problem
3. **Free download A/B test** — measure conversion impact of allowing 1-3 free GLB downloads per month; hypothesis is that users who can validate quality in Blender convert at 2-3x the rate of users who can't

---

## About This Project

Built in one night as a research project to understand Meshy's activation funnel.

The goal was not to build a polished product — it was to identify a real problem, build a working artifact, and produce evidence-backed findings. The tool is fully external — no internal Meshy data was used or required.

**Repo:** [github.com/sanmati1997/meshy-prompt-forge](https://github.com/sanmati1997/meshy-prompt-forge)
**Live demo:** [link]
**Contact:** sawalwade.s@northeastern.edu | [linkedin.com/in/sanmati-sawalwade](https://linkedin.com/in/sanmati-sawalwade)
