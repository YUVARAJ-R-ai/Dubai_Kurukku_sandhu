# Route Resilience
### Occlusion-Robust Road Extraction & Graph-Theoretic Criticality Analysis for Urban Mobility

> An end-to-end pipeline that "sees through" tree canopy, shadows and cloud cover to extract roads from satellite imagery, heals the fragmented mask into a **routable weighted graph**, then runs **criticality + stress-test analysis** to find a city's bottlenecks and simulate disaster scenarios.
>
> _ISRO / NNRMS hackathon — Team Dubai_Kurukku_sandhu_

---

## Where we are right now

**Immediate goal → IDEA SUBMISSION (due tomorrow).**
The 30-hour build hackathon only happens **if we get shortlisted** in this first round. So **everything this round is about the pitch**, not code: proposal doc, architecture diagram, deck, and a tight feasibility/metrics story.

The build tasks (Phases I–IV) are already on the board as backlog — we touch those **only after we're shortlisted**.

👉 **Round 1 priority issues:** [#1](https://github.com/YUVARAJ-R-ai/Dubai_Kurukku_sandhu/issues/1) proposal · [#2](https://github.com/YUVARAJ-R-ai/Dubai_Kurukku_sandhu/issues/2) architecture diagram · [#3](https://github.com/YUVARAJ-R-ai/Dubai_Kurukku_sandhu/issues/3) pitch deck · [#4](https://github.com/YUVARAJ-R-ai/Dubai_Kurukku_sandhu/issues/4) metrics & feasibility

---

## The idea in one minute

Standard satellite road extraction fails due to **"spectral blindness"** — canopy, shadows, clouds — producing broken masks that can't be used for routing or disaster response. We fix this in three moves:

1. **Occlusion-robust segmentation** — a context-aware deep-learning model (U-Net/DeepLabV3+ → SegFormer) trained with synthetic occlusions and a **topology-preserving clDice loss** to infer road continuity through obstructions.
2. **Topological healing** — convert the mask to a graph (skeletonize + `sknw`), then bridge occlusion gaps with **union-find + MST**, gated by distance and angular alignment, into one connected routable network.
3. **Criticality & stress testing** — **betweenness centrality** finds "Gatekeeper Nodes"; **node ablation** simulates floods/accidents and produces a **Resilience Index** that quantifies how badly the network degrades.

On top of this sits a **conversational decision layer** — a **LangGraph + LangChain** agent (powered by Claude) that lets non-technical planners ask resilience questions in plain English and have the analysis tools run for them.

Full detail, competitive landscape, and risks are in **[docs/research.md](docs/research.md)** — read this before the pitch.

---

## Team tracks

We work in **4 parallel tracks**. Each teammate owns one track; issues are labelled accordingly. _(Issues are currently unassigned — claim yours with `/start-task`.)_

| Track | Owner (GitHub) | Scope | Label | Build milestone |
|-------|----------------|-------|-------|-----------------|
| **A — ML / Segmentation** | _@_______ | Model training, clDice loss, attention/SegFormer, occlusion handling | `track-ml` | Phase I |
| **B — Data Pipeline** | _@_______ | Tiling (Rasterio/GDAL), OSM auto-labeling, occlusion augmentation | `track-data` | Phase I |
| **C — Graph & Analysis** | _@_______ | Mask→graph, MST healing, centrality, ablation, Resilience Index | `track-graph` | Phase II + III |
| **D — Dashboard + AI Assistant** | _@_______ | Streamlit + folium map, criticality heatmap, click-to-disable sim, **LangGraph/LangChain conversational assistant** | `track-dashboard` | Phase IV |

> **Fill in your GitHub username** in the table above and self-assign your issues on the [board](https://github.com/users/YUVARAJ-R-ai/projects/9).

**Key parallelism rule (from research):** Track C can start immediately on **mock/open-source vector baselines** without waiting for the model. To keep ML and graph teams from diverging, **freeze the mask→graph GeoJSON schema on hour 1** (issue [#12](https://github.com/YUVARAJ-R-ai/Dubai_Kurukku_sandhu/issues/12)).

---

## How to proceed (workflow)

We use a GitHub Project board with two custom skills baked into this repo (`.claude/skills/`). They work in Claude Code automatically once you've cloned and opened the repo.

### 1. Pick up work — `/start-task`
In Claude Code, run:
```
/start-task
```
It pulls your assigned Backlog issue, refines it into a user story, moves it to **Ready**, creates a `feat/<issue#>-...` branch, helps you implement it, and raises a PR. Move issue → **In Review** when the PR is up.

### 2. Log new work — `/new-issue`
Found a bug or a task that isn't on the board?
```
/new-issue
```
It writes a structured issue (with acceptance criteria) and drops it into **Backlog** on the board.

### 3. Board columns
`Backlog → Ready → In Progress → In Review → Done`
Keep your issue's **Status** current as you move through it.

### Manual fallback (no Claude Code)
```bash
gh issue list --repo YUVARAJ-R-ai/Dubai_Kurukku_sandhu        # see the backlog
gh issue develop <issue#> --repo YUVARAJ-R-ai/Dubai_Kurukku_sandhu -c   # branch + checkout
# ...work, commit...
gh pr create --fill                                          # raise PR
```

---

## Roadmap

| Milestone | What | When |
|-----------|------|------|
| **Round 1 — Idea Submission** | Proposal, architecture diagram, deck, metrics | **Tomorrow** (gate) |
| Phase I — Segmentation | Data pipeline + occlusion-robust model | If shortlisted |
| Phase II — Graph Healing | Mask→graph, MST/union-find healing, export | If shortlisted |
| Phase III — Analysis & Stress Test | Centrality, ablation, Resilience Index, APLS | If shortlisted |
| Phase IV — Dashboard + AI Assistant | Streamlit map, heatmap, click-to-disable sim, LangGraph/LangChain assistant | If shortlisted |

---

## Tech stack (planned)

| Layer | Choice |
|-------|--------|
| Segmentation | `segmentation-models-pytorch` U-Net + ResNet34 → SegFormer |
| Loss | Dice + BCE + **soft-clDice** (topology-preserving) |
| Geo I/O | Rasterio, GDAL, Albumentations |
| Mask→graph | scikit-image `skeletonize` + `sknw` |
| Healing | NetworkX MST + union-find, KD-tree |
| Analysis | NetworkX (betweenness, efficiency) + OSMnx (OSM ground truth) |
| Topology metric | APLS (CosmiQ) |
| Dashboard | Streamlit + streamlit-folium (Leaflet) |
| Agent / NL layer | LangGraph + LangChain + `langchain-anthropic` → Claude (`claude-sonnet-5`) |

---

## Repo layout

```
.
├── README.md            ← you are here
├── CLAUDE.md            ← project instructions + skill registration
├── docs/
│   └── research.md      ← full research brief (read before pitching)
└── .claude/
    └── skills/
        ├── new-issue/   ← /new-issue
        └── start-task/  ← /start-task
```

## Links
- 📋 Project board → https://github.com/users/YUVARAJ-R-ai/projects/9
- 📄 Research brief → [docs/research.md](docs/research.md)
- 📦 Repo → https://github.com/YUVARAJ-R-ai/Dubai_Kurukku_sandhu
