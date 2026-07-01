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

**② Topological healing.** Convert the mask to a graph (skeletonize + `sknw`), then bridge occlusion gaps with a **Union-Find + Minimum-Spanning-Tree** algorithm gated by Euclidean distance *and* angular alignment — so healed roads follow a natural trajectory rather than hallucinating shortcuts. Output: a single connected, length/road-class-weighted graph exported as GeoJSON/GraphML.

**③ Criticality & stress testing.** **Betweenness centrality** surfaces "Gatekeeper Nodes" — intersections that sit on the most shortest paths. **Node-ablation** then removes them one by one to simulate floods/accidents/closures, and we compute a **Resilience Index** quantifying how far network efficiency degrades.

*(See the architecture / data-flow diagram — issue #2 — for the full stage-by-stage view.)*

---

## 3. Novelty & Differentiators

| # | What most solutions do | What we do |
|---|------------------------|-----------|
| 1 | Optimise pixel accuracy (IoU) | Optimise **connectivity** via **clDice** topology-preserving loss (~73% fewer fragments vs Dice baseline) |
| 2 | Ship a broken raster mask | **Heal** it into a routable graph with **MST + Union-Find**, distance-and-angle gated |
| 3 | Report where roads are | Report **which roads matter** — betweenness criticality + a quantitative **Resilience Index** |
| 4 | Static output | **Interactive what-if simulation** — disable a node, see live rerouting + travel-time increase |

**Honest rigor:** betweenness alone is a known-imperfect resilience proxy, so our Resilience Index reports **largest-connected-component drop and global efficiency alongside it** — giving planners a defensible, multi-metric vulnerability score.

---

## 4. Impact

- **Disaster response** — pre-identify which intersections, if flooded or blocked, isolate a sector; plan evacuation reroutes before the event.
- **Urban planning** — an automated, occlusion-robust road asset map plus a heatmap of the city's "weakest links" for infrastructure investment.
- **Facility distribution** (Consumer Affairs use case) — verify routes and site facilities against a network that reflects *real* reachability, not a broken mask.
- **Downstream reuse** — the exported weighted graph (GeoJSON/GraphML) feeds any routing/traffic tool.

---

## 5. Feasibility (30-hour build plan)

**Data — zero manual labeling.**
- Imagery: **Sentinel-2** (10 m) + **Resourcesat LISS-IV** (5.8 m), both open; **Cartosat-3** provided at the event.
- Ground truth: **OpenStreetMap** vectors auto-rasterized into road masks; pre-train on **DeepGlobe / SpaceNet / OpenSatMap**.

**Compute.** Graph analysis + dashboard run on CPU. Only segmentation needs GPU — and we **fine-tune pretrained backbones**, not train from scratch, so it fits the 30-hour window.

**Parallel two-team workflow** (the key to finishing in 30h):
- **Team ML+Data** builds the segmentation model + data pipeline.
- **Team Graph+UI** builds healing, analysis, and the dashboard *immediately* using mock / open-source vector baselines — no waiting on the model.
- The two meet at a **frozen mask→graph GeoJSON contract (hour 1)**.

**Stack:** PyTorch · segmentation-models-pytorch · Rasterio/GDAL · Albumentations · scikit-image + `sknw` · NetworkX + OSMnx · Streamlit + folium/Leaflet.

---

## 6. Expected Outcomes

1. **High-fidelity routable topology** — a mathematically connected vector network, far beyond pixel segmentation, generalizing across urban / suburban-forested / rural terrain.
2. **Quantitative criticality map** — a spatial heatmap of high-betweenness "Gatekeeper Nodes" acting as single points of failure.
3. **Predictive impact assessment** — a **Resilience Index** and an interactive dashboard where a planner disables a node and instantly sees rerouting and travel-time increase.

**Evaluation metrics:** IoU & Dice with **occlusion-recall** · **Connectivity Ratio** (LCC gain after healing) · **APLS** topological accuracy vs OSM · relaxed IoU (3–5 px tolerance) · cross-terrain generalisation.

---

*Full research brief, competitive landscape, task breakdown and risk log: [docs/research.md](research.md).*
