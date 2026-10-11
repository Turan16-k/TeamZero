Bu turda eklenenler (Faz 2)
M2 — Taint/BOLA (CWE-639): sahiplik kontrolü olmadan kullanıcı-kontrollü id'nin DB sorgusuna akışını yakalıyor. SPEC §6.2 kabul senaryosu kanıtlandı — güvensiz getOrder bayraklanıyor, req.user.id ile sahiplik doğrulayan güvenli varyant bayraklanmıyor.
M6 — State Guard: completeOrder/checkout/updateRole gibi kritik fonksiyonların isPaid/isVerified/hasPermission önkoşulunu denetliyor.
M3 — Mobil (OWASP Mobile): güvensiz yerel depolama (SharedPreferences'a jwt_token), açık HTTP, TLS doğrulaması kapalı, export edilmiş Android bileşeni.
SBOM (CycloneDX 1.5): --sbom ile purl'li bileşen listesi.
Konfigürasyon: .avx.json (minGesi/disabledRules/allowPackages), satır-içi avx:ignore bastırma, ve hooks/pre-commit.
Birim test paketi: 9 paket yeşil (go test ./...).
