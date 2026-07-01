# Project Research: Route Resilience — Occlusion-Robust Road Extraction & Graph-Theoretic Criticality Analysis
_Generated: 2026-06-30_

## Problem & Goal
Satellite road extraction in dense Indian metropolises (e.g. Bengaluru) fails under "spectral blindness" — tree canopy, building shadows, cloud cover — producing fragmented masks that lack topological connectivity and are useless for routing, disaster response, or traffic simulation. The goal is an end-to-end pipeline that (1) uses context-aware deep learning to infer road continuity *through* occlusions, (2) heals fragmented masks into a routable weighted graph, and (3) runs graph-theoretic criticality + node-ablation analysis to surface bottlenecks and quantify network resilience. Built for a 30-hour hackathon (ISRO/NNRMS framing) with a parallel two-subteam workflow.

## Target Users
- Hackathon evaluators (ISRO/NNRMS, MeitY, Dept. of Consumer Affairs framing) — judged on segmentation accuracy, topological fidelity, and the resilience-simulation demo
- Urban / GIS planners — need occlusion-robust road asset maps and "weakest link" identification
- Disaster-response & resilience analysts — "what-if" closure simulation (flood, accident, construction)
- Downstream developers — consume the routable vector graph (GeoJSON / GraphML) for traffic/routing apps

