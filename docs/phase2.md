# Phase 2: Government-Grade Geospatial Database & Deduplication Backend (`pothole-secure-core`)

## 1. Project Overview & Objective
This repository houses the secure backend infrastructure, ingestion APIs, and geospatial database powering the platform. Designed to meet government and municipal procurement standards, it securely stores pothole records, enforces strict multi-tenant access controls, and prevents duplicate entries through automated spatial and perceptual deduplication.

---

## 2. System Architecture & Component Interactions
* **API Gateway & Auth Layer:** Enforces TLS 1.3 encryption, rate limiting, JWT authentication, and Role-Based Access Control (RBAC).
* **Event-Driven Idempotent Ingestion:** Validates deterministic client-side UUIDs (`capture_id`) so network retries never create duplicate database records.
* **Geospatial & Perceptual Deduplication Service:** Intercepts incoming payloads, queries PostGIS for nearby defects, and compares dimensional/visual hashes before committing new work orders.
* **Primary Operational Database:** PostgreSQL + PostGIS instance storing verified pothole geometry, dimensions, status lifecycle, and immutable audit trails.
* **Zero-Trust Media Storage:** Issues short-lived, KMS-encrypted S3/MinIO presigned `PUT` URLs so raw imagery uploads directly to object storage without overloading the API server.

---

## 3. Core Modules & Specifications
### A. Two-Tier Geospatial Deduplication Engine
1. **Spatial Proximity Filter:** Runs a PostGIS `ST_DWithin` query on a `GEOGRAPHY(Point, 4326)` column (indexed with GIST) within a configurable radius (e.g., 3 meters).
2. **Attribute & Visual Similarity Match:** Compares surface area deltas and a 64-bit perceptual hash (`pHash`) of the defect image.
3. **Progression Tracking:** If a match is found, the system flags the submission as a duplicate, attaches the new photo to a `pothole_observations` child table to track defect growth over time, and increments a confirmation counter instead of creating a redundant repair ticket.

### B. Government Security & Compliance Controls
* **Role-Based Access Control (RBAC):** Distinct permission tiers (`Field_Agent`, `Municipal_Engineer`, `Contractor_Viewer`, `System_Admin`).
* **Row-Level Security (RLS):** Database-enforced policies ensuring contractors and regional ward engineers only query records within their authorized jurisdictions.
* **Immutable Audit Logging:** PostgreSQL triggers recording every `INSERT`, `UPDATE`, and `DELETE` operation with actor ID, timestamp, and JSON diff metadata.

### C. Core Database Schema Overview
* `potholes`: `id (UUID)`, `geom (GEOGRAPHY(Point, 4326))`, `length_cm`, `width_cm`, `depth_cm`, `area_sq_cm`, `phash`, `severity_score`, `ward_id`, `status (OPEN | SCHEDULED | RESOLVED)`, `created_at`, `updated_at`.
* `pothole_observations`: `id (UUID)`, `pothole_id (FK)`, `storage_key`, `measured_area_sq_cm`, `captured_by`, `captured_at`.
* `audit_logs`: `id (BIGSERIAL)`, `table_name`, `record_id`, `action`, `actor_id`, `changed_at`, `diff_json`.

---

## 4. Recommended Tech Stack
* **Database:** PostgreSQL 16 + PostGIS 3.4
* **Backend API:** Python (FastAPI) or Node.js (NestJS / TypeScript)
* **Object Storage:** AWS S3 (with KMS encryption) or MinIO (for sovereign on-premise government hosting)
* **Infrastructure:** Docker + Alembic Migrations

---

## 5. Repository File Architecture
```text
pothole-secure-core/
├── .github/
│   └── workflows/
│       └── backend-ci.yml             # Pytest, PostGIS integration tests, security linting
├── alembic/
│   ├── versions/
│   │   ├── 001_enable_postgis.py      # PostGIS extension & GIST spatial indexes
│   │   ├── 002_core_tables.py         # potholes, pothole_observations, users
│   │   └── 003_rls_and_audits.py      # Row-Level Security policies & audit triggers
│   └── env.py
├── app/
│   ├── api/
│   │   ├── deps.py                    # JWT verification, RBAC role guards, DB session
│   │   └── v1/
│   │       ├── endpoints/
│   │       │   ├── ingest.py          # POST /ingest (idempotent capture submission)
│   │       │   ├── media.py           # POST /media/presign (KMS-encrypted S3/MinIO URLs)
│   │       │   ├── potholes.py        # GET/PATCH pothole records & status lifecycle
│   │       │   └── tiles.py           # GET /tiles/{z}/{x}/{y}.mvt (PostGIS vector tiles)
│   │       └── router.py
│   ├── core/
│   │   ├── config.py                  # Pydantic settings (DB URL, S3 keys, radius thresholds)
│   │   └── security.py                # Password hashing, JWT decoding, RBAC scopes
│   ├── models/
│   │   ├── audit.py                   # Immutable audit_logs ORM model
│   │   ├── pothole.py                 # Geography(Point, 4326) & dimensional columns
│   │   └── user.py                    # Users, roles, and assigned municipal ward_ids
│   ├── schemas/
│   │   ├── ingestion.py               # Strict Pydantic validation for incoming payloads
│   │   └── pothole.py                 # Response schemas & GeoJSON serializers
│   ├── services/
│   │   ├── deduplicator.py            # ST_DWithin spatial query + area delta + pHash check
│   │   ├── imageHasher.py             # 64-bit perceptual hashing (pHash) utility
│   │   └── storageService.py          # S3/MinIO presigned PUT/GET URL generator
│   └── main.py                        # FastAPI entrypoint, CORS, rate-limiting middleware
├── docker/
│   ├── Dockerfile
│   └── init-postgis.sql               # Initial DB roles and PostGIS setup
├── tests/
│   ├── test_deduplication.py          # GPS drift & duplicate merge edge-case tests
│   └── test_rls_security.py           # Cross-ward unauthorized access verification
├── docker-compose.yml                 # Local stack: FastAPI + PostGIS + MinIO
├── pyproject.toml
└── README.md
```

---

## 6. Implementation Roadmap
1. **Database Provisioning:** Deploy PostgreSQL with PostGIS enabled and configure automated encrypted backups.
2. **Schema & RLS Migration:** Create core spatial tables, GIST indexes, audit triggers, and Row-Level Security policies.
3. **Deduplication Middleware:** Implement the `ST_DWithin` + dimensional/pHash comparison engine and test GPS drift edge cases.
4. **Ingestion & Vector Tile APIs:** Build secure endpoints for telemetry ingestion, presigned media URLs, and Mapbox Vector Tile (`ST_AsMVT`) streaming.
5. **Security Hardening:** Verify RBAC isolation across municipal roles and audit log immutability.