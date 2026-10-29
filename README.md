# 🏛️ Pro AI Coaching – Enterprise System Architecture & Engineering Documentation

---

## 📌 Executive Summary

**Pro AI Coaching** is an enterprise-grade mobile health and fitness platform engineered with Flutter. The system ingests and processes high-frequency workout telemetry (set/rep volume, Rate of Perceived Exertion [RPE], fatigue indexes) alongside multimodal meal imagery to deliver real-time biomechanical feedback and dynamic nutritional adaptations.

The architecture eliminates client-side API key exposure through a secure serverless gateway, executes high-dimensional semantic lookups via Cloud Firestore Vector Search, and isolates billing operations using RevenueCat. This document details the end-to-end design across all four levels of the C4 model, along with data persistence schemas, RAG orchestration flows, and security protocols.

---

## 🏛️ Level 1: System Context Diagram

The System Context diagram models the platform boundaries, core user personas, and external enterprise integrations.

```mermaid
flowchart TB
    User(["🏋️‍♂️ Athlete / End User"])

    subgraph SystemBoundary ["Pro AI Coaching Platform Ecosystem"]
        App["📱 Pro AI Coaching Mobile Client<br/>(Flutter iOS & Android)"]
    end

    subgraph CloudInfra ["Google Cloud & Firebase Infrastructure"]
        BackendProxy["⚡ Cloud Functions Middleware<br/>(API Gateway, Auth & Rate Limiter)"]
        DB[("🔥 Cloud Firestore<br/>(User Telemetry, Logs & Vector Store)")]
        Storage[("📦 Cloud Storage<br/>(Meal Imagery & Form Videos)")]
        Auth["🔑 Firebase Authentication<br/>(JWT Identity Provider)"]
        RemoteConfig["⚙️ Firebase Remote Config<br/>(Dynamic Flags & Paywalls)"]
    end

    subgraph ExternalServices ["External Integrations & APIs"]
        Gemini["🧠 Google Gemini Multimodal API<br/>(Vision & LLM Inference Engine)"]
        RC["💳 RevenueCat & StoreKit / Play Billing<br/>(Subscription & Entitlement Engine)"]
        Security["🛡️ Firebase App Check<br/>(Play Integrity & Apple DeviceCheck)"]
    end

    User <-->|"Telemetry, Logs & Media<br/>Coaching Feedback & Analytics"| App

    App <-->|"Authentication (OAuth & JWT)"| Auth
    App <-->|"Client Config & Feature Toggles"| RemoteConfig
    App <-->|"Offline-First State Sync"| DB
    App -->|"Encrypted Media Upload"| Storage
    App <-->|"Subscription Lifecycle & Paywalls"| RC
    App -->|"Device Attestation Tokens"| Security
    App <-->|"Secure Coaching Inquiries (JWT + App Check)"| BackendProxy

    BackendProxy <-->|"Vector Lookups & Telemetry Persistence"| DB
    BackendProxy <-->|"Structured Prompting & Multimodal Inference"| Gemini
    BackendProxy <-->|"Server-Side Entitlement Validation"| RC
    Security -.->|"Attestation Verification"| BackendProxy

```

---

## 📦 Level 2: Container Diagram

The Container diagram decomposes the platform boundary into independently deployable units, detailing runtime environments, responsibilities, and communication protocols.