## Competitive Landscape
| Product | Strengths | Weaknesses | Key Takeaway |
|---------|-----------|------------|--------------|
| CosmiQ/APLS + City-Scale (SpaceNet) | Defines APLS topology metric, speed/travel-time graphs, mature vectorization | Research code, not occlusion-focused, dated tooling | Reuse APLS as the topological-accuracy metric instead of inventing one |
| Microsoft/OSMnx | Battle-tested street-network download, graph analysis, centrality, plotting | Consumes OSM vectors, not imagery; no extraction | Use as OSM ground-truth loader + the centrality/ablation engine — don't rebuild graph math |
| sknw + scikit-image skeletonize | Simple, reliable mask→graph (nodes at degree≥3, edges between) | No gap-healing; brittle on broken masks | Adopt for skeletonization; layer MST/union-find healing on top |
| clDice (CVPR'21) loss | 73% fewer connected components vs Dice baseline; topology-preserving | Needs differentiable soft-skeleton; tuning cost | High-leverage: directly attacks fragmentation, the core problem |
| Mask2Former-SwinT / SegFormer | SOTA road F-score (~90% DeepGlobe), long-range context for occlusion | Heavy to train in 30h from scratch | Fine-tune a pretrained backbone; don't train transformers from zero |
| DeH4R / SAM-Road (graph-extraction nets) | End-to-end image→graph, skip vectorization | Complex, hard to reproduce in a hackathon | Note as "don't build"; segmentation+heal is the safer path |

## Feature List

### Core (MVP)
- [ ] Road segmentation model (U-Net/DeepLabV3+ with pretrained ResNet backbone) — the foundation; everything downstream depends on a mask
- [ ] OSM-auto-labeling pipeline (rasterize OSM vectors → masks aligned to tiles) — zero-manual-effort ground truth for train/val/eval
- [ ] Tiling + preprocessing (Rasterio/GDAL, normalize, contrast) — imagery is too large to feed directly
- [ ] Synthetic occlusion augmentation (shadows, canopy, vehicle patches via Albumentations) — the whole differentiator is occlusion robustness; must train for it
- [ ] Mask → skeleton → graph (scikit-image skeletonize + sknw) — converts pixels to nodes/edges
- [ ] Topological healing (union-find / Disjoint Set + MST, gated by Euclidean distance + angular alignment) — bridges occlusion gaps; the headline graph contribution
- [ ] Betweenness centrality criticality map — identifies "Gatekeeper Nodes" / bottlenecks
- [ ] Node-ablation stress test + Resilience Index (baseline vs perturbed avg shortest path / global efficiency) — the predictive disaster-simulation deliverable
- [ ] Streamlit dashboard with map overlay (streamlit-folium) — makes it demoable to non-technical judges

### Important (v1.1)
- [ ] clDice / connectivity-aware loss (Dice + IoU + boundary + soft-clDice) — measurably cuts fragmentation before healing even runs
- [ ] Attention / transformer context module (SegFormer or attention blocks) — long-range dependency to "see through" occlusion
- [ ] Interactive simulation toggle (click node to disable → live reroute + travel-time delta) — the "wow" interaction in the brief
- [ ] Connectivity Ratio + APLS evaluation harness — quantifies healing gain and topological accuracy vs OSM
- [ ] Edge weighting by length/road-class for realistic travel time — turns the graph from topological to routable
- [ ] GeoJSON / GraphML export — makes output actually usable downstream
- [ ] Conversational planning assistant (LangGraph agent + LangChain tools) — natural-language front door to the analysis; a planner asks a question and the agent runs ablation/criticality/routing tools and explains the result in plain English. High demo value, directly serves the "decision support for non-technical planners" mandate
- [ ] Agentic pipeline orchestration (LangGraph state machine) — model the segmentation→healing→analysis stages as a stateful graph with retries/branching, so partial failures (e.g. a bad tile) are handled gracefully instead of crashing the run

### Nice-to-have (Backlog)
- [ ] Multi-resolution fusion (Sentinel-2 10m + LISS-IV 5.8m + Cartosat-3) — generalization across sources
- [ ] Generative inpainting of occluded regions — optional reconstruction path in the brief
- [ ] Relaxed/buffered IoU (3–5px tolerance) metric — fair scoring of minor alignment shifts
- [ ] Cross-terrain generalization eval (urban / suburban-forested / rural) — robustness story
- [ ] Leaflet.js custom front-end (vs Streamlit-folium) — richer UI if time allows
- [ ] Edge-betweenness (not just node) for segment-level criticality

### Don't Build
- End-to-end image→graph network (DeH4R/SAM-Road) — too complex to reproduce reliably in 30h; segmentation + heal is lower-risk
- Training transformers from scratch — burns the entire GPU budget; fine-tune pretrained instead
- Custom routing engine — NetworkX shortest-path is sufficient for the simulation
- Real-time traffic ingestion — out of scope; static graph is the deliverable
- Custom tile server / heavy GIS stack (full QGIS integration) — Streamlit + folium covers the demo

## Task Breakdown

### Road segmentation model
- [ ] Set up PyTorch + segmentation-models-pytorch, pretrained ResNet U-Net (S)
- [ ] Build training loop with Dice+IoU loss, AMP, checkpointing (M)
- [ ] Train baseline on DeepGlobe/SpaceNet, log IoU/Dice (M) ← depends on: data pipeline
- [ ] Evaluate on held-out occluded tiles, capture failure cases (S)

### OSM auto-labeling & data pipeline
- [ ] Tile imagery with Rasterio/GDAL, write tile index (M)
- [ ] Fetch OSM road vectors (OSMnx/Overpass) for AOI, rasterize to masks (M) ← depends on: tiling
- [ ] Albumentations pipeline incl. synthetic occlusion (shadow/canopy/vehicle) (M)
- [ ] Train/val/test split balanced for occluded road density (S)

### Mask → graph + healing
- [ ] Mask post-process (morphological close, small-object removal) (S) ← depends on: segmentation output
- [ ] Skeletonize + sknw graph build (nodes/edges) (M)
- [ ] Union-find connected components + MST gap bridging with distance+angle gate (L) ← depends on: skeleton graph
- [ ] Weight edges by pixel length / road class; export GeoJSON + GraphML (M)
- [ ] Connectivity Ratio metric (LCC before/after heal) (S)

### Network analysis & stress testing
- [ ] Betweenness centrality + criticality heatmap data (S) ← depends on: weighted graph
- [ ] Node-ablation loop: remove top-k centrality nodes, recompute efficiency (M)
- [ ] Resilience Index (baseline APL / perturbed APL) + per-removal curve (S)
- [ ] APLS topological-accuracy eval vs OSM graph (M)

### Dashboard
- [ ] Streamlit app scaffold + folium map of road graph (M)
- [ ] Criticality heatmap overlay (color by betweenness) (S) ← depends on: centrality
- [ ] Click-to-disable node → live reroute + travel-time delta (L) ← depends on: ablation engine
- [ ] Before/after healing + segmentation overlay toggle (S)

### Advanced model (v1.1)
- [ ] Add soft-clDice to loss, retrain, compare fragmentation (M) ← depends on: baseline
- [ ] Swap/add SegFormer or attention module, benchmark (L) ← depends on: training loop

### Conversational planning assistant (LangChain + LangGraph)
- [ ] Wrap graph operations as LangChain tools (`get_criticality`, `run_ablation`, `shortest_path`, `resilience_index`) (M) ← depends on: analysis + ablation engine
- [ ] Build LangGraph agent loop (plan → call tool → observe → answer) with Claude via `langchain-anthropic` (M) ← depends on: tools
- [ ] Wire the agent into the Streamlit dashboard as a chat panel; render tool outputs on the map (M) ← depends on: dashboard scaffold, agent
- [ ] (Optional) LangGraph state-machine orchestration of the segmentation→healing→analysis pipeline with retry/branch nodes (L)

## Tech Recommendations
| Layer | Recommendation | Reason |
|-------|---------------|--------|
| Segmentation | `segmentation-models-pytorch` U-Net + ResNet34 backbone, upgrade to SegFormer if time | Pretrained encoders = trainable in 30h; matches brief's commended stack |
| Loss | Dice + BCE + soft-clDice | clDice gives ~73% fewer fragments — directly fights the core problem before healing |
| Geo I/O | Rasterio + GDAL + Albumentations | Brief-specified; robust tiling/augmentation |
| Skeleton→graph | scikit-image `skeletonize` + `sknw` | Simplest reliable mask→graph; avoids reinventing vectorization |
| Healing | NetworkX MST + `scipy`/custom union-find, KD-tree for candidate gaps | Distance+angle-gated bridging is cheap, CPU-only, explainable |
| Graph analysis | NetworkX (betweenness, efficiency, shortest path) + OSMnx for OSM ground truth | Don't rebuild graph math; OSMnx loads OSM cleanly |
| Topology metric | APLS (CosmiQ) | Standard, judge-credible, avoids a homemade metric |
| Dashboard | Streamlit + streamlit-folium (Leaflet under the hood) | Fastest path to interactive map demo; Leaflet polish only if time |
| Agent / NL layer | LangGraph (agent + optional pipeline orchestration) + LangChain (tools) + `langchain-anthropic` → **Claude Sonnet 5** (`claude-sonnet-5`) for the loop, **Opus 4.8** (`claude-opus-4-8`) for hardest reasoning | LangGraph gives an explicit, debuggable state machine over tool calls — better than a raw prompt for multi-step graph queries; Claude is the default per house AI-app guidance |
| Compute | GPU for training (local workstation), CPU for all graph/UI/agent | Matches brief; lets the graph subteam work without GPU contention; agent is API-based |

## Risks & Open Decisions
### Risks
- **30h GPU budget** — training SOTA from scratch won't finish → mitigation: fine-tune pretrained backbone, freeze early layers, train at 256px tiles
- **Image↔OSM misalignment** — OSM vectors drift from imagery, poisoning labels → mitigation: buffer OSM lines 3–5px, use relaxed IoU, manual spot-check a few tiles
- **Betweenness ≠ resilience** (recent literature caveat) — pure betweenness may not flag true connectivity-critical nodes → mitigation: also report largest-connected-component drop and global-efficiency alongside betweenness in the Resilience Index
- **Healing hallucination** — MST can bridge gaps that aren't real roads → mitigation: hard cap on bridge distance + angular-alignment threshold; show healed edges in a distinct color for transparency
- **Cartosat-3 data only at event** — model must generalize to unseen high-res source → mitigation: train multi-scale, strong augmentation, validate on a held-back resolution
- **Parallel-team integration risk** — graph subteam mocks vectors while ML trains; interfaces may diverge → mitigation: freeze the mask→graph contract (GeoJSON schema) on hour 1

### Open Decisions
- [ ] Primary training dataset: DeepGlobe vs SpaceNet-3 vs OpenSatMap (pick one for pretraining, recommend DeepGlobe for density)
- [ ] Backbone: stick with U-Net/ResNet (safe) or push SegFormer (higher ceiling, more risk)
- [ ] Dashboard front-end: Streamlit-folium only, or invest in custom Leaflet.js
- [ ] Resilience Index definition: avg-shortest-path ratio (brief) vs global-efficiency ratio (handles disconnection better) — recommend reporting both
- [ ] Edge weight: pure pixel length vs length × road-class speed (for travel-time realism)
- [ ] Target AOI for the demo (Bengaluru tile) and which OSM extract to freeze
- [ ] Conversational assistant scope: read-only Q&A over the graph (safe, fast to build) vs. letting the agent trigger live simulations — recommend read-only + explicit "run simulation" tool for the demo
- [ ] LLM provider: default **Claude via `langchain-anthropic`** (`claude-sonnet-5`); confirm API key availability at the event, else fall back to a local model via `langchain-community`

## GitHub References
- [jeffwen/road_building_extraction](https://github.com/jeffwen/road_building_extraction) — PyTorch U-Net for road/building extraction; clean reference training structure
- [tsdinh442/road-extraction](https://github.com/tsdinh442/road-extraction) — U-Net on DeepGlobe at 256px; tiling + inference pattern to copy
- [hikaruhotta/roadNet](https://github.com/hikaruhotta/roadNet) — UNet + Pix2Pix aiming at urban+rural generalization; relevant to the cross-terrain goal
- [CosmiQ/apls (SpaceNet)](https://github.com/CosmiQ/apls) — APLS topological-accuracy metric; adopt directly for evaluation
- [OSMnx](https://github.com/gboeing/osmnx) — OSM street-network loading + centrality/analysis; the ground-truth + graph-analysis backbone
- [sknw](https://github.com/Image-Py/sknw) — skeleton-to-network conversion; core of the mask→graph step
- [clDice](https://github.com/jocpae/clDice) — topology-preserving loss; the highest-leverage fragmentation fix

---
_Sources: [EFMOD-Net/OARENet occlusion road extraction](https://doi.org/10.3390/rs17173037), [DL road extraction survey](https://www.sciencedirect.com/science/article/pii/S0924271625002758), [vision-transformer road extraction eval](https://www.sciencedirect.com/science/article/pii/S2666017224000749), [Seg-Road](https://www.mdpi.com/2072-4292/15/6/1602), [clDice CVPR'21](https://arxiv.org/abs/2003.07311), [betweenness-as-resilience caveat](https://findingspress.org/article/159015-betweenness-centrality-is-not-a-network-resilience-metric), [Messina road-network disruption case study](https://link.springer.com/article/10.1007/s41109-025-00739-2), [City-Scale road speeds/APLS](https://arxiv.org/pdf/1908.09715)._
