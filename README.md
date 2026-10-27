# 🏛️ Pro AI Coaching – System Architecture

[![Flutter](https://img.shields.io/badge/Flutter-02569B?style=flat&logo=flutter&logoColor=white)](https://flutter.dev)
[![Firebase](https://img.shields.io/badge/Firebase-FFCA28?style=flat&logo=firebase&logoColor=black)](https://firebase.google.com)
[![Google Gemini](https://img.shields.io/badge/Gemini_API-8E75C2?style=flat&logo=google&logoColor=white)](https://deepmind.google/technologies/gemini/)
[![RevenueCat](https://img.shields.io/badge/RevenueCat-E7525A?style=flat&logo=revenuecat&logoColor=white)](https://www.revenuecat.com/)
[![Security: App Check](https://img.shields.io/badge/Security-Play_Integrity_%7C_DeviceCheck-brightgreen)](https://firebase.google.com/docs/app-check)

Pro AI Coaching features an end-to-end hybrid mobile architecture engineered to process athletes' historical workout logs, fatigue/RPE metrics, multimodal meal inputs, and progressive overload telemetry to deliver real-time biomechanical and nutritional adaptations.

This document outlines the client architecture, cloud services, artificial intelligence pipelines, and security protocols structured according to the C4 model standards.

---

## 📌 Level 1: System Context Diagram

Visualizes user interactions, platform boundaries, and third-party enterprise integrations.

```mermaid
flowchart TB
    User(["🏋️‍♂️ Athlete / End User"])

    subgraph SystemBoundary ["Pro AI Coaching Ecosystem"]
        App["📱 Pro AI Coaching Mobile Client<br/>(Flutter - iOS & Android)"]
    end

    subgraph CloudInfra ["Google Cloud & Firebase Infrastructure"]
        BackendProxy["⚡ Cloud Functions Middleware<br/>(API Gateway & Rate Limiter)"]
        DB[("🔥 Cloud Firestore<br/>(User Telemetry & Workout Logs)")]
        Storage[("📦 Cloud Storage<br/>(Meal & Form Analysis Media)")]
        Auth["🔑 Firebase Authentication"]
    end

    subgraph ExternalServices ["External Integrations & APIs"]
        Gemini["🧠 Google Gemini API / Vertex AI<br/>(Multimodal Vision & LLM Engine)"]
        RC["💳 RevenueCat & StoreKit / Play Billing<br/>(Subscription & Entitlement Engine)"]
        Security["🛡️ Firebase App Check<br/>(Play Integrity & DeviceCheck)"]
    end

    User <-->|"Input (Voice, Text, Vision)<br/>Feedback (Workouts, Macros)"| App

    App <-->|"Authentication"| Auth
    App <-->|"Real-Time Telemetry Sync"| DB
    App -->|"Media Upload"| Storage
    App <-->|"Secure API Requests (JWT, App Check Token)"| BackendProxy
    App <-->|"Subscription & In-App Purchases"| RC
    App -->|"Client Attestation Request"| Security

    BackendProxy <-->|"Structured Prompting & JSON Output"| Gemini
    Security -.->|"Device Attestation Provider"| BackendProxy
```

---

## 📌 Level 2: Container Diagram

Models the high-level technical runtime, responsibilities of independent containers (client, serverless middleware, databases, storage), and inter-service communication protocols.

```mermaid
C4Container
title Container Diagram (Level 2) - Pro AI Coaching

Person(user, "Athlete / User", "End user logging exercises, receiving real-time AI biomechanical form and nutritional feedback.")

Container_Boundary(c1, "Pro AI Coaching Platform Boundaries") {
    Container(mobile_app, "Mobile Client Application", "Flutter, Dart", "Clean Architecture, reactive state management, offline-first local cache, native video rendering engine (Media3/ExoPlayer).")
    Container(backend_proxy, "Cloud Functions Middleware", "Node.js / Cloud Functions", "Secure API Gateway. Token throttling, prompt orchestration, App Check attestation, and subscription verification.")
    ContainerDb(firestore, "Cloud Firestore", "NoSQL Document DB", "User profiles, workout history, set/rep telemetry, and progressive overload analytics.")
    ContainerDb(storage, "Cloud Storage", "Object Storage", "Meal photographs and video recordings for form analysis.")
    Container(auth, "Firebase Authentication", "Identity Provider", "User registration, authentication, sessions, and JWT issuance.")
}

System_Ext(gemini, "Google Gemini API / Vertex AI", "Multimodal Vision & LLM. Food macro extraction and dynamic workout adaptation.")
System_Ext(revenuecat, "RevenueCat & StoreKit", "Apple App Store & Google Play Billing subscription lifecycle and entitlement engine.")
System_Ext(app_check, "Firebase App Check", "Play Integrity API & Apple DeviceCheck client attestation provider.")

Rel(user, mobile_app, "Interacts with", "Flutter UI")
Rel(mobile_app, auth, "Authenticates", "Firebase Auth SDK / HTTPS")
Rel(mobile_app, firestore, "Syncs workouts and profile telemetry", "Firestore SDK / gRPC")
Rel(mobile_app, storage, "Uploads meal photos and form videos", "Cloud Storage SDK")
Rel(mobile_app, revenuecat, "Queries subscription & paywall status", "RevenueCat SDK / HTTPS")
Rel(mobile_app, app_check, "Fetches device integrity token", "Play Integrity / DeviceCheck")

Rel(mobile_app, backend_proxy, "Sends AI analysis and coaching requests", "HTTPS / JSON (JWT + App Check)")
Rel(backend_proxy, gemini, "Dispatches structured prompts & extracts JSON schemas", "HTTPS / REST API")
Rel(backend_proxy, revenuecat, "Validates active subscription entitlements", "REST API")
Rel(backend_proxy, firestore, "Persists generated coaching logs and analytics", "Firebase Admin SDK")
```

### Container & Protocol Specifications

| Container / Component | Technology Stack | Network Protocol | Primary Responsibility |
| :--- | :--- | :--- | :--- |
| **Mobile Client** | Flutter, Dart | Native SDK & HTTPS | Cross-platform UI, reactive state handling, and local media processing. |
| **Backend Middleware** | Firebase Cloud Functions | HTTPS / REST (JSON) | API key isolation, rate-limiting, and prompt orchestration. |
| **Database** | Cloud Firestore | gRPC / Firestore SDK | Offline-first sync for workout telemetry, user profiles, and hypertrophy trends. |
| **Media Storage** | Cloud Storage | HTTPS / Storage SDK | Secure object storage for nutritional images and form analysis videos. |
| **AI Inference Engine**| Gemini API / Vertex AI | HTTPS / REST | Multimodal meal analysis, real-time exercise substitution, and coaching inferences. |
| **Client Attestation** | Firebase App Check | Play Integrity / DeviceCheck | Prevents unauthorized bots, emulators, and API abuse via cryptographic attestation. |

---

## 📌 Level 3: Component Diagram (Backend & RAG Pipeline)

Illustrates the internal service architecture of the backend tier, highlighting request validation, semantic search pipelines with pgvector, and contextual RAG (Retrieval-Augmented Generation) flows.

```mermaid
C4Component
title Component Diagram (Level 3) - Backend & RAG Pipeline

Container(mobile_app, "Mobile Client (Flutter)", "Dart, Riverpod", "Dispatches HTTPS requests with user telemetry context.")
ContainerDb(pg_db, "PostgreSQL & pgvector", "SQL Vector Database", "Stores relational profiles, telemetry logs, and high-dimensional semantic embeddings.")
System_Ext(gemini_api, "Google Gemini API", "LLM Inference Engine", "Produces context-augmented workout adjustments and nutritional breakdowns.")

Container_Boundary(api_boundary, "FastAPI Backend Boundary") {
    Component(auth_controller, "Auth & Security Handler", "FastAPI / JWT", "Handles token validation, entitlement verification, and rate-limiting.")
    Component(workout_router, "Workout & Telemetry Router", "FastAPI Endpoints", "Ingests exercise streams: sets, reps, weight, and RPE telemetry.")
    Component(schema_validator, "Pydantic Validator", "Pydantic v2", "Strict payload type validation and schema standardization.")
    Component(rag_orchestrator, "RAG Orchestrator", "Python Async Core", "Coordinates query analysis, prompt templating, and contextual inference pipelines.")
    Component(vector_repo, "Vector Repository", "asyncpg / SQLAlchemy", "Executes semantic cosine-distance lookups across athlete historical logs.")
    Component(gemini_client, "Gemini SDK Client", "Google AI SDK", "Dispatches optimized system prompts and receives strictly typed JSON outputs.")
}

Rel(mobile_app, auth_controller, "Authenticates request via Bearer JWT", "HTTPS / JSON")
Rel(mobile_app, workout_router, "Sends workout metrics / requests adaptation", "HTTPS / JSON")

Rel(workout_router, schema_validator, "Validates payload structure", "Python Internal Call")
Rel(schema_validator, rag_orchestrator, "Forwards sanitized payload", "In-Memory")

Rel(rag_orchestrator, vector_repo, "Queries historical similarity context", "Async Call")
Rel(vector_repo, pg_db, "Performs semantic vector search (<-> operator)", "SQL / pgvector")

Rel(rag_orchestrator, gemini_client, "Builds enriched prompt with athlete context", "Structured Prompt")
Rel(gemini_client, gemini_api, "Dispatches multimodal inference request", "HTTPS REST")
```

### Component Architectural Highlights

1. **Deterministic RAG Orchestration:** Requests are not passed directly to the raw model. The `Vector Repository` retrieves the athlete's last 3–4 weeks of volume loading, active joint fatigue markers, and past failure points, injecting them as grounded context into the prompt schema to prevent hallucinations.
2. **Strict Type Safety (Pydantic v2):** Missing workout variables, corrupt RPE values, or malformed telemetry payloads are intercepted and rejected at the gateway boundary before hitting backend compute.
3. **Non-Blocking Asynchronous I/O:** Relational transactions and vector similarity lookups run concurrently via connection-pooled `asyncpg` drivers to minimize inference latency during live gym sessions.
