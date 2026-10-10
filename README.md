# TeamZero
# TEKNİK ŞARTNAME VE SİSTEM GENELGESİ: "ANTI-VIBE ENGINE (AVX-DEFENSE)"

**Belge Kodu:** SEC-ENG-2026-AVX-SPEC-FINAL

**Sürüm:** 3.0 (Endüstriyel DevSecOps, Ofansif Doğrulama & Full-Stack Saha Standardı)

**Kapsam:** Yapay Zeka Destekli Üretilen Kodların Statik, Semantik, Leke (Taint) ve İş Mantığı Düzeyinde Güvenlik ve Kalite Denetimi; Küresel Standartlara Uyumluluk Doğrulaması

---

## 1. Amaç ve Kapsam

Büyük Dil Modelleri (LLM) ve otonom kodlama ajanları (Claude Code, Cursor, Copilot vb.) hız kazandırsa da; yüzeyde hatasız görünen fakat arka planda **Broken Object Level Authorization (BOLA/IDOR), mimari çürüme, istisna yutma, güvensiz yerel veri saklama ve halüsinasyon bağımlılıkları** barındıran kod blokları üretmektedir.

Bu şartname; geliştirilen yazılımın hem **Full-Stack/Mobil Mühendislik** pratiklerine (temiz mimari, bellek optimizasyonu, çökme direnci) hem de **Kıdemli Penetrasyon Testi (Pentest)** saha standartlarına (**OWASP Top 10, OWASP Mobile Top 10, OWASP API Top 10, CWE Top 25, NIST SP 800-218 SSDF v1.1**) tam uyumlu olduğunu doğrulamak üzere geliştirilecek **Anti-Vibe Engine (AVX)** denetim motorunun bağlayıcı teknik isterlerini tanımlar.

---

## 2. Çekirdek Sistem Mimarisi ve Fonksiyonel Modüller

Sistem; geliştirici ortamında (`pre-commit`), derleme hattında (CI/CD) ve PR denetim aşamalarında tekil bir binary veya mikro-container olarak çalışacak 6 ana modülden oluşur.

```
[Kaynak Kod / Staged Diff]
         │
         ├──► Modül 1: Deterministik AST & Mimari Bütünlük (Anti-Spagetti)
         ├──► Modül 2: Leke Analizi (Taint Engine) & BOLA/IDOR Denetimi
         ├──► Modül 3: Mobil İstemci & İletişim Güvenliği (OWASP Mobile)
         ├──► Modül 4: Bağımlılık & Halüsinasyon Avcısı (SCA & Supply Chain)
         ├──► Modül 5: Kriptografik Hijyen & Secret Taraması
         └──► Modül 6: Otonom Mantık ve Durum Geçiş Denetimi (State Guard)
         │
         ▼
[GESI Skorlama Motoru] ──► [SARIF / SBOM Raporu] ──► [Otomatik .patch Üretimi]

```

---

### Modül 1: Deterministik AST ve Mimari Bütünlük (Anti-Drift)

* **Katman İhlali Engelleme:** Sunum/UI katmanından doğrudan veritabanı, ORM veya yetkilendirilmemiş ağ sürücüsü çağrılarını bağımlılık grafiği (Dependency Graph) çıkararak engeller.
* **Dead Code ve Bloat Filtresi:** Yapay zekanın bağlam üretirken bıraktığı, projede hiçbir referansı bulunmayan fonksiyon, arayüz ve ölü döngüleri tespit eder.
* **Sessiz İstisna Blokları (Silent Failures):** Boş `catch` veya `except: pass` bloklarını, loglanmamış hata yönetimlerini derleme kırıcı hata (`pipeline breaker`) olarak işaretler.

### Modül 2: Gelişmiş Leke Analizi (Taint Engine) ve BOLA / IDOR Tespiti

* **Veri Akış Grafı (Data Flow Graph - DFG):** Dış dünyadan (HTTP request, parametre, query, deep link, IPC) gelen girdileri "leke" (taint/source) olarak işaretler; bu verinin aradaki bir yetkilendirme (Auth/RBAC) fonksiyonundan geçmeden doğrudan veri tabanı sorgusuna veya nesneye eriştiği (sink) noktaları yakalar.
* **BOLA / IDOR Denetimi:** Kullanıcı kimliği (`userId`, `accountId`) alan rotaların/servislerin nesne düzeyinde erişim doğrulama kontrolü (`session.user.id == target.ownerId`) barındırmadığı durumları bayraklar (CWE-639 / OWASP API1:2023).

### Modül 3: Mobil İstemci ve İletişim Güvenliği (OWASP Mobile Top 10)

