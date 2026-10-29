# 🏛️ Pro AI Coaching – Enterprise System Architecture & Engineering Documentation

[![Architecture: C4 Model](https://img.shields.io/badge/Architecture-C4_Model-0B4884?style=flat)](https://c4model.com/)
[![Flutter](https://img.shields.io/badge/Client-Flutter_3.x-02569B?style=flat&logo=flutter&logoColor=white)](https://flutter.dev)
[![Firebase](https://img.shields.io/badge/Backend-Cloud_Functions_Gen2-FFCA28?style=flat&logo=firebase&logoColor=black)](https://firebase.google.com/docs/functions)
[![Firestore](https://img.shields.io/badge/Database-Firestore_Vector_Search-FFCA28?style=flat&logo=firebase&logoColor=black)](https://firebase.google.com/docs/firestore)
[![Google Gemini](https://img.shields.io/badge/AI_Engine-Gemini_Multimodal_API-8E75C2?style=flat&logo=google&logoColor=white)](https://ai.google.dev/)
[![RevenueCat](https://img.shields.io/badge/Billing-RevenueCat_StoreKit-E7525A?style=flat&logo=revenuecat&logoColor=white)](https://www.revenuecat.com/)
[![Security: App Check](https://img.shields.io/badge/Security-Play_Integrity_%7C_DeviceCheck-2EA043?style=flat)](https://firebase.google.com/docs/app-check)
[![CI/CD: Codemagic](https://img.shields.io/badge/CI%2FCD-Codemagic_Pipelines-F4511E?style=flat)](https://codemagic.io/)

---

## 📌 Executive Summary

**Pro AI Coaching** is an enterprise-grade mobile health and fitness platform engineered with Flutter. The system ingests and processes high-frequency workout telemetry (set/rep volume, Rate of Perceived Exertion [RPE], fatigue indexes) alongside multimodal meal imagery to deliver real-time biomechanical feedback and dynamic nutritional adaptations.

The architecture eliminates client-side API key exposure through a secure serverless gateway, executes high-dimensional semantic lookups via Cloud Firestore Vector Search, and isolates cross-platform billing operations using RevenueCat. This document details the end-to-end design across all four levels of the C4 model, alongside data persistence schemas, RAG orchestration flows, and security protocols.

---

## 🏛️ Level 1: System Context Diagram

Visualizes core user interactions, platform boundaries, managed cloud services, and third-party enterprise integrations.

```mermaid
%%{init: {'flowchart': {'nodeSpacing': 45, 'rankSpacing': 65, 'curve': 'basis'}}}%%
flowchart TB
    User(["🏋️‍♂️ Athlete / End User<br/>[Person]"])

    subgraph SystemBoundary ["Pro AI Coaching Platform Ecosystem"]
        App["📱 Pro AI Coaching Mobile Client<br/>[Flutter 3.x - iOS & Android]"]
        BackendProxy["⚡ Cloud Functions Gen 2<br/>[API Gateway & Rate Limiter]"]
        DB[("🔥 Cloud Firestore<br/>[Telemetry & Vector Store]")]
        Storage[("📦 Cloud Storage<br/>[Meal & Form Media]")]
        Auth["🔑 Firebase Auth & Remote Config<br/>[Identity & Dynamic Flags]"]
    end

    subgraph ExternalServices ["External Enterprise Integrations"]
        Gemini["🧠 Google Gemini API<br/>[Multimodal Vision & LLM]"]
        RC["💳 RevenueCat<br/>[StoreKit & Play Billing]"]
        Security["🛡️ Firebase App Check<br/>[Play Integrity & DeviceCheck]"]
    end

    User <-->|"Workouts, Meal Media & AI Feedback"| App

    App <-->|"OAuth JWT & Feature Flags"| Auth
    App <-->|"Offline-First Telemetry Sync"| DB
    App -->|"Encrypted Media Upload"| Storage
    App <-->|"Coaching Requests (JWT + App Check)"| BackendProxy
    App <-->|"Subscriptions & Paywalls"| RC
    App -->|"Client Attestation"| Security

    BackendProxy <-->|"Vector Lookups & Persistence"| DB
    BackendProxy <-->|"Structured Multimodal Inference"| Gemini
    RC -.->|"Billing Webhooks"| BackendProxy
    Security -.->|"Token Verification"| BackendProxy

    classDef person fill:#08427B,stroke:#052E56,stroke-width:2px,color:#ffffff
    classDef client fill:#1168BD,stroke:#0B4884,stroke-width:2px,color:#ffffff
    classDef backend fill:#1B75BC,stroke:#0E4D80,stroke-width:2px,color:#ffffff
    classDef storage fill:#2A6F97,stroke:#184E77,stroke-width:2px,color:#ffffff
    classDef external fill:#5C677D,stroke:#33415C,stroke-width:1.5px,color:#ffffff

    class User person
    class App client
    class BackendProxy,Auth backend
    class DB,Storage storage
    class Gemini,RC,Security external

    style SystemBoundary fill:none,stroke:#6E7681,stroke-width:2px,stroke-dasharray: 5 5
    style ExternalServices fill:none,stroke:#6E7681,stroke-width:1.5px,stroke-dasharray: 5 5
```

---

## 📦 Level 2: Container Diagram

Decomposes the platform boundary into independently deployable containers, detailing runtime environments, core responsibilities, and inter-service network protocols.

```mermaid
%%{init: {'flowchart': {'nodeSpacing': 45, 'rankSpacing': 70, 'curve': 'basis'}}}%%
flowchart TB
    User(["👤 Athlete / End User<br/>[Person]<br/>Logs exercise sets, scans meals,<br/>executes AI-guided adaptations"])

    subgraph PlatformBoundary ["Pro AI Coaching Platform Boundary (C4 Level 2)"]
        MobileApp["📱 Mobile Client Application<br/>[Container: Flutter 3.x, Dart]<br/>Clean Architecture, Riverpod state,<br/>Hive cache, Media3 video engine"]
        Auth["🔑 Firebase Auth & Remote Config<br/>[Container: Identity & Config]<br/>JWT issuance, OAuth sessions,<br/>dynamic paywall & feature flags"]
        Storage[("📦 Cloud Storage<br/>[Container: Object Storage]<br/>Encrypted meal photos &<br/>form analysis video clips")]
        BackendProxy["⚡ Serverless Middleware<br/>[Container: Cloud Functions Gen 2]<br/>API Gateway, App Check guard,<br/>rate limiting & RAG orchestration"]
        Firestore[("🔥 Cloud Firestore<br/>[Container: Document & Vector DB]<br/>Workout telemetry, user profiles &<br/>768-dim cosine embeddings")]
    end

    subgraph ExternalBoundary ["External Enterprise Systems"]
        Gemini["🧠 Google Gemini API<br/>[External: Multimodal LLM]<br/>Macro extraction & context-aware<br/>workout volume adaptation"]
        RevenueCat["💳 RevenueCat Infrastructure<br/>[External: Billing Platform]<br/>Play Billing & StoreKit lifecycle,<br/>entitlements & webhooks"]
        AppCheck["🛡️ Firebase App Check<br/>[External: Attestation Provider]<br/>Play Integrity & DeviceCheck<br/>cryptographic verification"]
    end

    User -->|"Interacts with UI (Material 3)"| MobileApp

    MobileApp -->|"Exchanges OAuth for JWT (HTTPS)"| Auth
    MobileApp -->|"Uploads Raw Media (Storage SDK)"| Storage
    MobileApp <-->|"Syncs Telemetry (gRPC)"| Firestore
    MobileApp <-->|"Coaching Requests (HTTPS JSON)"| BackendProxy
    MobileApp <-->|"Purchases & Entitlements (SDK)"| RevenueCat
    MobileApp -->|"Fetches Integrity Token"| AppCheck

    BackendProxy <-->|"Vector Search & Admin Writes"| Firestore
    BackendProxy <-->|"Structured Prompt & JSON Schema"| Gemini
    BackendProxy -->|"Verifies Active Entitlements"| RevenueCat
    RevenueCat -.->|"Dispatches Billing Webhooks"| BackendProxy

    classDef person fill:#08427B,stroke:#052E56,stroke-width:2px,color:#ffffff
    classDef client fill:#1168BD,stroke:#0B4884,stroke-width:2px,color:#ffffff
    classDef backend fill:#1B75BC,stroke:#0E4D80,stroke-width:2px,color:#ffffff
    classDef storage fill:#2A6F97,stroke:#184E77,stroke-width:2px,color:#ffffff
    classDef external fill:#5C677D,stroke:#33415C,stroke-width:1.5px,color:#ffffff

    class User person
    class MobileApp client
    class BackendProxy,Auth backend
    class Firestore,Storage storage
    class Gemini,RevenueCat,AppCheck external

    style PlatformBoundary fill:none,stroke:#6E7681,stroke-width:2px,stroke-dasharray: 5 5
    style ExternalBoundary fill:none,stroke:#6E7681,stroke-width:1.5px,stroke-dasharray: 5 5
```

### Container Specifications & Network Matrix

| Container / Component | Technology Stack | Network Protocol | Primary Responsibility |
| :--- | :--- | :--- | :--- |
| **Mobile Client** | Flutter 3.x, Dart | Native SDK / HTTPS / gRPC | Cross-platform UI rendering, reactive state management, offline caching, native media capture. |
| **Backend Middleware** | Cloud Functions Gen 2 | HTTPS / REST (JSON) | API key isolation, token rate-limiting, device attestation checks, RAG prompt assembly. |
| **Document & Vector DB**| Cloud Firestore | gRPC / Firestore SDK | Offline-first telemetry synchronization, user profiles, native vector indexing (`findNearest`). |
| **Media Storage** | Cloud Storage | HTTPS / Storage SDK | Access-controlled object storage for nutritional imagery and biomechanical video clips. |
| **Identity & Config** | Firebase Auth & Config | HTTPS / Firebase SDK | OAuth credential handling, short-lived JWT issuance, remote feature and paywall flags. |
| **AI Inference Engine** | Google Gemini API | HTTPS / REST | Multimodal computer vision, contextual reasoning, strict JSON schema output generation. |
| **Entitlement Service** | RevenueCat | HTTPS / REST & Webhooks | Cross-platform paywall orchestration, Google Play Billing and Apple StoreKit validation. |

---

## 🧩 Level 3: Component Diagram (Serverless Middleware & RAG Pipeline)

Inspects the internal modular architecture of the Cloud Functions middleware, illustrating how client requests are attested, validated, enriched with semantic history, and routed to Gemini.

```mermaid
%%{init: {'flowchart': {'nodeSpacing': 45, 'rankSpacing': 60, 'curve': 'basis'}}}%%
flowchart TB
    MobileClient["📱 Mobile Client Application<br/>[Container: Flutter, Riverpod]<br/>Dispatches telemetry context & media URIs"]

    subgraph MiddlewareBoundary ["Cloud Functions Gen 2 Serverless Middleware (C4 Level 3)"]
        SecurityGuard["🛡️ App Check & Auth Guard<br/>[Component: Security Middleware]<br/>Validates Play Integrity / DeviceCheck<br/>tokens & Firebase JWT claims"]
        EntitlementGuard["🎟️ Entitlement Validator<br/>[Component: Billing Guard]<br/>Verifies active pro_access status<br/>before unlocking AI compute"]
        GatewayRouter["🔀 Inference Gateway Router<br/>[Component: Request Router]<br/>Routes traffic to meal vision parsing<br/>or hypertrophy adaptation flows"]
        SchemaValidator["📐 Input Schema Validator<br/>[Component: Validation Layer]<br/>Enforces RPE bounds (1-10), positive<br/>weights & strict payload structure"]
        RagOrchestrator["🧩 Contextual RAG Orchestrator<br/>[Component: Core Pipeline]<br/>Computes rolling volume metrics &<br/>assembles grounded system prompts"]
        VectorService["🔍 Vector Retrieval Service<br/>[Component: Firestore Admin]<br/>Executes k-NN cosine similarity search<br/>across athlete history embeddings"]
        GeminiClient["🤖 Gemini SDK Client<br/>[Component: Google GenAI SDK]<br/>Executes multimodal inference with<br/>strict JSON responseSchema rules"]
    end

    RevenueCatAPI["💳 RevenueCat REST API<br/>[External System]<br/>Server-side entitlement state"]
    FirestoreDB[("🔥 Cloud Firestore & Vector Store<br/>[Container: NoSQL + Vector Index]<br/>Logs & 768-dim context vectors")]
    GeminiAPI["🧠 Google Gemini API<br/>[External System]<br/>Multimodal LLM inference"]

    MobileClient -->|"HTTPS POST (JWT + App Check)"| SecurityGuard
    SecurityGuard -->|"Authenticated Context"| EntitlementGuard
    EntitlementGuard -->|"Verifies Tier (HTTPS)"| RevenueCatAPI
    EntitlementGuard -->|"Unlocked Pipeline"| GatewayRouter
    GatewayRouter -->|"Raw Payload"| SchemaValidator
    SchemaValidator -->|"Sanitized Telemetry"| RagOrchestrator

    RagOrchestrator <-->|"Queries Athlete History"| VectorService
    VectorService <-->|"findNearest (Cosine, k=3)"| FirestoreDB

    RagOrchestrator -->|"Grounded Prompt"| GeminiClient
    GeminiClient <-->|"Schema-Constrained REST"| GeminiAPI
    GeminiClient -->|"Persists Output & Embedding"| FirestoreDB

    classDef client fill:#1168BD,stroke:#0B4884,stroke-width:2px,color:#ffffff
    classDef component fill:#2B6CB0,stroke:#1A4971,stroke-width:1.5px,color:#ffffff
    classDef storage fill:#2A6F97,stroke:#184E77,stroke-width:2px,color:#ffffff
    classDef external fill:#5C677D,stroke:#33415C,stroke-width:1.5px,color:#ffffff

    class MobileClient client
    class SecurityGuard,EntitlementGuard,GatewayRouter,SchemaValidator,RagOrchestrator,VectorService,GeminiClient component
    class FirestoreDB storage
    class RevenueCatAPI,GeminiAPI external

    style MiddlewareBoundary fill:none,stroke:#6E7681,stroke-width:2px,stroke-dasharray: 5 5
```

### Component Design Highlights

1. **Gatekeeping via Cryptographic Attestation:** The `App Check & Auth Guard` rejects requests lacking valid Play Integrity or DeviceCheck tokens at the edge, neutralizing automated scraping and denial-of-wallet attacks before invoking downstream compute.
2. **Deterministic Schema Enforcement:** Calls dispatched through the `Gemini SDK Client` enforce strict `responseSchema` declarations, ensuring LLM outputs conform to strongly typed JSON contracts with zero client parsing failures.
3. **Single-Hop Semantic Grounding:** Rather than maintaining a separate vector cluster, the `Vector Retrieval Service` leverages Cloud Firestore's native `findNearest` cosine index to retrieve the athlete's top 3 relevant fatigue, injury, or volume logs in a single low-latency roundtrip.

---

## 🧱 Level 4: Code & Clean Architecture (Mobile Client)

The Flutter client enforces strict separation of concerns via Clean Architecture and Feature-First modularization, decoupling presentation state from core domain rules and external SDKs.

```mermaid
%%{init: {'flowchart': {'nodeSpacing': 45, 'rankSpacing': 55, 'curve': 'basis'}}}%%
flowchart TB
    subgraph PresentationLayer ["1. Presentation Layer (UI & Reactive State)"]
        UI["🖼️ Screens & Widgets<br/>WorkoutSessionScreen, MealScannerScreen,<br/>PaywallScreen"]
        Controllers["🔄 Riverpod AsyncNotifiers<br/>ActiveWorkoutNotifier, CoachingNotifier,<br/>SubscriptionNotifier"]
    end

    subgraph DomainLayer ["2. Domain Layer (Pure Business Logic)"]
        UseCases["⚙️ Domain Use Cases<br/>CalculateOverloadUseCase,<br/>AnalyzeMealMacrosUseCase"]
        Entities["📦 Immutable Entities (Freezed)<br/>WorkoutSession, SetTelemetry,<br/>EntitlementStatus"]
        RepoContracts["📜 Repository Interfaces<br/>IWorkoutRepository, ICoachingRepository,<br/>ISubscriptionRepository"]
    end

    subgraph DataLayer ["3. Data Layer (Infrastructure & Persistence)"]
        RepoImpl["🔌 Repository Implementations<br/>WorkoutRepositoryImpl,<br/>SubscriptionRepositoryImpl"]
        LocalData["💾 Local Data Sources<br/>Hive Offline Cache,<br/>FlutterSecureStorage"]
        RemoteData["☁️ Remote Data Sources<br/>Firestore SDK, Cloud Functions Client,<br/>RevenueCat Purchases SDK"]
    end

    UI --> Controllers
    Controllers --> UseCases
    UseCases --> Entities
    UseCases --> RepoContracts
    RepoImpl -.->|"Implements"| RepoContracts
    RepoImpl --> LocalData
    RepoImpl --> RemoteData

    classDef presentation fill:#1168BD,stroke:#0B4884,stroke-width:1.5px,color:#ffffff
    classDef domain fill:#1E6091,stroke:#184E77,stroke-width:1.5px,color:#ffffff
    classDef data fill:#2A6F97,stroke:#184E77,stroke-width:1.5px,color:#ffffff

    class UI,Controllers presentation
    class UseCases,Entities,RepoContracts domain
    class RepoImpl,LocalData,RemoteData data

    style PresentationLayer fill:none,stroke:#6E7681,stroke-width:1.5px,stroke-dasharray: 5 5
    style DomainLayer fill:none,stroke:#6E7681,stroke-width:1.5px,stroke-dasharray: 5 5
    style DataLayer fill:none,stroke:#6E7681,stroke-width:1.5px,stroke-dasharray: 5 5
```

### Modular Directory Structure

```text
lib/
├── core/                                # Cross-cutting application infrastructure
│   ├── constants/                       # Route paths, entitlement IDs, design tokens
│   ├── network/                         # Dio HTTP client, App Check & JWT interceptors
│   ├── services/                        # Native bridges, Media3 / ExoPlayer controllers
│   └── errors/                          # Typed Failure classes and exception mappers
├── features/                            # Encapsulated feature-first modules
│   ├── workout_tracking/                # Exercise, set, rep, and RPE logging engine
│   │   ├── data/
│   │   │   ├── datasources/             # Hive local box & Firestore remote service
│   │   │   ├── models/                  # Freezed DTOs and JSON serialization models
│   │   │   └── repositories/            # WorkoutRepositoryImpl
│   │   ├── domain/
│   │   │   ├── entities/                # WorkoutSession, SetMetric, FatigueIndex
│   │   │   ├── repositories/            # IWorkoutRepository (Abstract contract)
│   │   │   └── usecases/                # SyncSessionUseCase, CalculateProgressiveOverload
│   │   └── presentation/
│   │       ├── controllers/             # ActiveWorkoutNotifier (AsyncNotifierProvider)
│   │       └── widgets/                 # SetInputRow, LiveVolumeChart, RestTimer
│   ├── ai_coaching/                     # Multimodal meal scanner & workout adaptation
│   │   ├── data/                        # Cloud Function gateways & Gemini DTO mappers
│   │   ├── domain/                      # BiomechanicalFeedback & NutritionalPlan entities
│   │   └── presentation/                # CameraOverlay, AnalysisBottomSheet
│   └── subscription/                    # Monetization, paywalls, and billing lifecycle
│       ├── data/                        # RevenueCat Purchases SDK service wrapper
│       ├── domain/                      # EntitlementStatus, SubscriptionPackage entities
│       └── presentation/                # PaywallScreen, SubscriptionStatusListener
└── main.dart                            # ProviderScope bootstrap & Firebase initialization
```

---

## 🗄️ Data Model: Cloud Firestore Schema & Vector Search (ERD)

Balances real-time offline-first document synchronization with high-throughput workout set logging and 768-dimensional semantic embeddings.

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
        string active_entitlement "free or pro_access"
        timestamp created_at
        timestamp last_synced_at
    }

    WORKOUT_SESSIONS {
        string id PK "UUID v4"
        string user_id FK "USERS.id"
        timestamp started_at
        timestamp completed_at
        float total_volume_kg
        int session_rpe "Scale 1 to 10"
        string notes
    }

    WORKOUT_SETS {
        string id PK "Composite session and order"
        string session_id FK "WORKOUT_SESSIONS.id"
        string exercise_id FK "EXERCISES.id"
        int set_order
        int reps
        float weight_kg
        float rpe
        boolean is_failure_set
    }

    EXERCISES {
        string id PK "System catalog ID"
        string name
        string muscle_group "chest, back, legs, shoulders"
        string movement_pattern "push, pull, hinge, squat"
    }

    MEAL_LOGS {
        string id PK "UUID v4"
        string user_id FK "USERS.id"
        string image_storage_uri
        int estimated_calories
        float protein_grams
        float carbs_grams
        float fat_grams
        timestamp logged_at
    }

    EMBEDDING_CONTEXTS {
        string id PK "UUID v4"
        string user_id FK "USERS.id"
        string context_type "fatigue, injury, or strength_pr"
        string raw_summary_text
        vector context_embedding "768-dim Firestore Cosine Index"
        timestamp created_at
    }
```

---

## ⚡ Multimodal RAG & Inference Sequence

Illustrates the end-to-end execution flow transforming raw session telemetry into grounded coaching adaptations without exposing backend credentials on the client.

```mermaid
sequenceDiagram
    autonumber
    actor Athlete as 🏋️‍♂️ Athlete
    participant App as 📱 Mobile Client
    participant RC as 💳 RevenueCat SDK
    participant Gateway as ⚡ Cloud Functions
    participant Firestore as 🔥 Cloud Firestore
    participant Gemini as 🧠 Gemini API

    Athlete->>App: Submits completed workout (Sets, RPE, Fatigue)
    App->>RC: Verifies active pro_access entitlement
    RC-->>App: Entitlement confirmed active

    App->>App: Retrieves App Check integrity token
    App->>Gateway: POST /v1/coaching/adapt-workout (JWT + App Check)

    Gateway->>Gateway: Validates App Check token and Auth UID
    Gateway->>Firestore: Runs vector search findNearest(k=3, Cosine)
    Firestore-->>Gateway: Returns top 3 context chunks (injuries, volume)

    Gateway->>Gateway: Builds grounded prompt with JSON responseSchema
    Gateway->>Gemini: POST generateContent (Prompt + Schema)
    Gemini-->>Gateway: Returns structured JSON adaptation payload

    Gateway->>Firestore: Persists adaptation log and new session vector
    Gateway-->>App: 200 OK (Typed coaching payload)
    App-->>Athlete: Renders recovery and volume insights on Dashboard
```

---

## 💳 Subscription & Navigation Lifecycle (RevenueCat)

Prevents post-purchase navigation lockups by coordinating native billing sheets, entitlement verification, and router transitions through a single reactive notifier.

```mermaid
sequenceDiagram
    autonumber
    actor User as 👤 User
    participant Paywall as 🖼️ Paywall View
    participant SubNotifier as 🔄 SubscriptionNotifier
    participant RCSdk as 📱 Purchases SDK
    participant AppRouter as 🧭 GoRouter

    User->>Paywall: Selects subscription plan and taps Subscribe
    Paywall->>SubNotifier: purchaseSubscription(package)
    SubNotifier->>RCSdk: Purchases.purchasePackage(package)

    Note over RCSdk: Google Play or App Store sheet processes payment
    RCSdk-->>SubNotifier: Returns updated CustomerInfo

    alt Entitlement pro_access is Active
        SubNotifier->>SubNotifier: Updates state to SubscriptionStatus.premium
        SubNotifier->>AppRouter: Replaces route stack with /dashboard
        AppRouter-->>User: Transitions directly to Dashboard
    else Purchase Cancelled by User
        SubNotifier->>SubNotifier: Resets state to SubscriptionStatus.free
        Paywall-->>User: Shows cancellation notice and remains on Paywall
    else Webhook or Store Sync Delay
        SubNotifier->>RCSdk: Executes syncPurchases and getCustomerInfo
        RCSdk-->>SubNotifier: Returns refreshed CustomerInfo
        SubNotifier->>AppRouter: Replaces route stack with /dashboard
    end
```

### Post-Purchase Deterministic Routing Snippet

```dart
Future<void> purchaseSubscription(Package package) async {
  state = const AsyncValue.loading();
  try {
    final customerInfo = await Purchases.purchasePackage(package);
    final isEntitled =
        customerInfo.entitlements.all['pro_access']?.isActive ?? false;

    if (isEntitled) {
      state = const AsyncValue.data(SubscriptionStatus.premium);
      ref.read(appRouterProvider).go('/dashboard');
    } else {
      final syncedInfo = await Purchases.getCustomerInfo();
      final isSyncedEntitled =
          syncedInfo.entitlements.all['pro_access']?.isActive ?? false;

      if (isSyncedEntitled) {
        state = const AsyncValue.data(SubscriptionStatus.premium);
        ref.read(appRouterProvider).go('/dashboard');
      } else {
        state = const AsyncValue.data(SubscriptionStatus.free);
      }
    }
  } on PlatformException catch (e) {
    final errorCode = PurchasesErrorHelper.getErrorCode(e);
    if (errorCode == PurchasesErrorCode.purchaseCancelledError) {
      state = const AsyncValue.data(SubscriptionStatus.free);
    } else {
      state = AsyncValue.error(e, StackTrace.current);
    }
  }
}
```

---

## 🛡️ Enterprise Security, Reliability & CI/CD Standards

* **Zero-Trust Client Boundary:** All Gemini API credentials, prompt templates, and system instructions remain isolated inside Cloud Functions Gen 2; zero secret keys are bundled into compiled mobile binaries.
* **Cryptographic Client Attestation:** Backend endpoints enforce Firebase App Check verification via Google Play Integrity (Android) and Apple DeviceCheck / App Attest (iOS) prior to processing requests.
* **Schema-Enforced AI Outputs:** Serverless functions validate incoming telemetry ranges and constrain Gemini responses via `responseSchema`, guaranteeing deterministic JSON deserialization on the mobile client.
* **Offline-First Resilience:** Workout sets and RPE logs write immediately to local Hive storage and synchronize asynchronously to Cloud Firestore over persistent gRPC channels, preventing data loss in low-connectivity gym environments.
* **Automated CI/CD Pipelines:** Codemagic workflows run static analysis (`flutter analyze`), unit tests, environment secret injection, and automated code signing for Google Play Store (AAB) and Apple App Store (IPA) releases.
