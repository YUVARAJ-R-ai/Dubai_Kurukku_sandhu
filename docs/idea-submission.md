# Route Resilience
### Occlusion-Robust Road Extraction & Graph-Theoretic Criticality Analysis for Urban Mobility

**Team:** Dubai_Kurukku_sandhu · **Track:** ISRO / NNRMS — Satellite EO for Urban Mobility

> **One line:** We turn cloud-, canopy- and shadow-blinded satellite imagery into a *connected, routable* road graph, then mathematically pinpoint the intersections whose failure would break the city — and simulate the fallout.

---

## 1. Problem

Satellite-based road extraction in dense Indian metropolises (e.g. Bengaluru) suffers from two compounding failures:

- **Spectral blindness** — tree canopy, building shadows, and cloud cover hide road pixels, so deep-learning masks come out **fragmented**.
- **Broken topology** — a fragmented mask is not a network. It has gaps, dead-ends, and disconnected components, making it **useless for routing, traffic simulation, or disaster response** — exactly the applications that need it.

This gap is squarely within **ISRO's NNRMS** mandate to maximise the downstream value of indigenous EO satellites (Cartosat, Resourcesat LISS-IV), and has been flagged in national meets by **MeitY** (GIS-based urban planning, e-governance) and the **Ministry of Consumer Affairs** (infrastructure mapping, facility layout, route verification).

**The core insight:** extracting road *pixels* is a solved-enough problem; extracting a road *network you can compute on* is not. We attack the second problem.

---

## 2. Proposed Solution

An end-to-end pipeline in **three stages**:

```
Satellite imagery
      │
      ▼
① OCCLUSION-ROBUST SEGMENTATION   →  road mask (sees through canopy/shadow)
      │
      ▼
② TOPOLOGICAL HEALING              →  connected, weighted, routable graph
      │
      ▼
③ CRITICALITY + STRESS TESTING     →  bottlenecks, Resilience Index, what-if sim
```

**① Occlusion-robust segmentation.** A context-aware deep-learning model (U-Net/DeepLabV3+ → SegFormer) trained with **synthetic occlusions** (shadows, canopy, vehicles) and a **topology-preserving clDice loss** so it infers road *continuity* under obstructions instead of dropping the pixels.