```mermaid
C4Container
title Container Diagram (Level 2) - Pro AI Coaching

Person(user, "Athlete / User", "Logs exercise sets, captures meal photos, and executes AI-guided adaptations.")

Container_Boundary(c1, "Pro AI Coaching Platform Boundaries") {
    Container(mobile_app, "Mobile Client Application", "Flutter, Dart", "Clean Architecture, Riverpod state management, offline-first cache (Hive), native video acceleration (Media3/ExoPlayer).")
    Container(backend_proxy, "Serverless Backend Middleware", "Cloud Functions (Node.js/Python)", "API Gateway, request throttling, App Check attestation, structured prompt assembly, and RevenueCat webhook receiver.")
    ContainerDb(firestore, "Cloud Firestore", "NoSQL Document DB & Vector Store", "Stores user profiles, workout logs, historical volume, and 768-dim embeddings for vector search.")
    ContainerDb(storage, "Cloud Storage", "Object Storage", "Stores user-uploaded meal images and biomechanical analysis video recordings.")
    Container(auth, "Firebase Authentication", "Identity Service", "Token issuance, biometric session refresh, and user credential management.")
}

System_Ext(gemini, "Google Gemini API", "Multimodal LLM", "Processes multimodal inputs, executes nutritional breakdowns, and dynamically adjusts training volume.")
System_Ext(revenuecat, "RevenueCat Infrastructure", "Billing Platform", "Orchestrates Google Play Billing and StoreKit subscription states.")
System_Ext(app_check, "Firebase App Check", "Attestation Provider", "Cryptographically validates client authenticity using Play Integrity and DeviceCheck.")

Rel(user, mobile_app, "Interacts with UI", "Flutter Material 3")
Rel(mobile_app, auth, "Exchanges credentials for JWT", "Firebase Auth SDK / HTTPS")
Rel(mobile_app, firestore, "Syncs workout records & telemetry", "Firestore SDK / gRPC")
Rel(mobile_app, storage, "Uploads raw meal photos & exercise clips", "Cloud Storage SDK / HTTPS")
Rel(mobile_app, revenuecat, "Purchases subscriptions & restores purchases", "Purchases SDK / HTTPS")
Rel(mobile_app, app_check, "Retrieves device attestation token", "App Check SDK")

Rel(mobile_app, backend_proxy, "Dispatches AI analysis requests", "HTTPS / JSON (Bearer JWT + App Check)")
Rel(backend_proxy, firestore, "Executes vector search & logs telemetry", "Firebase Admin SDK")
Rel(backend_proxy, gemini, "Transmits structured context & schemas", "HTTPS / REST API")
Rel(backend_proxy, revenuecat, "Verifies active entitlements", "REST API")
Rel_Back(backend_proxy, revenuecat, "Listens to subscription webhooks", "HTTPS Webhook")

```

### Container Specifications & Network Matrix

| Container / Component | Technology Stack | Network Protocol | Primary Responsibility |
| --- | --- | --- | --- |
| **Mobile Client** | Flutter 3.x, Dart | Native SDK / HTTPS / gRPC | UI rendering, reactive state management, local caching, native media capture. |
| **Backend Middleware** | Cloud Functions Gen 2 | HTTPS / REST (JSON) | API key isolation, rate-limiting, device verification, prompt construction. |
| **Document & Vector DB** | Cloud Firestore | gRPC / Firestore SDK | Telemetry persistence, offline synchronization, native vector indexing (`findNearest`). |
| **Media Storage** | Cloud Storage | HTTPS / Storage SDK | Scalable, access-controlled storage for raw imagery and form capture clips. |
| **Identity Management** | Firebase Auth | HTTPS / SDK | Secure credential handling, OAuth integrations, short-lived JWT generation. |
| **AI Inference Engine** | Google Gemini API | HTTPS / REST | Contextual reasoning, JSON schema enforcement, multimodal computer vision. |
| **Entitlement Service** | RevenueCat | HTTPS / REST | Cross-platform paywall management, Google Play/Apple receipt validation. |

---

## 🧩 Level 3: Component Diagram (Serverless Middleware & RAG Pipeline)

The Component diagram inspects the internal services of the Cloud Functions middleware, detailing how requests are validated, enriched with historical context, and processed by the AI engine.