* **İstemci Tarafı Güvensiz Depolama (Insecure Storage):** `SharedPreferences`, `UserDefaults` veya yerel SQLite veritabanına şifrelenmemiş hassas veri (token, kimlik bilgisi, parola) yazılmasını engeller (CWE-312).
* **Ağ ve Taşıma Güvenliği:** Açık HTTP protokol çağrılarını, devre dışı bırakılmış SSL/TLS sertifika kontrollerini (`trustAllCerts`, `allowInvalidCertificates`) ve zayıf TLS yapılandırmalarını engeller.
* **Açık IPC ve Deep Link Filtreleme:** Export edilmiş Android `Activity`/`Service` ve intent-filter bileşenlerinin yetkisiz çağrılara açık olup olmadığını doğrular.

### Modül 4: Yazılım Tedarik Zinciri ve Halüsinasyon Avcısı (SCA)

* **Paket Halüsinasyonu Doğrulaması:** LLM'in uydurduğu sahte veya typo-squatting riski taşıyan kütüphane isimlerini resmi paket kayıt defterleri (`npm`, `PyPI`, `Pub.dev`, `Crates.io`) üzerinden gerçek zamanlı doğrular.
* **SBOM ve CVE Entegrasyonu:** CycloneDX v1.5 standardında Software Bill of Materials üretir; bağımlılıkları bilinen CVE veritabanlarıyla eşleştirip kritik açığı bulunan paketleri işaretler.

### Modül 5: Kriptografik Hijyen ve Hassas Veri Taraması

* **Entropi Tabanlı Secret Tespiti:** Koda gömülü API anahtarları, özel anahtarlar (RSA, Ed25519), AWS/GCP credential'ları yüksek entropi ve regex motoruyla yakalar.
* **Güvensiz Kripto Engelleme:** Kriptografik işlemlerde kullanılan zayıf algoritmaları (MD5, SHA1, DES, ECB modlu AES) ve güvensiz sözde-rastgele sayı üreteçlerini (`Math.random()`, `rand()`) engeller.

### Modül 6: Otonom Mantık ve Durum Geçiş Denetimi (State Guard)

* **Önkoşul (Guard Clause) Denetimi:** Kritik finansal/işlem fonksiyonlarının (ödeme onaylama, sipariş tamamlama, rol güncelleme) çağrılmadan önce zorunlu durum bayraklarını (`isPaid`, `isVerified`, `hasPermission`) denetleyip denetlemediğini inceler.

---

## 3. Performans, Kalite ve Güvenlik Metrikleri

Motor, analiz edilen kod tabanına veya diff akışına **Küresel Mühendislik ve Güvenlik Endeksi (GESI)** atar:

$$\text{GESI} = 100 - \left( 0.35 \cdot S_{\text{vuln}} + 0.25 \cdot T_{\text{taint}} + 0.20 \cdot M_{\text{arch}} + 0.20 \cdot Q_{\text{debt}} \right)$$

* $S_{\text{vuln}}$: CVSS v3.1 tabanlı statik zafiyet ağırlığı.
* $T_{\text{taint}}$: Leke analizinde doğrulanmamış veri akışı ve BOLA risk puanı.
* $M_{\text{arch}}$: Katman ihlali ve mimari sapma oranı.
* $Q_{\text{debt}}$: Karmaşıklık aşımı, bellek sızıntısı ve test eksikliği puanı.

| Değerlendirme Parametresi | İdeal / Kabul Edilebilir | Kritik Sınır (İşlemi Bloke Eder) |
| --- | --- | --- |
| **CVSS v3.1 Skoru (Herhangi bir bulgu)** | $\le 3.9$ (Low) | $\ge 7.0$ (High/Critical - 0 Tolerans) |
| **BOLA / Yetkisiz Veri Akışı (Taint)** | $0$ Bulgu | $\ge 1$ (0 Tolerans - Derleme Kırılır) |
| **Hardcoded Secret / API Token** | $0$ Bulgu | $\ge 1$ (0 Tolerans - Commit Engellenir) |
| **Doğrulanmamış / Sahte Paket** | $0$ Bulgu | $\ge 1$ (0 Tolerans) |
| **Cyclomatic Complexity (Fonksiyon Başı)** | $\le 10$ | $> 15$ |
| **Cognitive Complexity (Fonksiyon Başı)** | $\le 12$ | $> 18$ |
| **Sessiz Hata Yakalama (Empty Catch)** | $0$ Bulgu | $\ge 1$ (Bloke) |
| **Genel GESI Skoru** | $\ge 85 / 100$ | $< 70 / 100$ |

---

