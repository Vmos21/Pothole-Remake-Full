# Phase 1: Field Capture & Telemetry Edge Application (`pothole-field-capture`)

## 1. Project Overview & Objective
This repository contains the mobile and edge-capture application designed for municipal field agents and survey vehicles. Its primary goal is to detect road surface defects, capture high-resolution visual evidence, and bind each defect with precise physical dimensions and high-accuracy GPS coordinates at the moment of capture—even in low-connectivity environments.

---

## 2. System Architecture & Component Interactions
* **Edge Client:** Cross-platform mobile/tablet application or vehicle-mounted camera interface.
* **Two-Stage Edge Vision Pipeline:** Runs a lightweight object detection model (YOLOv8-nano exported to TFLite/ONNX) on-device at 15–30 FPS to draw bounding boxes and verify frame sharpness before shutter capture.
* **Sensor-Assisted Perspective Scaling:** Combines camera focal length with accelerometer/gyroscope pitch angles and ground-plane homography (or ARCore/ARKit plane detection) to convert pixel bounding boxes into real-world centimeter dimensions.
* **Telemetry & Geolocation Module:** Hardware-level GNSS/GPS service capturing latitude, longitude, altitude, heading, and speed.
* **Offline-First Sync Queue (SQLite Store-and-Forward):** Persists validated defect payloads locally on disk when cellular coverage drops and drains the queue to the Phase 2 Ingestion API via background workers with exponential backoff.

---

## 3. Core Modules & Specifications
### A. Image Acquisition & Quality Gate
* Captures high-resolution JPEG/WebP imagery with embedded EXIF timestamps and sensor telemetry.
* Runs a real-time Laplacian variance blur check and lighting validation to block blurry or obstructed frames before upload.

### B. Dimensional Estimation Engine
* Calculates real-world defect geometry without requiring a physical ruler in every frame:
  * **Length ($\text{cm}$)**
  * **Width ($\text{cm}$)**
  * **Surface Area ($\text{cm}^2$)**
  * **Estimated Depth ($\text{cm}$)**
* Provides interactive overlay handles allowing field agents to manually fine-tune bounding boxes if needed.

### C. Precision Geolocation & Local Geofence Cache
* Locks onto high-accuracy GNSS coordinates so road contractors can locate the exact pothole on-site.
* Maintains a lightweight local SQLite `geofence_cache` of recently reported defects within a 1–2 km radius to warn field agents on-screen if they are standing next to an already-logged pothole.

---

## 4. Recommended Tech Stack
* **Mobile / Edge Framework:** React Native (TypeScript) or Flutter
* **Edge Computer Vision:** TensorFlow Lite / ONNX Runtime (YOLOv8-nano)
* **Local Offline Storage:** SQLite / WatermelonDB
* **Sensors & Location:** Native Camera2 / AVFoundation APIs + Fused Location Provider

---

## 5. Repository File Architecture
```text
pothole-field-capture/
├── .github/
│   └── workflows/
│       └── mobile-ci.yml              # Linting, unit tests, and APK/IPA build checks
├── assets/
│   └── models/
│       ├── pothole_yolov8n.tflite     # Quantized edge object-detection model
│       └── labels.txt                 # Defect class labels
├── src/
│   ├── api/
│   │   ├── client.ts                  # HTTP client with JWT refresh interceptors
│   │   └── ingestionService.ts        # Presigned URL fetch & telemetry JSON upload
│   ├── components/
│   │   ├── CameraViewfinder.tsx       # Live camera preview with bounding-box SVG overlay
│   │   ├── DimensionOverlay.tsx       # Interactive length/width/depth adjustment handles
│   │   └── SyncStatusBanner.tsx       # Offline queue counter & manual sync trigger
│   ├── db/
│   │   ├── schema.ts                  # SQLite tables (local_captures, geofence_cache)
│   │   └── captureRepository.ts       # CRUD operations for offline store-and-forward queue
│   ├── navigation/
│   │   └── AppNavigator.tsx           # Stack/Tab navigation (Capture, Queue, Settings)
│   ├── screens/
│   │   ├── CaptureScreen.tsx          # Main viewfinder + shutter + blur warning UI
│   │   ├── ReviewDefectScreen.tsx     # Dimension verification, depth slider & notes
│   │   └── OfflineQueueScreen.tsx     # List of pending/failed/synced local records
│   ├── services/
│   │   ├── blurDetector.ts            # Laplacian variance sharpness check
│   │   ├── dimensionEstimator.ts      # Pitch/roll sensor + homography pixel-to-cm math
│   │   ├── geofenceCache.ts           # Local proximity check to warn of duplicate holes
│   │   ├── locationService.ts         # High-accuracy GNSS/GPS + heading + altitude lock
│   │   └── syncWorker.ts              # Background upload queue with exponential backoff
│   ├── types/
│   │   └── telemetry.ts               # TypeScript interfaces matching Phase 2 API contract
│   └── App.tsx
├── .env.example
├── package.json
├── tsconfig.json
└── README.md
```

---

## 6. Implementation Roadmap
1. **Project Scaffolding:** Initialize repository structure, TypeScript/Dart linting, and environment variables.
2. **Camera & Edge Inference:** Build the viewfinder UI with live TFLite bounding boxes and Laplacian blur detection.
3. **Telemetry & Dimension Math:** Integrate GNSS location locking and sensor-assisted pixel-to-centimeter scaling.
4. **SQLite Offline Buffer:** Implement local disk persistence (`PENDING` $\rightarrow$ `UPLOADING` $\rightarrow$ `SYNCED`) and automatic cleanup of synced media.
5. **API Handshake:** Integrate OAuth2/JWT authentication, presigned image uploads, and end-to-end payload delivery to Phase 2.