```mermaid
C4Component
title Component Diagram (Level 3) - Serverless Middleware & RAG Pipeline

Container(mobile_app, "Mobile Client (Flutter)", "Dart, Riverpod", "Dispatches HTTPS requests with workout context & media references.")
ContainerDb(firestore_db, "Cloud Firestore & Vector Store", "NoSQL + Vector Search", "Athlete history, volume metrics, and 768-dim context embeddings.")
System_Ext(gemini_api, "Google Gemini API", "LLM Inference Engine", "Produces context-grounded workout adjustments and macro extractions.")
System_Ext(revenuecat_api, "RevenueCat API", "Billing Validation", "Validates subscription status before unlocking premium inference.")

Container_Boundary(functions_boundary, "Cloud Functions Serverless Middleware") {
    Component(security_interceptor, "App Check & Auth Guard", "Middleware", "Validates Firebase App Check attestation tokens and incoming JWT claims.")
    Component(entitlement_guard, "Entitlement Validator", "Service Module", "Cross-checks active subscription entitlements via RevenueCat before processing.")
    Component(telemetry_router, "Inference Gateway Router", "Express / Cloud Function", "Routes requests to either multimodal meal parsing or hypertrophy coaching pipelines.")
    Component(schema_validator, "Input Schema Validator", "JSON Schema / Zod", "Validates payload types, ranges (RPE 1-10, positive weight), and structures.")
    Component(rag_orchestrator, "Contextual RAG Orchestrator", "Core Logic", "Extracts current session metrics, computes moving volume, and formats prompt.")
    Component(vector_service, "Vector Retrieval Service", "Firestore Admin API", "Performs cosine similarity search over user embedding chunks.")
    Component(gemini_client, "Gemini SDK Client", "Google GenAI SDK", "Executes LLM calls enforcing strict JSON output schemas.")
}

Rel(mobile_app, security_interceptor, "Dispatches AI analysis request", "HTTPS / JSON")
Rel(security_interceptor, entitlement_guard, "Forward authenticated request", "Internal Call")
Rel(entitlement_guard, revenuecat_api, "Verifies entitlement cache / status", "HTTPS REST")
Rel(entitlement_guard, telemetry_router, "Unlocks execution pipeline", "Internal Call")

Rel(telemetry_router, schema_validator, "Validates payload integrity", "Internal Call")
Rel(schema_validator, rag_orchestrator, "Passes sanitized telemetry", "In-Memory")

Rel(rag_orchestrator, vector_service, "Requests historical session similarity", "Async Call")
Rel(vector_service, firestore_db, "Executes findNearest(vector, Cosine, k=3)", "gRPC / Firestore SDK")

Rel(rag_orchestrator, gemini_client, "Injects system instruction + context", "Structured Context")
Rel(gemini_client, gemini_api, "Dispatches prompt with JSON schema constraint", "HTTPS REST")
Rel(gemini_client, firestore_db, "Persists output & updates embedding vector", "gRPC / Firestore SDK")

```

### Component Design Highlights

1. **Gatekeeping via Cryptographic Attestation:** The `App Check & Auth Guard` rejects any request lacking a valid Firebase App Check token before executing backend logic, preventing script-driven denial-of-wallet attacks.
2. **Deterministic Schema Enforcement:** Outputs from the `Gemini SDK Client` use `responseSchema` parameters to ensure that responses strictly match strongly typed JSON representations, eliminating parsing errors in the mobile client.
3. **Targeted Vector Grounding:** Rather than querying an external vector database, the `Vector Retrieval Service` uses Cloud Firestore's native vector search index (`findNearest`), retrieving the top 3 contextual workout or injury notes via cosine similarity in a single database roundtrip.

---

## 🧱 Level 4: Code & Clean Architecture (Mobile Client)

The mobile client is engineered using Clean Architecture and Feature-First package separation, isolating UI components from domain business logic and external infrastructure drivers.

```mermaid
graph TD
    subgraph PresentationLayer ["1. Presentation Layer (UI & State)"]
        UI["Widgets / Screen Pages<br/>(e.g., WorkoutSessionScreen, MealScannerScreen)"]
        Controllers["Riverpod Notifiers & State<br/>(e.g., ActiveWorkoutNotifier, SubscriptionNotifier)"]
    end

    subgraph DomainLayer ["2. Domain Layer (Pure Business Logic)"]
        UseCases["Use Cases<br/>(e.g., CalculateOverloadUseCase, ParseMealMacrosUseCase)"]
        Entities["Domain Entities & Value Objects<br/>(e.g., WorkoutSession, SetTelemetry, Entitlement)"]
        RepoContracts["Repository Interfaces (Abstract Contracts)"]
    end

    subgraph DataLayer ["3. Data Layer (Infrastructure & Persistence)"]
        RepoImpl["Repository Implementations<br/>(e.g., WorkoutRepositoryImpl)"]
        LocalData["Local Data Source<br/>(Hive Cache, SecureStorage)"]
        RemoteData["Remote Data Source<br/>(Firestore SDK, Cloud Functions HTTP, RevenueCat)"]
    end

    UI --> Controllers
    Controllers --> UseCases
    UseCases --> Entities
    UseCases --> RepoContracts
    RepoImpl -.->|Implements| RepoContracts
    RepoImpl --> LocalData
    RepoImpl --> RemoteData

```

### Modular Directory Structure