## 4. Sistem Optimizasyonu ve Operasyonel Kısıtlar

* **Gecikme (Latency SLA):** Git `pre-commit` aşamasında staged diff analizi **$\le 1.5$ saniyede** tamamlanmalıdır. 100.000 satırlık tam depo analizi CI/CD ortamında **$\le 45$ saniyeyi** geçemez.
* **Bağımsız Çalışabilirlik (Zero-Dependency):** Motor harici runtime (Node, JVM, Python) gerektirmeden tek bir derlenmiş statik binary (Go veya Rust) veya ultra-hafif Docker imajı ($< 60\text{ MB}$) olarak dağıtılmalıdır.
* **Kaynak Kullanımı:** Analiz esnasında maksimum bellek kullanımı 256 MB ile sınırlandırılmalıdır.

---

## 5. Çıktı Standardı ve Kendi Kendini Onarma (Self-Healing)

Motor, analizi tamamladığında iki resmi standart çıktı üretmekle yükümlüdür:

1. **SARIF (v2.1.0):** GitHub Advanced Security, GitLab ve SonarQube uyumlu yapılandırılmış zafiyet raporu.
2. **Otomatik Onarım Yaması (`.patch`):** Tespit edilen kural ihlalleri için doğrudan `git apply` ile entegre edilebilecek temiz kod düzeltme dosyası.

### Örnek Terminal Çıktısı (TUI / CLI)

```text
[ANTI-VIBE ENGINE (AVX) - ENDÜSTRİYEL SAHA DENETİMİ]
─────────────────────────────────────────────────────────────────────────────
Hedef: git@github.com:core/app.git | Dal: feature/checkout | Commit: a3f82c
Standartlar: OWASP Top 10, OWASP Mobile, OWASP API (BOLA), NIST SSDF v1.1
─────────────────────────────────────────────────────────────────────────────
Küresel Mühendislik & Güvenlik Endeksi (GESI): 38/100  [DURUM: KESİN RED]

KRİTİK GÜVENLİK BULGULARI:
├── [BOLA / IDOR] Yetkilendirilmemiş Veri Akışı
│   └── src/controllers/order.controller.ts:48
│       Leke: 'req.params.orderId' yetki middleware'i olmaksızın doğrudan DB'ye aktarıldı.
├── [OWASP M2] Güvensiz Yerel Veri Saklama
│   └── lib/services/storage_service.dart:22
│       Bulgu: 'jwt_token' SharedPreferences içerisine şifresiz düz metin yazılıyor.
├── [TEDARİK ZİNCİRİ] Paket Halüsinasyonu Tespit Edildi
│   └── pubspec.yaml:19
│       Kütüphane: 'flutter_secure_crypto_helper' resmi pub.dev kayıtlarında mevcut değil!

KOD KALİTESİ VE MİMARİ SAPMA:
├── [MİMARİ] UI Katmanından Veritabanı Çağrısı: src/views/profile_view.dart:88
├── [KALİTE] Sessiz Hata Bloklama (Empty Catch): 3 adet tespit edildi.
└── [KARMAŞIKLIK] Cognitive Complexity: 26 (Eşik değeri: 12)
─────────────────────────────────────────────────────────────────────────────
SONUÇ: Dağıtım/Commit engellendi.
ONARIM YAMASI HAZIRLANDI: ./patches/remediation_a3f82c.patch ('git apply' ile uygulayın)

```

---

## 6. Kabul ve Doğrulama Protokolü (Doğrulama Senaryoları)

Bu şartnameye göre geliştirilen motor, aşağıdaki üç sahne testinden başarıyla geçtiğinde **"Saha Onaylı"** mührü alır:

1. **Full-Stack Mobil Dayanıklılık Testi:** Yapay zeka tarafından üretilmiş, içinde UI katmanından doğrudan SQL/Firestore çağrısı yapılan, boş `catch` blokları içeren ve sahte bir kütüphane talep eden mobil projeyi derleme anında durdurmalı ve mimari ihlalleri listelemelidir.
2. **Pentester BOLA/Taint Testi:** Parametrik SQL kullanılmış olsa dahi, kullanıcı nesne ID'sini oturum token'ı ile doğrulamadan veri tabanına ileten endpoint'i veri akış grafiğiyle (DFG) yakalayıp süreci kırmalıdır.
3. **Sıfır Hata Toleransı & Patch Doğrulaması:** Tespit ettiği SQL parametrelendirme ve boş catch blokları için ürettiği `.patch` dosyası uygulandığında projenin derlenebilirliğini bozmamalı ve GESI skorunu anında 85'in üzerine çıkarmalıdır.
