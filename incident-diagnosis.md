Incident Diagnosis: YerimVar



### What happened?
Yoğun kullanım anında (dönem başı ilan patlaması / eşleşme anı) veritabanı bağlantı havuzu (connection pool) tükendi, backend servisleri yanıt veremez hale geldi ve kullanıcılar ilan ekleyemedi ya da ilanları görüntüleyemedi.

### Process Gap
Yük ve Performans Testi (Load/Stress Testing) Eksikliği: Sistemin canlıya alınmadan önce beklenen eşzamanlı (concurrent) kullanıcı yükü altında nasıl tepki vereceği test edilmedi. Ayrıca canlı ortam için bir Otomatik Ölçeklendirme (Auto-scaling) ve Devre Kesici (Circuit Breaker) mekanizması kurgulanmamıştı.

### Missing Evidence
* Yük testi raporları ve stres testi metrikleri (RPS, Latency grafikler).
* Veritabanı sorgu performans kayıtları (Slow Query Logs).
* Sistem seviyesinde bağlantı sınırı (connection limit) ve kaynak kullanım alarm (alerting) kuralları.

---

## Breakpoint 2

### What happened?
Aynı ilan veya işlem üzerinde eşzamanlı istekler yapıldığında (örneğin iki kullanıcının aynı ürünü/koltuğu aynı anda rezerve etmeye çalışması) veri tutarsızlığı (race condition) oluştu ve sistem çift rezervasyon / mükerrer işlem kaydetti.

### Process Gap
**Eşzamanlılık ve İş Mantığı Doğrulama (Concurrency & Integration Testing) Eksikliği:** Kod gözden geçirme (Code Review) aşamasında veri kilitleme (pessimistic/optimistic locking) ve veritabanı seviyesindeki tutarlılık (atomic transactions) kuralları göz ardı edildi. Birim testlerde (unit tests) yalnızca tek kullanıcılı başarılı senaryolar (happy path) test edildi.

### Missing Evidence
* Eşzamanlı istekleri simüle eden entegrasyon test senaryoları (Integration Test Suites).
* Veritabanı seviyesinde `UNIQUE` kısıtlamaları veya kilitlenme (lock) stratejilerini belirten mimari tasarım dokümanı.
* İşlem çakışmalarını izleyen uygulama içi hata (application error) logları.

---

## Breakpoint 3

### What happened?
Production (canlı) ortamda ortaya çıkan kritik bir hatayı çözmek için doğrudan canlı sunucu üzerinde/veritabanında manuel müdahale ve kod güncellemesi yapıldı. Yapılan bu manuel değişiklik durumu daha da kötüleştirerek servisin tamamen kesintiye uğramasına (downtime) neden oldu.

### Process Gap
**Güvenli Yaygınlaştırma ve Otomasyon (CI/CD & Rollback) Süreçlerinin Bulunmaması:** Üretim ortamına yapılan tüm müdahalelerin otomatik bir pipeline (CI/CD) üzerinden geçmesi, staged/canary deployment uygulanması ve olası aksilikte tek tıkla geri alma (rollback) prosedürünün bulunmaması.

### Missing Evidence
* Otomatik yaygınlaştırma ve rollback logları.
* Değişiklik Yönetimi (Change Management) kayıtları ve kod onay mekanizması (Pull Request approvals).
* Canlı ortam sağlık taramaları (Health-check / Readiness probes) ve sistem durum izleme (monitoring) panoları.