```text
lib/
├── core/                                # Cross-cutting application infrastructure
│   ├── constants/                       # Route names, asset keys, and design tokens
│   ├── network/                         # Dio client, App Check & Auth interceptors
│   ├── services/                        # Native platform bridges, Media3 video player
│   └── errors/                          # Failure definitions and exception mappers
├── features/                            # Encapsulated feature-first domains
│   ├── workout_tracking/                # Exercise, set, rep, and RPE logging engine
│   │   ├── data/
│   │   │   ├── datasources/             # Local Hive cache & Remote Firestore service
│   │   │   ├── models/                  # Freezed DTOs and JSON serialization models
│   │   │   └── repositories/            # WorkoutRepositoryImpl
│   │   ├── domain/
│   │   │   ├── entities/                # WorkoutSession, SetMetric, FatigueIndex
│   │   │   ├── repositories/            # IWorkoutRepository (Domain contract)
│   │   │   └── usecases/                # SyncSessionUseCase, CalculateProgressiveOverload
│   │   └── presentation/
│   │       ├── controllers/             # ActiveWorkoutNotifier (AsyncNotifierProvider)
│   │       └── widgets/                 # SetInputRow, LiveVolumeChart, RestTimer
│   ├── ai_coaching/                     # Multimodal meal scanner & workout adaptation
│   │   ├── data/                        # Cloud Function endpoints & Gemini DTO mappers
│   │   ├── domain/                      # BiomechanicalFeedback & NutritionalPlan entities
│   │   └── presentation/                # CameraOverlay, AnalysisBottomSheet
│   └── subscription/                    # Monetization, paywalls, and billing state
│       ├── data/                        # RevenueCat Purchases SDK wrapper
│       ├── domain/                      # EntitlementStatus, SubscriptionPackage entities
│       └── presentation/                # PaywallScreen, SubscriptionStatusListener
└── main.dart                            # Dependency injection & Firebase initialization

```

---

## 🗄️ Data Model: Cloud Firestore Schema & Vector Search (ERD)

The database architecture balances real-time UI synchronization with high-throughput set telemetry, storing vector embeddings directly within user contexts.

```mermaid
erDiagram
    USERS ||--o{ WORKOUT_SESSIONS : "owns"
    USERS ||--o{ MEAL_LOGS : "captures"
    USERS ||--o{ EMBEDDING_CONTEXTS : "indexes"
    WORKOUT_SESSIONS ||--o{ WORKOUT_SETS : "contains"
    EXERCISES ||--o{ WORKOUT_SETS : "references"

    USERS {
        string id PK "Firebase Auth UID"
        string email
        string display_name
        string active_entitlement "free | pro_athlete"
        timestamp created_at
        timestamp last_synced_at
    }

    WORKOUT_SESSIONS {
        string id PK "Auto-generated UUID"
        string user_id FK "Ref: USERS.id"
        timestamp start_time
        timestamp end_time
        float total_volume_kg
        int session_rpe "1 - 10"
        string notes
    }

    WORKOUT_SETS {
        string id PK "Composite session_id + index"
        string session_id FK "Ref: WORKOUT_SESSIONS.id"
        string exercise_id FK "Ref: EXERCISES.id"
        int set_order
        int reps
        float weight_kg
        float rpe
        boolean is_failure_set
    }

    EXERCISES {
        string id PK "System Catalog ID"
        string name
        string muscle_group "chest | back | legs | etc"
        string movement_pattern "push | pull | hinge | squat"
    }

    MEAL_LOGS {
        string id PK "Auto-generated UUID"
        string user_id FK "Ref: USERS.id"
        string image_storage_uri
        int estimated_calories
        float protein_grams
        float carbs_grams
        float fat_grams
        timestamp logged_at
    }

    EMBEDDING_CONTEXTS {
        string id PK "Auto-generated UUID"
        string user_id FK "Ref: USERS.id"
        string context_type "fatigue_profile | injury_log | strength_pr"
        string raw_summary_text
        vector_768 context_embedding "Firestore Vector Field (Cosine)"
        timestamp created_at
    }

```

---

## ⚡ Multimodal RAG & Inference Sequence

The following sequence illustrates how telemetry and media are transformed into personalized coaching feedback without leaking backend secrets or exposing ungrounded LLM prompts.

