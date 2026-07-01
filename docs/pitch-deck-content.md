# Pitch Deck Content: Route Resilience

## Slide 1: Title
**Title:** Route Resilience: Occlusion-Robust Road Extraction & Graph-Theoretic Criticality Analysis
**Track:** ISRO / NNRMS — Satellite EO for Urban Mobility
**Team:** Dubai_Kurukku_sandhu
**Hook:** We turn cloud-, canopy- and shadow-blinded satellite imagery into a *connected, routable* road graph, then mathematically pinpoint the intersections whose failure would break the city — and simulate the fallout.

## Slide 2: Problem
**Header:** The Broken Mask Problem
- **Spectral Blindness:** Satellite imagery in dense cities suffers from tree canopy, shadows, and clouds. Standard ML road masks come out fragmented.
- **Broken Topology:** A fragmented mask is not a network. Gaps and dead-ends make it useless for routing, traffic simulation, or disaster response.
- **The Gap:** Extracting road pixels is solved; extracting a road network you can compute on is not.

## Slide 3: Proposed Solution
**Header:** End-to-End Pipeline
1. **Occlusion-Robust Segmentation:** Context-aware U-Net/SegFormer trained with synthetic occlusions and a topology-preserving clDice loss.
2. **Topological Healing:** Convert mask to graph (skeletonize + `sknw`, cleaned with RDP simplification). Bridge gaps with Union-Find + a **cycle-preserving, distance/angle-gated reconnection** — closes gaps inside grid loops, not just isolated islands (a pure MST can't).
3. **Criticality & Stress Testing:** Flag high Betweenness Centrality nodes ("Gatekeepers"). Simulate node failures and compute a Resilience Index.
4. **Conversational AI Layer:** A LangGraph/Claude assistant that lets planners ask "what-if" questions in plain English.

## Slide 4: Architecture
**Header:** System Architecture
*[Insert image: docs/Prob4Extensivediagram.png]*
- Satellite Imagery -> U-Net Segmentation -> Topological Healing (gated gap-bridging) -> Graph Analysis -> Interactive Agent Dashboard.
*[Insert image: docs/Prob4ShortDia.png for simplified view if needed]*

## Slide 5: Differentiators
**Header:** Why This is Different
- **Connectivity > Pixels:** We optimize for topology (clDice) instead of just pixel IoU.
- **Healed, Routable Graph:** We don't just ship a raster mask; we deliver a weighted, connected vector graph.
- **Actionable Criticality:** We don't just find roads, we find the ones that matter using betweenness centrality.
- **Interactive Simulation:** Live rerouting and travel-time simulation on node failure.
- **Agentic UI:** Plain-English queries powered by LangGraph, not just static expert dashboards.

## Slide 6: Expected Outcomes & Resilience Index
**Header:** Outcomes & Metrics
- **Resilience Index:** Primary metric is **global network efficiency (R_eff)** — it degrades gracefully even when a closure disconnects the graph; average path length (R_apl) is reported alongside (it goes to ∞ on disconnection).
- **High-Fidelity Topology:** A mathematically connected vector network generalizing across terrains.
- **Criticality Map:** A spatial heatmap of high-betweenness single points of failure.
- **Disaster Response & Urban Planning:** Instantly assess isolated sectors and infrastructure weak links.

## Slide 7: Tech Stack
**Header:** Technology Stack
- **ML & CV:** PyTorch, segmentation-models-pytorch, Albumentations
- **Geospatial & Graph:** Rasterio/GDAL, scikit-image, `sknw`, NetworkX, OSMnx
- **App & AI:** Streamlit, folium/Leaflet, LangGraph, LangChain, Anthropic Claude
- **Data:** Cartosat-3 (primary) + pan-sharpened Resourcesat LISS-IV; Sentinel-2 + open sets (SpaceNet/DeepGlobe) for pre-training; OSM vectors (drift-buffered) for ground truth

## Slide 8: Feasibility & Team
**Header:** 30-Hour Build Feasibility
- **Automated, Drift-Aware Labeling:** OSM vectors auto-rasterized with a 3–5 px buffer + relaxed IoU to absorb orthorectification drift; pre-train on open datasets (SpaceNet/DeepGlobe), fine-tune on Cartosat-3.
- **Parallel Workflow:** ML/Data team handles segmentation; Graph/UI team builds healing and dashboard simultaneously using mock vector baselines. They meet at a frozen GeoJSON contract in Hour 1.
- **Compute:** Graph analysis & dashboard are CPU-only; GPU is dedicated to ML fine-tuning.

## Slide 9: Mock Visuals
**Header:** Criticality Heatmap Concept
- **Visual Description:** A dark-themed urban map where the road network glows in blue. Key intersections are marked with pulsing red nodes, scaled by their Betweenness Centrality.
- **Sidebar:** A chat interface where the user asks, "If the main highway floods, what happens?" The map instantly grays out the flooded section and highlights the detours in orange, displaying a 35% drop in the Resilience Index.