**② Topological healing.** Convert the mask to a graph (skeletonize + `sknw`, then **RDP simplification + node-merging** to collapse the spur edges and blobby-intersection micro-nodes that raw skeletonization produces), then bridge occlusion gaps with a **cycle-preserving, distance- and angle-gated reconnection**: Union-Find tracks road fragments while a nearest-neighbour endpoint bridge — unlike a pure MST — can also close gaps *inside* grid loops, not just reconnect fully isolated islands. Output: a single connected, weighted graph exported as GeoJSON/GraphML. Edges carry geometric length; travel-time weights use **OSM road-class speeds where a road already exists, and a default residential speed for newly-discovered, canopy-hidden edges** (the binary mask itself doesn't classify road type).

**③ Criticality & stress testing.** **Betweenness centrality** surfaces "Gatekeeper Nodes" — intersections that sit on the most shortest paths. **Node-ablation** then removes them one by one to simulate floods/accidents/closures, and we compute a **Resilience Index** quantifying how far network efficiency degrades.

**④ Conversational decision layer (agentic).** A planner shouldn't need graph theory to use this. A **LangGraph** agent — with the analysis functions exposed as **LangChain** tools and **Claude** (`langchain-anthropic`) doing the reasoning — turns a plain-English question (*"if Silk Board floods, which areas are cut off and by how much does travel time rise?"*) into the right sequence of tool calls (ablation → routing → resilience) and a clear, cited answer on the map. The same LangGraph state machine can orchestrate the pipeline itself, handling bad tiles or failed stages gracefully.

*(See the architecture / data-flow diagram — issue #2 — for the full stage-by-stage view.)*

---

## 3. Novelty & Differentiators

| # | What most solutions do | What we do |
|---|------------------------|-----------|
| 1 | Optimise pixel accuracy (IoU) | Optimise **connectivity** via **clDice** topology-preserving loss (clDice paper reports ~73% fewer fragments vs a Dice baseline) |
| 2 | Ship a broken raster mask | **Heal** it into a routable graph with a **cycle-preserving, distance/angle-gated reconnection** (Union-Find fragments + endpoint bridging that closes intra-loop gaps, not a pure MST) |
| 3 | Report where roads are | Report **which roads matter** — betweenness criticality + a quantitative **Resilience Index** |
| 4 | Static output | **Interactive what-if simulation** — disable a node, see live rerouting + travel-time increase |
| 5 | Expert-only dashboards | **Agentic natural-language interface** — a LangGraph + LangChain assistant (Claude) lets non-technical planners *ask* for a resilience analysis in plain English |

**Honest rigor:** betweenness alone is a known-imperfect resilience proxy, so we treat it as a *shortlist* and rank real damage by **largest-connected-component drop and global network efficiency**. The latter is our **primary Resilience Index** — it degrades gracefully even when a closure disconnects the graph, whereas the average-path-length ratio blows up to ∞ there. Centrality is **precomputed once**, and the live click-to-disable demo uses **approximate (k-sampled) betweenness on a demo-sized AOI**, so the dashboard stays responsive instead of recomputing exact O(V·E) centrality per click.

---

## 4. Impact

- **Disaster response** — pre-identify which intersections, if flooded or blocked, isolate a sector; plan evacuation reroutes before the event.
- **Urban planning** — an automated, occlusion-robust road asset map plus a heatmap of the city's "weakest links" for infrastructure investment.
- **Facility distribution** (Consumer Affairs use case) — verify routes and site facilities against a network that reflects *real* reachability, not a broken mask.
- **Downstream reuse** — the exported weighted graph (GeoJSON/GraphML) feeds any routing/traffic tool.

---

## 5. Feasibility (30-hour build plan)

**Data — automated, drift-aware labeling.**
- Imagery: we **standardize the model on one high-resolution band** — **Cartosat-3** (~sub-metre, provided at the event) with **pan-sharpened Resourcesat LISS-IV** (5.8 m) as backup — rather than mixing 10 m and sub-metre in a single network (a 2-lane road is sub-pixel at 10 m but a wide polygon at sub-metre, and one 30h model can't do both well). **Sentinel-2** and the open sets (**DeepGlobe / SpaceNet / OpenSatMap**) are used for **pre-training** only.
- Ground truth: **OpenStreetMap** vectors auto-rasterized into masks, **buffered 3–5 px and scored with a relaxed IoU** to absorb the 5–15 m orthorectification drift between OSM and imagery — automated, but not naïvely "clean".

**Compute.** Graph analysis + dashboard run on CPU. Only segmentation needs GPU — and we **fine-tune pretrained backbones**, not train from scratch, so it fits the 30-hour window.

**Parallel two-team workflow** (the key to finishing in 30h):
- **Team ML+Data** builds the segmentation model + data pipeline.
- **Team Graph+UI** builds healing, analysis, and the dashboard *immediately* using mock / open-source vector baselines — no waiting on the model.
- The two meet at a **frozen mask→graph GeoJSON contract (hour 1)**.

**Stack:** PyTorch · segmentation-models-pytorch · Rasterio/GDAL · Albumentations · scikit-image + `sknw` · NetworkX + OSMnx · Streamlit + folium/Leaflet · **LangGraph + LangChain + `langchain-anthropic` (Claude)** for the conversational decision layer.

---

## 6. Expected Outcomes

1. **High-fidelity routable topology** — a mathematically connected vector network, far beyond pixel segmentation, generalizing across urban / suburban-forested / rural terrain.
2. **Quantitative criticality map** — a spatial heatmap of high-betweenness "Gatekeeper Nodes" acting as single points of failure.
3. **Predictive impact assessment** — a **Resilience Index** and an interactive dashboard where a planner disables a node and instantly sees rerouting and travel-time increase.
4. **Conversational decision support** — a LangGraph/LangChain assistant that answers plain-English resilience questions by orchestrating the analysis tools, making the whole system usable by non-technical planners.

**Evaluation metrics:** IoU & Dice with **occlusion-recall** · **Connectivity Ratio** (LCC gain after healing) · **APLS** topological accuracy vs OSM · relaxed IoU (3–5 px tolerance) · cross-terrain generalisation.

---

*Full research brief, competitive landscape, task breakdown and risk log: [docs/research.md](research.md).*
