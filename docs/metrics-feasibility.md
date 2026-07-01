# Evaluation Metrics & Feasibility
_Issue #4 — Route Resilience · Idea Submission_

---

## How we measure success

| Metric | What it tells us | Target |
|--------|-----------------|--------|
| **IoU / Dice** | How accurately we segment roads — including ones hidden under tree cover *(Occlusion-Recall)* | IoU > 0.65 on occluded tiles |
| **Connectivity Ratio** | % increase in the largest connected road component after healing — did we actually fix the broken mask? | > 30% gain |
| **APLS** | Do shortest paths on our graph match OSM's? Catches topological errors that pixel metrics miss | > 0.70 |
| **Relaxed IoU** *(3–5 px buffer)* | Avoids penalising roads that are detected but slightly offset — fairer on high-res imagery | Reported alongside strict IoU |

---

## Resilience Index — how we define it

We report **two numbers together**, not just one:

- **R_apl** — baseline avg. path length ÷ perturbed avg. path length *(from the problem brief)*
- **R_eff** — global network efficiency after node removal ÷ before

> Why both? R_apl breaks when the graph disconnects (you get ∞ in the denominator). R_eff handles disconnection cleanly. Together they give a complete picture of how badly a closure hurts.

---

## A caveat on betweenness centrality

Betweenness centrality is a good *starting point* for finding critical nodes — but recent work shows it doesn't always predict which removals hurt the network most.

Our fix: we use betweenness to **shortlist** candidates, then rank them by actual LCC drop and efficiency loss from node ablation. The node that causes the biggest real-world damage gets flagged as the Gatekeeper — not just the one with the highest centrality score.

---

## Can we build this in 30 hours?

Yes — the key is that **both sub-teams run in parallel from hour one.**

| Phase | Who | Est. time |
|-------|-----|-----------|
| Data prep + OSM auto-labelling | Data team | 2–3 h |
| Model fine-tuning *(pretrained ResNet34, not from scratch)* | ML team | 6–8 h |
| Graph construction + gated gap-bridging healing | Graph team | 4–5 h |
| Centrality, ablation, Resilience Index | Graph team | 2–3 h |
| Streamlit dashboard | Dashboard team | 3–4 h |
| Integration + buffer | All | ~6 h |

**Data is not a blocker.** SpaceNet and DeepGlobe are available right now for pre-training. Cartosat-3 tiles arrive at the event and we fine-tune on those — the model doesn't train from zero.

**Compute is not a blocker.** Graph analysis and the dashboard are CPU-only and run without touching the GPU. The ML team has the GPU to themselves the whole time.
