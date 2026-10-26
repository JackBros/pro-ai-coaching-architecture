# 🏛️ Pro AI Coaching – Sistem Mimarisi (System Architecture)

[![Flutter](https://img.shields.io/badge/Flutter-02569B?style=flat&logo=flutter&logoColor=white)](https://flutter.dev)
[![Firebase](https://img.shields.io/badge/Firebase-FFCA28?style=flat&logo=firebase&logoColor=black)](https://firebase.google.com)
[![Google Gemini](https://img.shields.io/badge/Gemini_API-8E75C2?style=flat&logo=google&logoColor=white)](https://deepmind.google/technologies/gemini/)
[![RevenueCat](https://img.shields.io/badge/RevenueCat-E7525A?style=flat&logo=revenuecat&logoColor=white)](https://www.revenuecat.com/)
[![Security: App Check](https://img.shields.io/badge/Security-Play_Integrity_%7C_DeviceCheck-brightgreen)](https://firebase.google.com/docs/app-check)

Pro AI Coaching; sporcuların antrenman geçmişini, yorgunluk/RPE metriklerini, görsel öğün analizlerini ve hipertrofi hedeflerini işleyerek salondayken anlık biyomekanik ve beslenme uyarlamaları sunan uçtan uca hibrit bir mobil mimariye sahiptir.

Bu doküman, sistemin istemci mimarisini, bulut servislerini, yapay zekâ katmanını ve güvenlik protokollerini C4 model prensiplerine uygun olarak görselleştirir.

---

## 📌 1. Seviye: Sistem Bağlam Diyagramı (System Context - Level 1)

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
```

---

## 📌 2. Seviye: Kapsayıcı Diyagramı (Container Diagram - Level 2)

Bu diyagram; sistemin yüksek seviyeli teknik mimarisini, sorumluluk alanlarına göre ayrılmış servisleri (istemci, sunucusuz backend, veritabanı, depolama) ve aralarındaki ağ/protokol bağlantılarını modeller.

```mermaid
C4Container
title Container Diyagramı (Level 2) - Pro AI Coaching

Person(user, "Sporcu / Kullanıcı", "Antrenman takibi yapan, yapay zekâ destekli form ve beslenme analizi alan son kullanıcı.")

Container_Boundary(c1, "Pro AI Coaching Platform Sınırları") {
    Container(mobile_app, "Mobil İstemci Uygulaması", "Flutter, Dart", "Clean Architecture, reaktif durum yönetimi, yerel önbellek, video oynatma motoru (Media3/ExoPlayer).")
    Container(backend_proxy, "Cloud Functions Middleware", "Node.js / Cloud Functions", "Güvenli API Gateway. Token kota yönetimi, prompt orkestrasyonu, App Check ve abonelik doğrulama.")
    ContainerDb(firestore, "Cloud Firestore", "NoSQL Document DB", "Kullanıcı profilleri, antrenman geçmişi, set/tekrar kayıtları ve progresif aşırı yüklenme verileri.")
    ContainerDb(storage, "Cloud Storage", "Obje Depolama", "Öğün fotoğrafları, egzersiz form analizi video ve görselleri.")
    Container(auth, "Firebase Authentication", "Auth Servisi", "Kullanıcı kaydı, giriş, oturum ve JWT tabanlı kimlik yönetimi.")
}

System_Ext(gemini, "Google Gemini API / Vertex AI", "Büyük Dil ve Çok Modlu (Multimodal) Vizyon Modeli. Öğün fotoğrafı analizi ve dinamik antrenman revizyonu.")
System_Ext(revenuecat, "RevenueCat & StoreKit", "Apple App Store ve Google Play Billing abonelik/paywall doğrulama ve gelir altyapısı.")
System_Ext(app_check, "Firebase App Check", "Play Integrity API ve Apple DeviceCheck tabanlı istemci bütünlük koruması.")

Rel(user, mobile_app, "Etkileşime girer", "Flutter UI")
Rel(mobile_app, auth, "Giriş yapar, kimlik doğrular", "Firebase Auth SDK / HTTPS")
Rel(mobile_app, firestore, "Antrenman ve profil verilerini senkronize eder", "Firestore SDK / gRPC")
Rel(mobile_app, storage, "Öğün fotoğrafı ve form videosu yükler", "Cloud Storage SDK")
Rel(mobile_app, revenuecat, "Abonelik ve paywall durumunu sorgular", "RevenueCat SDK / HTTPS")
Rel(mobile_app, app_check, "Cihaz bütünlük kanıtı (Attestation token) alır", "Play Integrity / DeviceCheck")

Rel(mobile_app, backend_proxy, "Yapay zekâ analiz ve koçluk istekleri gönderir", "HTTPS / JSON (JWT + App Check)")
Rel(backend_proxy, gemini, "Yapılandırılmış JSON şemalarıyla prompt iletir, çıkarım alır", "HTTPS / REST API")
Rel(backend_proxy, revenuecat, "Aktif abonelik yetkilerini (Entitlement) doğrular", "REST API")
Rel(backend_proxy, firestore, "İşlenmiş koçluk verilerini ve analiz loglarını yazar", "Firebase Admin SDK")
```

### Kapsayıcı ve Protokol Özeti

| Kapsayıcı / Bileşen | Teknoloji | İletişim Protokolü | Görev ve Sorumluluk |
| :--- | :--- | :--- | :--- |
| **Mobil İstemci** | Flutter, Dart | Yerel SDK & HTTPS | Kullanıcı arayüzü, reaktif durum yönetimi ve yerel medya akışı. |
| **Backend Middleware** | Firebase Cloud Functions | HTTPS / REST (JSON) | API anahtarı gizleme, hız sınırlama (rate-limiting) ve prompt orkestrasyonu. |
| **Veritabanı** | Cloud Firestore | gRPC / Firestore SDK | Çevrimdışı destekli antrenman logları, kullanıcı hedefleri ve metrik takibi. |
| **Medya Depolama** | Cloud Storage | HTTPS / Storage SDK | Yemek fotoğrafları ve form analizi videolarının güvenli saklanması. |
| **Yapay Zekâ Motoru**| Gemini API / Vertex AI | HTTPS / REST | Görselden makro çıkarma, anlık egzersiz alternatifi ve koçluk akışı. |
| **Uç Nokta Doğrulama**| Firebase App Check | Play Integrity / DeviceCheck | Sahte istemcileri, emülatörleri ve yetkisiz API sızıntılarını engelleme. |
