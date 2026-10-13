# Yazılım Tasarım Belgesi (SDD)

**Proje Adı:** Tohumlu Rastgele Zindan Keşif Oyunu (Roguelike)  
**Tarih:** 14.10.2026

---

## 1. Mimari Tasarım (4 Katmanlı Yapı)
Oyun, herhangi bir framework kullanılmadan **PHP 8.2+** ve **Composer (PSR-4)** standardı ile oluşturulmuştur.
1. **Sunum Katmanı (Public & View):** `index.php` (Front Controller) ve `parca.php` (AJAX Uç Noktası). Sadece Bootstrap 5, CSS Grid ve Unicode Emojilerle render yapılır.
2. **Kontrolcü Katmanı (Controller):** Gelen HTTP isteklerini (`GameController`, `AuthController`) karşılar, CSRF kontrollerini yapar ve servisleri çağırır.
3. **İş Mantığı Katmanı (Service):** Zindan üretim algoritması (`mt_srand`), Fog of War filtrelemesi, varlık üretimi (Factory Pattern) ve BFS yol bulma mantığı burada çalışır.
4. **Veri Erişim Katmanı (Repository):** WampServer MySQL veritabanı işlemleri (PDO) ve 3NF yapısı bu katmanda soyutlanarak yürütülür.

---

## 2. Çekirdek Algoritmalar

### 2.1. Tohumlu Harita ve Determinizm
Aynı tohumla oynayan herkes aynı haritayı görür. Günlük yarışmada tohum: `$seed = crc32(date('Y-m-d'));`. Her zindan katı için PHP'nin `mt_srand($seed + $floor)` fonksiyonu çağrılarak determinizm sağlanır.

### 2.2. Eşzamanlılık ve Session Kilidi Çözümü
Oyun sırasında sağ paneldeki skor tablosunun AJAX ile her 3 saniyede bir güncellenmesi gerekir. PHP'nin varsayılan `Session` mekanizması diğer istekleri bloklar (Session Locking). Çözüm olarak, `parca.php` dosyasında `session_start()` işleminden hemen sonra **`session_write_close()`** komutu çağrılacaktır.

---

## 3. Algoritma Diyagramı: BFS (Erişilebilirlik Kontrolü)
Zindan rastgele üretilirken, çıkış kapısının kapalı kalmasını önleyen BFS algoritmasının akışı:

```mermaid
flowchart TD
    A[Zindan Katı Üretildi] --> B[Başlangıç Karesini SplQueue'ye Ekle]
    B --> C{Kuyruk Boş mu?}
    C -- Evet --> D[HATA: Çıkışa Yol Yok! Haritayı Yeniden Üret]
    C -- Hayır --> E[Kuyruktan Çıkar, Komşuları İncele]
    E --> F{Komşu == Çıkış Kapısı 🚪?}
    F -- Evet --> G[BAŞARILI: Harita Onaylandı]
    F -- Hayır --> H{Geçilebilir mi?}
    H -- Evet --> I[Kuyruğa Ekle ve İşaretle]
    H -- Hayır --> C
    I --> C