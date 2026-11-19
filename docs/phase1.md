# Field Capture & Telemetry Edge Application

## 1. Project Overview & Objective
This repository contains the mobile and edge-capture application designed for municipal field agents and survey vehicles. Its primary goal is to detect road surface defects, capture high-resolution visual evidence, and bind each defect with precise physical dimensions and high-accuracy GPS coordinates at the moment of capture.

## 2. System Architecture & Component Interactions
* **Edge Client:** Cross-platform mobile/tablet application or vehicle-mounted camera interface.
* **On-Device Inference:** Lightweight computer vision model for real-time bounding box detection and blur filtering.
* **Telemetry & Geolocation Module:** Hardware-level GNSS/GPS service capturing latitude, longitude, altitude, heading, and device pitch/roll for perspective correction.
* **Local Offline Cache & Sync Engine:** Queues validated defect payloads locally when cellular connectivity drops and transmits encrypted JSON + image binaries to the Phase 2 Ingestion API once online.

## 3. Core Modules & Specifications
### A. Image Acquisition & Quality Gate
* Captures uncompressed or high-bitrate JPEG/WebP imagery with embedded EXIF timestamps and sensor telemetry.
* Rejects blurry, overexposed, or obstructed frames before upload to ensure contractors receive actionable visual references.

### B. Dimensional Estimation Engine
* Uses reference-object scaling, camera focal-length geometry, or LiDAR/depth-sensor APIs (where supported) to compute:
  * **Length (cm)**
  * **Width (cm)**
  * **Surface Area ($cm^2$)**
  * **Estimated Depth (cm)**

### C. Precision Geolocation & Payload Builder
* Locks onto high-accuracy GNSS coordinates (targeting sub-meter accuracy where hardware permits) so field contractors can locate the exact pothole on-site.
* Packages imagery and structured telemetry into a signed JSON payload for ingestion.

## 4. Recommended Tech Stack
* **Mobile / Edge Framework:** React Native (TypeScript) or Flutter
* **Edge Computer Vision:** TensorFlow Lite / ONNX Runtime (YOLOv8-nano or MobileNet-SSD)
* **Offline Storage:** SQLite / WatermelonDB
* **Media & Telemetry:** Native Camera2 / AVFoundation APIs + Fused Location Provider

## 5. Implementation Roadmap
1. **Project Scaffolding:** Initialize repository structure, TypeScript/Dart linting, and environment configuration.
2. **Camera & Sensor Integration:** Build the viewfinder UI with live bounding-box overlays and EXIF/GNSS telemetry binding.
3. **Dimensional Measurement Module:** Implement depth/scale estimation logic and manual override sliders for field agents.
4. **Offline Queue & Sync Service:** Build local SQLite persistence with exponential-backoff background uploads.
5. **API Handshake:** Integrate OAuth2/JWT authentication and test end-to-end payload delivery to the Phase 2 Ingestion Gateway.