```mermaid
sequenceDiagram
    autonumber
    actor Athlete as 🏋️‍♂️ Athlete
    participant App as 📱 Mobile Client (Flutter)
    participant RC as 💳 RevenueCat SDK
    participant Gateway as ⚡ Cloud Functions Gateway
    participant Firestore as 🔥 Cloud Firestore (Vector Search)
    participant Gemini as 🧠 Google Gemini API

    Athlete->>App: Submits completed workout (Sets, RPE, Fatigue)
    App->>RC: Checks customerInfo.entitlements["pro_access"]
    RC-->>App: Entitlement Verified (Active)
    
    App->>App: Fetches App Check token (Play Integrity / DeviceCheck)
    App->>Gateway: POST /v1/coaching/adapt-workout (JWT + App Check + Telemetry)
    
    Gateway->>Gateway: Validates App Check attestation & Auth UID
    Gateway->>Firestore: Execute vector search: findNearest(recent_fatigue_vector, Cosine, k=3)
    Firestore-->>Gateway: Returns relevant context chunks (injury history, past volume)
    
    Gateway->>Gateway: Assembles dynamic prompt (System instruction + Grounded context + JSON Schema)
    Gateway->>Gemini: POST generateContent (Prompt + Schema constraints)
    Gemini-->>Gateway: Structured JSON response (Volume adjustments, recovery alerts)
    
    Gateway->>Firestore: Writes adaptation record & persists new session embedding
    Gateway-->>App: Returns typed coaching payload (200 OK)
    App->>Athlete: Renders adaptation insights on Dashboard

```

---

## 💳 Subscription & Navigation Lifecycle (RevenueCat)

To prevent navigation lockups and handle post-purchase routing deterministically, the application orchestrates billing events using a centralized reactive listener.

```mermaid
sequenceDiagram
    autonumber
    actor User as 👤 User
    participant Paywall as 🖼️ Paywall View
    participant SubNotifier as 🔄 SubscriptionNotifier (Riverpod)
    participant RCSdk as 📱 Purchases SDK (RevenueCat)
    participant AppRouter as 🧭 GoRouter / Navigation Stack

    User->>Paywall: Selects subscription package & taps purchase
    Paywall->>SubNotifier: executePurchase(Package)
    SubNotifier->>RCSdk: purchasePackage(package)
    
    Note over RCSdk: Native Google Play / App Store sheet handles payment
    RCSdk-->>SubNotifier: Returns CustomerInfo
    
    alt Entitlement Active
        SubNotifier->>SubNotifier: Updates state: SubscriptionState.premiumActive
        SubNotifier->>AppRouter: Navigates to Dashboard (pushReplacementNamed('/dashboard'))
        Paywall-->>User: Transitions smoothly to active dashboard
    else Purchase Cancelled / Pending
        SubNotifier->>SubNotifier: Updates state: SubscriptionState.idle
        Paywall-->>User: Displays non-blocking notification; remains on paywall
    else Entitlement Missing (Webhook Delay)
        SubNotifier->>RCSdk: syncPurchases() / getCustomerInfo()
        RCSdk-->>SubNotifier: Refreshed CustomerInfo
        SubNotifier->>AppRouter: Navigates to Dashboard
    end

```

### Essential Navigation Logic

```dart
Future<void> purchaseSubscription(Package package) async {
  state = const AsyncValue.loading();
  try {
    final customerInfo = await Purchases.purchasePackage(package);
    final isEntitled = customerInfo.entitlements.all['pro_access']?.isActive ?? false;

    if (isEntitled) {
      state = AsyncValue.data(SubscriptionStatus.premium);
      // Ensure context validity before driving the navigation stack
      ref.read(navigationServiceProvider).navigateToDashboard();
    } else {
      state = AsyncValue.data(SubscriptionStatus.free);
    }
  } on PlatformException catch (e) {
    final errorCode = PurchasesErrorHelper.getErrorCode(e);
    if (errorCode != PurchasesErrorCode.purchaseCancelledError) {
      state = AsyncValue.error(e, StackTrace.current);
    } else {
      state = AsyncValue.data(SubscriptionStatus.free);
    }
  }
}

```

---

## 🛡️ Enterprise Security, Reliability & CI/CD Standards

* **Zero-Trust Client Boundary:** All Gemini API interactions and system prompts reside strictly within Cloud Functions Gen 2. No LLM tokens or administrative credentials are compiled into the client binaries.
* **Cryptographic Attestation via App Check:** The backend rejects any request lacking an attestation token generated by Apple DeviceCheck or Google Play Integrity, neutralizing unauthorized scrapers and replay attacks.
* **Type-Safe Dynamic JSON Validation:** Serverless endpoints validate input payloads using explicit schema validators and enforce response structural integrity using Gemini `responseSchema` configurations, preventing client deserialization errors.
* **Offline-First Resilience:** Telemetry entries are written first to local cache via Hive and synchronized asynchronously to Cloud Firestore via persistent gRPC channels, guaranteeing zero data loss in intermittent connectivity scenarios.
* **Automated CI/CD Delivery:** Codemagic pipelines automate static Dart analysis, unit test suites, integration mocks, App Check environment credentialing, and store-compliant App Bundle / IPA code signing across staging and production build flavors.
