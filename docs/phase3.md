# Phase 3: GIS Command Dashboard, Traffic Triage & ML Cost Estimation Engine (`pothole-command-analytics`)

## 1. Project Overview & Objective
This repository contains the municipal command web application and the machine learning pricing engine. It visualizes verified potholes on an interactive GIS map, ranks repair urgency on a 1–10 scale using vehicle traffic density, predicts individual repair costs from an isolated historical repair database, and enables commercial multi-pothole batch quoting.

---

## 2. System Architecture & Component Interactions
* **GIS Command Web App:** Interactive vector-tile map interface displaying real-world pothole markers, high-resolution images, exact dimensions, and triage scores.
* **Asynchronous Triage Scoring Worker:** Background worker that cross-references pothole coordinates with road segment traffic density data to compute a 1–10 urgency score.
* **Isolated ML Training Database:** Dedicated historical dataset linking completed pothole repairs (dimensions, road class, material) to actual verified contractor invoices.
* **Cost Prediction & Batching Microservice:** Serves single-hole cost estimates via a trained regression model and computes bundled commercial project pricing with route-optimized economies of scale.

---

## 3. Core Modules & Specifications
### A. Interactive GIS Map & Defect Inspector
* Streams Mapbox Vector Tiles (`MVT`) from Phase 2 so thousands of potholes render smoothly at any zoom level.
* Clicking any marker opens an inspection drawer displaying the capture photo, exact physical dimensions ($\text{length}$, $\text{width}$, $\text{depth}$, $\text{area}$), GPS coordinates, 1–10 urgency score, and predicted repair cost.

### B. Dynamic Priority Scoring (Scale 1–10)
* Combines physical defect severity with real-time or historical **vehicle traffic density** (Average Annual Daily Traffic / congestion index) on that specific road segment:
  $$\text{Priority} = w_1(\text{Normalized Area} \times \text{Depth}) + w_2(\text{Traffic Density}) + w_3(\text{Road Class Multiplier})$$
* Normalizes output to a clean **1–10 Urgency Scale** so high-traffic arterial hazards are prioritized immediately.

### C. ML Repair Cost Prediction Engine
* **Historical Training DB:** Stores resolved repair records (`area_sq_cm`, `depth_cm`, `road_type`, `asphalt_grade`, `labor_region`, `actual_cost`) isolated from operational tables.
* **Model Pipeline:** Trains gradient-boosted regression models (XGBoost / LightGBM / Scikit-Learn) to predict baseline repair costs for newly detected potholes.

### D. Commercial Multi-Project Batching Module
* Allows municipal engineers or commercial contractors to multi-select several potholes via map polygon/lasso selection or table filtering.
* Uses spatial routing heuristics to cluster nearby defects and calculate a **Bundled Project Estimate**, amortizing fixed expenses (equipment mobilization, crew transport, and bulk asphalt batching) across all selected holes.

---

## 4. Recommended Tech Stack
* **Frontend & GIS:** React (TypeScript), Tailwind CSS, Mapbox GL JS or Leaflet
* **ML & Pricing Microservice:** Python (FastAPI, Scikit-Learn, XGBoost, Pandas, OR-Tools)
* **Traffic Data Integration:** Municipal GIS Traffic API / TomTom or HERE Traffic Flow API
* **Training Data Store:** PostgreSQL (isolated analytics schema)

---

## 5. Repository File Architecture
```text
pothole-command-analytics/
├── frontend/                          # React + TypeScript + Mapbox GL / Leaflet
│   ├── public/
│   ├── src/
│   │   ├── api/
│   │   │   ├── coreApi.ts             # Connects to Phase 2 Pothole & MVT Tile APIs
│   │   │   └── pricingApi.ts          # Connects to ML Cost & Batching service
│   │   ├── components/
│   │   │   ├── map/
│   │   │   │   ├── PotholeMap.tsx     # Vector tile map with priority color-coding
│   │   │   │   └── BatchSelector.tsx  # Lasso/polygon multi-select tool on map
│   │   │   ├── inspector/
│   │   │   │   ├── DefectDrawer.tsx   # High-res photo, dimensions, 1-10 score, cost
│   │   │   │   └── PriorityBadge.tsx  # Visual 1-10 urgency indicator
│   │   │   └── commercial/
│   │   │       └── BatchQuoteModal.tsx# Bundled cost breakdown & PDF/CSV export
│   │   ├── hooks/
│   │   │   └── useBatchSelection.ts   # State management for multi-pothole quotes
│   │   ├── pages/
│   │   │   ├── CommandDashboard.tsx   # Main GIS triage view
│   │   │   └── HistoricalLedger.tsx   # Admin view to log completed repair invoices
│   │   └── App.tsx
│   ├── package.json
│   └── tsconfig.json
│
├── ml-pricing-service/                # Python FastAPI + XGBoost/Scikit-Learn + OR-Tools
│   ├── app/
│   │   ├── api/
│   │   │   ├── predict.py             # POST /predict-cost (single pothole estimate)
│   │   │   ├── batch.py               # POST /batch-quote (multi-hole mobilization math)
│   │   │   └── triage.py              # POST /score-priority (traffic-weighted 1-10 score)
│   │   ├── services/
│   │   │   ├── batchOptimizer.py      # Route clustering & mobilization cost amortization
│   │   │   ├── modelLoader.py         # Loads serialized .joblib/.ubj regression artifact
│   │   │   └── trafficClient.py       # Fetches AADT / live congestion index for road segment
│   │   └── main.py
│   ├── ml_pipeline/                   # Offline training & evaluation scripts
│   │   ├── data/
│   │   │   └── schema_historical.sql  # Isolated historical_repairs training table
│   │   ├── extract_features.py        # Area, depth, volume, road class, region encoding
│   │   ├── train_cost_model.py        # Trains XGBoost regressor & logs RMSE/MAE
│   │   └── artifacts/
│   │       └── cost_model_v1.joblib   # Trained model binary
│   ├── tests/
│   │   ├── test_priority_scoring.py   # Verifies 1-10 scaling across traffic extremes
│   │   └── test_batch_discounts.py    # Verifies economies-of-scale bundle math
│   └── pyproject.toml
│
├── docker-compose.yml                 # Runs Frontend + ML Pricing Service
└── README.md
```

---

## 6. Implementation Roadmap
1. **GIS Dashboard Foundation:** Build the React vector-tile map interface connected to Phase 2 APIs with marker clustering and detail drawers.
2. **Traffic Triage Engine:** Integrate road traffic density lookups and implement the 1–10 weighted priority scoring formula.
3. **ML Training Pipeline:** Structure the isolated historical repair cost database, build feature engineering scripts, and train the initial cost regression model.
4. **Commercial Batch Calculator:** Build the multi-select map tool and route/mobilization discount algorithm for bundled project quotes.
5. **Export & Reporting:** Add PDF/CSV tender generation for contractor work orders.

---

## 7. Future Roadmap (Reserved Architecture Hooks)
* **Contractor Post-Repair Verification App (Tentative):** A dedicated mobile workflow requiring contractors to capture a post-repair photo at the exact GPS coordinates of the original defect. A dual-stage computer vision pipeline will verify:
  1. **Spatial & Background Scene Alignment:** Confirming the photo was taken at the exact same location and angle.
  2. **Genuine Asphalt Remediation:** Filtering out fraudulent, incomplete, or placeholder photos before authorizing municipal payout.