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
