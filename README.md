## Delhivery ETA Optimization
Graph‑based ETA prediction and bottleneck analysis for Delhivery's logistics network.

This project builds a graph‑intelligence system to predict delivery times more accurately than the existing OSRM routing engine. By modeling facilities as nodes and corridors as edges, we capture network‑wide delay patterns, identify bottleneck hubs, and recommend optimal route types (FTL vs Carting) – directly reducing SLA breaches and improving operational efficiency.

## Problem Statement
OSRM consistently underestimates actual delivery time on a significant fraction of routes. Real‑world logistics is messy: congestion, facility dwell time, seasonal volume spikes, and route‑type constraints cause actual delivery times to deviate by ~69% on average. Our graph‑based model treats the logistics network as a connected graph – not independent point‑to‑point estimates – to produce more accurate ETAs and identify which hubs and corridors are systematically causing delays.

## Approach
#### 1. Data Cleaning & Feature Engineering
Raw Data: 144,867 trip segments, 24 columns (OSRM estimates, actual times, facility codes, route types, timestamps).

Cleaning: Removed duplicates, negative times, unrealistic speeds, extreme delay ratios; outlier removal using max(6×IQR, 99.5th percentile).

Features Engineered (103 leak‑free):

OSRM‑based (time, distance, speed, log/sqrt transforms)

Circular time features (hour, day, month sin/cos)

Corridor‑level stats (median delay, trip count, IQR, chronic flag)

Hub‑level stats (median delay, trip volume, delay differential)

State‑level encoding

Interactions (route type × hour, distance × route type, etc.)

Trip‑level aggregates (total segments, segment position, trip total OSRM time)

#### 2. Graph Construction
Nodes: 1,383 facilities (source/destination hubs)

Edges: 1,814 corridors with hour‑stratified median delay ratios

Graph Type: Directed weighted graph (route_type, hour, delay ratio, trip count, etc.)

Metrics: Betweenness centrality, in/out‑degree, clustering coefficient, PageRank, SLA contribution

#### 3. Graph Embeddings (Node2Vec)
Dimensions: 64 (raw) → 20 (PCA, 95.6% variance retained)

Walk Parameters: p=1, q=0.5 (DFS‑like exploration for logistics networks)

Output: Source and destination embeddings merged into trip data

#### 4. Model Training
Model	MAE (min)	R²	Within 15% of actual
XGBoost Baseline (no graph)	7.70	0.463	43.7%
XGBoost + Graph	7.64	0.467	44.0%
LightGBM + Graph	7.69	0.458	43.6%
Stacked Ensemble	7.90	0.467	42.9%
Target: sqrt(segment_actual_time) – best transform (skew → 0.42)

Best Model: XGBoost with node2vec embeddings + graph metrics

Graph Advantage: MAE improved by 0.07 min; R² improved by 0.004

## Key Findings
Top 5 Bottleneck Hubs (by SLA contribution)
Rank	Hub Code	SLA Share	Betweenness
1	IND000000ACB	6.82%	0.1127
2	IND562132AAA	3.21%	0.0564
3	IND421302AAG	3.10%	0.0292
4	IND712311AAA	2.01%	0.0353
5	IND160002AAC	1.95%	0.0282
These five hubs cause 17.1% of all national delay minutes. Fixing the top 3 alone would reduce SLA breaches by over 13%.

FTL vs Carting Trade‑Off
Distance	Carting (min)	FTL (min)	FTL Saves (min)	Recommendation
0–20 km	16	18	-2	Carting
20–50 km	34	33	+1	Either
50–100 km	83	64	+19	FTL
100–200 km	271	124	+147	FTL
200–500 km	646	594	+52	FTL
Classifier Accuracy: 93% (Random Forest with dispatch‑time features)

## Live Dashboard
URL: https://delhivery-eta-optimization-yvvahmmn9a9xzfh3punncn.streamlit.app/


## 📁 Repository Structure
text
delhivery-eta-optimization/
├── app.py                          # Streamlit dashboard
├── requirements.txt                # Python dependencies
├── README.md                       # This file
├── .gitignore                      # Exclude large files
├── ETA_optimization_final_1.ipynb  # Complete Jupyter notebook (all phases)
├── models/                         # Trained models
│   ├── eta_model_xgb_graph.pkl     # Best ETA model
│   ├── route_classifier.pkl        # FTL vs Carting classifier
│   ├── deployment_artifacts.pkl    # Feature lists & fill values
│   └── pca_embeddings.pkl          # PCA transformer for embeddings
├── data/                           # Data files (small; large CSVs excluded from GitHub)
│   ├── edges_data.csv              # Corridor-level delay data
│   ├── hub_metrics.csv             # Hub bottleneck metrics
│   └── ftl_carting_tradeoff.csv    # Trade-off table
└── outputs/                        # Visualizations & reports
    ├── network_visualization.png
    ├── residual_analysis.png
    ├── feature_importance_best_model.png
    ├── bottleneck_summary.json
    └── ... (other .png files)


## Results Summary
Best ETA Model: XGBoost + Graph → MAE 7.64 min, R² 0.467, 44.0% within 15% of actual.

Graph Advantage: MAE improved by 0.07 min; R² improved by 0.004.

Bottleneck Audit: Top 5 hubs cause 17.1% of all delay – specific interventions recommended.

FTL vs Carting: Classifier accuracy 93%; FTL saves 19–147 min on routes >50 km.

Revenue‑at‑Risk: Estimated ₹4.6 crore/month (assuming ₹30/min delay cost).

🔗 Live Demo
Streamlit Dashboard: https://delhivery-eta-optimization-yvvahmmn9a9xzfh3punncn.streamlit.app/
