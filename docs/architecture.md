# System Architecture & Data-Flow Diagram
_Issue #2 — Route Resilience Pipeline_

Two sub-teams run in parallel. ML builds the model; Graph builds the analysis pipeline on OSM mock data. They merge at the road mask.

---

```mermaid
flowchart TD
    classDef src  fill:#0d2137,stroke:#4a9eff,color:#cde,font-weight:bold
    classDef prep fill:#0d2108,stroke:#6abf45,color:#cec,font-weight:bold
    classDef ml   fill:#2d0d22,stroke:#d45fb3,color:#ecd,font-weight:bold
    classDef gr   fill:#0d2225,stroke:#45c4c4,color:#cee,font-weight:bold
    classDef an   fill:#252508,stroke:#c4b345,color:#eec,font-weight:bold
    classDef db   fill:#250808,stroke:#c44545,color:#ecc,font-weight:bold
    classDef out  fill:#111,stroke:#aaa,color:#eee,font-weight:bold
    classDef fork fill:#1a1a2e,stroke:#fff,color:#fff,font-weight:bold

    subgraph SRC["📡 Data Sources"]
        direction LR
        IMG["Sentinel-2 · LISS-IV · Cartosat-3"]
        GT["SpaceNet · DeepGlobe · OSM"]
    end
    class IMG,GT src

    PREP["Tile · Augment · OSM Auto-label\nRasterio · GDAL · Albumentations"]
    class PREP prep

    SRC --> PREP
    PREP --> FORK{{"⚡ Teams split here"}}
    class FORK fork

    subgraph ML["🧠 ML Sub-team  [GPU]"]
        direction TB
        MODEL["U-Net + ResNet34\nsegmentation-models-pytorch"]
        LOSS["Dice + clDice Loss"]
        MASK["Binary Road Mask"]
    end
    class MODEL,LOSS,MASK ml

    subgraph GR["🔗 Graph Sub-team  [CPU · no GPU wait]"]
        direction TB
        SKEL["Skeletonise\nscikit-image · sknw"]
        HEAL["MST Healing\nUnion-Find · KD-tree"]
        WGHT["Weighted Graph\nGeoJSON · GraphML"]
    end
    class SKEL,HEAL,WGHT gr

    FORK --> MODEL --> LOSS --> MASK
    FORK --> SKEL
    MASK --> HEAL
    SKEL --> HEAL --> WGHT

    AN["Betweenness Centrality → Gatekeeper Nodes\nNode Ablation → Resilience Index\nNetworkX · OSMnx · APLS"]
    class AN an

    WGHT --> AN

    DASH["Streamlit + Folium\nCriticality Heatmap · Click-to-Disable Sim"]
    class DASH db

    AN --> DASH

    OUT["✅ Routable Graph · Resilience Index · Criticality Map · Live Demo"]:::out
    DASH --> OUT
```

---

## Export to PNG / SVG

```bash
npm install -g @mermaid-js/mermaid-cli
mmdc -i docs/architecture.md -o docs/architecture.png -w 1920
```

Or paste the diagram block into [mermaid.live](https://mermaid.live) → Download PNG.
