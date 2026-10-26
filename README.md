# 🏛️ Pro AI Coaching – Sistem Mimarisi (System Architecture)

[![Flutter](https://img.shields.io/badge/Flutter-02569B?style=flat&logo=flutter&logoColor=white)](https://flutter.dev)
[![Firebase](https://img.shields.io/badge/Firebase-FFCA28?style=flat&logo=firebase&logoColor=black)](https://firebase.google.com)
[![Google Gemini](https://img.shields.io/badge/Gemini_API-8E75C2?style=flat&logo=google&logoColor=white)](https://deepmind.google/technologies/gemini/)
[![RevenueCat](https://img.shields.io/badge/RevenueCat-E7525A?style=flat&logo=revenuecat&logoColor=white)](https://www.revenuecat.com/)
[![Security: App Check](https://img.shields.io/badge/Security-Play_Integrity_%7C_DeviceCheck-brightgreen)](https://firebase.google.com/docs/app-check)

Pro AI Coaching; sporcuların antrenman geçmişini, yorgunluk/RPE metriklerini, görsel öğün analizlerini ve hipertrofi hedeflerini işleyerek salondayken anlık biyomekanik ve beslenme uyarlamaları sunan uçtan uca hibrit bir mobil mimariye sahiptir.

Bu doküman, sistemin istemci mimarisini, bulut servislerini, yapay zekâ katmanını ve güvenlik protokollerini C4 model prensiplerine uygun olarak görselleştirir.

---

## 📌 1. Seviye: Sistem Bağlam Diyagramı (System Context)

Kullanıcının sistemle olan etkileşimini, platform sınırlarını ve dış servis entegrasyonlarını özetler.

```mermaid
flowchart TB
    User(["🏋️‍♂️ Sporcu / Son Kullanıcı"])

    subgraph SystemBoundary ["Pro AI Coaching Ekosistemi"]
        App["📱 Pro AI Coaching Mobil Uygulaması<br/>(Flutter - iOS & Android)"]
    end

    subgraph CloudInfra ["Google Cloud & Firebase Altyapısı"]
        BackendProxy["⚡ Cloud Functions Middleware<br/>(API Gateway & Rate Limiter)"]
        DB[("🔥 Cloud Firestore<br/>(Kullanıcı & Antrenman Verileri)")]
        Storage[("📦 Cloud Storage<br/>(Yemek & Form Görselleri/Videoları)")]
        Auth["🔑 Firebase Authentication"]
    end

    subgraph ExternalServices ["Dış Entegrasyonlar & API'ler"]
        Gemini["🧠 Google Gemini API / Vertex AI<br/>(LLM & Multimodal Vision)"]
        RC["💳 RevenueCat & StoreKit / Play Billing<br/>(Abonelik & Paywall Yönetimi)"]
        Security["🛡️ Firebase App Check<br/>(Play Integrity & DeviceCheck)"]
    end

    User <-->|"Girdi (Ses, Metin, Kamera)<br/>Geri Bildirim (Antrenman, Makro)"| App

    App <-->|"Kimlik Doğrulama"| Auth
    App <-->|"Canlı Veri Senkronizasyonu"| DB
    App -->|"Medya Yükleme"| Storage
    App <-->|"Güvenli API İstekleri (JWT, App Check Token)"| BackendProxy
    App <-->|"Abonelik Doğrulama & Satın Alma"| RC
    App -->|"İstemci Bütünlük Onayı"| Security

    BackendProxy <-->|"Structured Prompting & JSON Output"| Gemini
    Security -.->|"Cihaz Doğrulama Sağlayıcısı"| BackendProxy

---

## 2. Seviye: Kapsayıcı Diyagramı (Container Diagram – Level 2)

Bu diyagram; sistemin yüksek seviyeli teknik mimarisini, sorumluluk alanlarına göre ayrılmış konteynerlerini (istemci, sunucu katmanı, veritabanı) ve bu parçaların birbirleriyle hangi protokoller üzerinden haberleştiğini gösterir.

```mermaid
flowchart TB
    User(["🏋️‍♂️ Sporcu / Mobil Kullanıcı"])

    subgraph ClientApp ["Mobil İstemci (Mobile Client)"]
        Flutter["📱 Flutter Mobil Uygulaması<br/><b>[Flutter & Dart]</b><br/>• Reaktif UI & BLoC Durum Yönetimi<br/>• Ses Kaydı & Canlı Form Kamerası<br/>• Çevrimdışı Önbellek (Hive / SQLite)"]
    end

    subgraph BackendGateway ["Uygulama & API Katmanı (Backend Services)"]
        FastAPI["⚡ Core Backend & API Gateway<br/><b>[Python / FastAPI]</b><br/>• REST API & SSE (Server-Sent Events) Akış Motoru<br/>• Rate Limiter & Token Throttling<br/>• Prompt Orchestrator & Semantik Arama Katmanı<br/>• Pydantic Tabanlı Yapılandırılmış JSON Doğrulayıcı"]
    end

    subgraph DataTier ["Veri & Bellek Katmanı (Data Tier)"]
        Postgres[("🗄️ İlişkisel & Vektör Veritabanı<br/><b>[PostgreSQL + pgvector]</b><br/>• Kullanıcı Profilleri & Antrenman Geçmişi<br/>• Egzersiz ve Hipertrofi Vektör Gömümleri (Embeddings)<br/>• Semantik Önbellek (Semantic Cache)")]
        RedisCache[("⚡ Önbellek & Kuyruk (In-Memory)<br/><b>[Redis]</b><br/>• Oturum & Rate-Limit Kotaları<br/>• Hızlı Yanıt Önbellekleme")]
    end

    subgraph ExternalEcosystem ["Harici Servisler & Güvenlik Sağlayıcıları"]
        Auth["🔑 Firebase Authentication<br/><b>[OAuth / JWT]</b>"]
        Storage["📦 Firebase Cloud Storage<br/><b>[Blob / Medya Deposu]</b>"]
        AppCheck["🛡️ Firebase App Check<br/><b>[Play Integrity / DeviceCheck]</b>"]
        Gemini["🧠 Google Gemini 1.5 API<br/><b>[LLM & Vision Modelleri]</b>"]
        RevenueCat["💳 RevenueCat & Mağaza Faturalandırma<br/><b>[StoreKit / Google Play Billing]</b>"]
    end

    %% Kullanıcı - İstemci
    User -->|"Dokunma, Ses ve Kamera Girdisi"| Flutter

    %% İstemci - Güvenlik & Harici
    Flutter -->|"1. Kimlik Doğrulama (JWT)"| Auth
    Flutter -->|"2. Cihaz Bütünlük Onayı (App Check Token)"| AppCheck
    Flutter -->|"3. Video/Görsel Yükleme (Form & Yemek)"| Storage
    Flutter -->|"4. Abonelik Satın Alma & Paywall"| RevenueCat

    %% İstemci - Backend İletişimi
    Flutter <-->|"HTTPS / REST (JSON)<br/>+ SSE (Kelime Akışı / Streaming)<br/>[Bearer Token + App Check Header]"| FastAPI

    %% Backend - Veritabanı
    FastAPI <-->|"AsyncPG / SQLAlchemy (TCP :5432)<br/>CRUD & Kosinüs Benzerlik Sorguları"| Postgres
    FastAPI <-->|"Redis Protocol (TCP :6379)<br/>Token & Limit Kontrolü"| RedisCache

    %% Backend - Dış Entegrasyonlar
    FastAPI -->|"Webhook / Entitlement Doğrulama"| RevenueCat
    FastAPI <-->|"HTTPS / gRPC<br/>Yapılandırılmış İstemler (Structured Prompts)"| Gemini
