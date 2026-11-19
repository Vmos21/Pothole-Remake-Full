# GIS Command Dashboard, Traffic Triage & ML Cost Estimation Engine

## 1. Project Overview & Objective
This repository contains the municipal command web application and the machine learning pricing engine. It visualizes verified potholes on an interactive real-world map, ranks repair urgency on a 1–10 scale using vehicle traffic density, predicts individual repair costs from historical data, and enables commercial multi-pothole batch quoting.

## 2. System Architecture & Component Interactions
* **GIS Command Web App:** Interactive map interface displaying real-world pothole markers, high-resolution images, exact dimensions, and triage scores.
* **Traffic-Weighted Triage Service:** Cross-references pothole coordinates with road segment traffic density data to compute a 1–10 priority score.
* **Isolated ML Training Database:** Dedicated historical dataset linking completed pothole repairs (dimensions, road class, material) to actual verified contractor invoices.
* **Cost Prediction & Batching Engine:** Serves single-hole cost estimates and computes bundled commercial project pricing with economies of scale.

## 3. Core Modules & Specifications
### A. Interactive GIS Map & Defect Inspector
* Plots all active potholes using clustered vector markers color-coded by priority score.
* Clicking any marker opens a detailed inspection drawer showing the capture photo, exact dimensions (length, width, depth, area), GPS coordinates, and estimated repair cost.

### B. Dynamic Priority Scoring (Scale 1–10)
* Combines physical severity (surface area and depth) with real-time or historical **vehicle traffic density** (Average Annual Daily Traffic / congestion index) on that specific road segment.
* Outputs an actionable **1–10 Urgency Score** so high-traffic arterial hazards are prioritized over low-traffic residential defects.

### C. ML Repair Cost Prediction Engine
* **Historical Training DB:** Stores resolved repair records (`area_sq_cm`, `depth_cm`, `road_type`, `asphalt_grade`, `labor_region`, `actual_cost`).
* **Model Pipeline:** Trains gradient-boosted regression models (e.g., XGBoost / LightGBM / Scikit-Learn) to predict the baseline repair cost for newly detected potholes.

### D. Commercial Multi-Project Batching Module
* Allows municipal engineers or commercial contractors to multi-select several potholes via polygon selection or list filtering.
* Computes a **Bundled Project Estimate** that optimizes total cost by amortizing fixed expenses (equipment mobilization, crew transport, and bulk asphalt batching) across all selected holes.

## 4. Recommended Tech Stack
* **Frontend & GIS:** React (TypeScript), Tailwind CSS, Mapbox GL JS or Leaflet
* **ML & Pricing Microservice:** Python (FastAPI, Scikit-Learn, XGBoost, Pandas)
* **Traffic Data Integration:** Municipal GIS Traffic API / TomTom or HERE Traffic Flow API
* **Training Data Store:** PostgreSQL (isolated analytics schema)

## 5. Implementation Roadmap
1. **GIS Dashboard Foundation:** Build the React map interface connected to the Phase 2 read APIs with marker clustering and detail drawers.
2. **Traffic Triage Engine:** Integrate road traffic density lookups and implement the 1–10 weighted priority scoring formula.
3. **ML Training Pipeline:** Structure the historical repair cost schema, build feature engineering scripts, and train the initial cost regression model.
4. **Commercial Batch Calculator:** Build the multi-select UI and route/mobilization discount algorithm for bundled project quotes.
5. **Export & Reporting:** Add PDF/CSV tender generation for contractor work orders.

## 6. Future Roadmap (Reserved Architecture Hooks)
* **Contractor Post-Repair Verification App (Tentative):** A dedicated mobile workflow requiring contractors to capture a post-repair photo at the exact GPS coordinates of the original defect. A dual-stage computer vision pipeline will verify (1) spatial and background scene alignment (confirming it is the exact same location) and (2) genuine asphalt remediation (filtering out fraudulent or placeholder photos) before authorizing payout.