# Government-Grade Geospatial Database & Deduplication Backend

## 1. Project Overview & Objective
This repository houses the secure backend infrastructure, ingestion APIs, and spatial database powering the platform. Designed to meet government and municipal procurement standards, it securely stores pothole records, enforces strict access controls, and prevents duplicate entries through automated geospatial deduplication.

## 2. System Architecture & Component Interactions
* **API Gateway & Auth Layer:** Enforces TLS 1.3 encryption, rate limiting, JWT authentication, and Role-Based Access Control (RBAC).
* **Geospatial Deduplication Service:** Intercepts incoming capture payloads and queries PostGIS to check if a similar pothole already exists within a defined spatial radius.
* **Primary Operational Database:** PostgreSQL + PostGIS instance storing verified pothole geometry, dimensions, status lifecycle, and audit trails.
* **Encrypted Object Storage:** S3-compatible private bucket storing raw capture imagery with server-side KMS encryption and signed short-lived URLs.

## 3. Core Modules & Specifications
### A. Geospatial Deduplication Engine
* Runs a spatial proximity query (`ST_DWithin`) within a configurable threshold (e.g., 2–5 meters) of the incoming coordinates.
* Evaluates dimensional similarity (comparing length, width, and surface area deltas).
* **Resolution Logic:** If a match is found, the system flags the submission as a duplicate, attaches the new photo as corroborating evidence to the existing record, and increments a confirmation counter instead of creating a redundant work order.

### B. Government Security & Compliance Controls
* **Role-Based Access Control (RBAC):** Distinct permission tiers (`Field_Agent`, `Municipal_Engineer`, `Contractor_Viewer`, `System_Admin`).
* **Row-Level Security (RLS):** Ensures contractors and regional jurisdictions can only query records assigned to their authorized zones.
* **Immutable Audit Logging:** Database triggers recording every `INSERT`, `UPDATE`, and `DELETE` operation with user ID, timestamp, and IP metadata.

### C. Core Database Schema Overview
* `potholes`: `id (UUID)`, `geom (GEOGRAPHY(Point, 4326))`, `length_cm`, `width_cm`, `depth_cm`, `area_sq_cm`, `severity_score`, `status (OPEN | SCHEDULED | RESOLVED)`, `created_at`, `updated_at`.
* `pothole_media`: `id (UUID)`, `pothole_id (FK)`, `storage_key`, `captured_by`, `captured_at`.
* `audit_logs`: `id (BIGSERIAL)`, `table_name`, `record_id`, `action`, `actor_id`, `changed_at`, `diff_json`.

## 4. Recommended Tech Stack
* **Database:** PostgreSQL 16 + PostGIS 3.4
* **Backend API:** Python (FastAPI) or Node.js (NestJS / TypeScript)
* **Object Storage:** AWS S3 (with KMS encryption) or MinIO (for on-premise sovereign government hosting)
* **Infrastructure as Code:** Docker + Terraform

## 5. Implementation Roadmap
1. **Database Provisioning:** Deploy PostgreSQL with PostGIS enabled and configure automated encrypted backups.
2. **Schema & RLS Migration:** Create core spatial tables, GIST spatial indexes, audit triggers, and Row-Level Security policies.
3. **Deduplication Middleware:** Implement the `ST_DWithin` + dimensional comparison algorithm and unit-test GPS drift edge cases.
4. **Ingestion & Media APIs:** Build secure endpoints for receiving telemetry payloads and generating pre-signed upload/view URLs.
5. **Security Hardening:** Enforce RBAC middleware, audit logging verification, and penetration-test readiness checks.