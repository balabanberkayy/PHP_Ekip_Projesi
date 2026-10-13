# Yazılım Gereksinim Belgesi (SRS)

**Proje Adı:** Tohumlu Rastgele Zindan Keşif Oyunu (Roguelike)  
**Tarih:** 14.10.2026  
**Belge Sürümü:** 1.0 (Tam Kapsam)

**Grup Üyeleri:**
* **Ahmet Melih Karakaya 2407231054** (Çekirdek Oyun Motoru, Tohumlu Harita & BFS Algoritmaları, PHPUnit Testleri)
* **Berkay Balaban 2407231012** (Veri Erişim Katmanı [JSON/WampServer MySQL Repository], 3NF Veritabanı, Güvenlik & Oturum Yönetimi)
* **Mustafa Arif Gürsel 2407231073** (Arayüz [Bootstrap 5 & CSS Grid/Emoji], AJAX Hamle, Kısmi Yenileme & Raporlama Ekranları)

---

## 1. Giriş ve Proje Tanımı

### 1.1. Amaç
Bu belgenin amacı, PHP 8.2+ kullanılarak herhangi bir hazır uygulama çatısı (framework) kullanılmadan geliştirilecek olan web tabanlı **Tohumlu Rastgele Zindan Keşif Oyunu (Roguelike)** projesinin fonksiyonel gereksinimlerini, kullanım senaryolarını (Use Cases) ve kabul kriterlerini en ince detayına kadar tanımlamaktır. Projenin veritabanı altyapısı için **WampServer içerisinde gelen yerleşik MySQL (MariaDB)** sunucusu kullanılacaktır.

### 1.2. Proje Özeti
Oyuncu kareli (grid tabanlı) ve en az 5 katlı bir zindanda gezer, canavarlarla savaşır, eşyaları toplar ve her katın çıkış merdivenine ulaşmaya çalışır. Haritalar tohum (`seed`) değeriyle rastgele üretilir. Serbest oyun modunun yanı sıra, her gün tüm oyuncuların aynı günlük tohumla üretilen haritada yarıştığı bir **Günlük Yarışma Modu** ve canlı güncellenen **Skor Tablosu** bulunur.

### 1.3. Kapsam Sınırı (Yapılmayacaklar)
* **Resim, ikon veya çizim dosyası hazırlanmaz:** Harita, zindan katları, oyuncu, canavarlar ve eşyalar yalnızca HTML tabloları, CSS Grid ve **Unicode emojiler** (`🧙‍♂️`, `🧱`, `⬜`, `⬛`, `👺`, `💀`, `🐉`, `🧪`, `⚔️`, `🛡️`, `🚪`) kullanılarak gösterilir.
* **Çok oyunculu (multiplayer) mod yoktur:** Oyuncular aynı harita içinde eşzamanlı olarak birbirleriyle etkileşime girmez.
* **Hazır PHP framework'leri kullanılmaz:** Mimari saf PHP 8.2+ ve Composer PSR-4 otomatik yükleme (`autoload`) ile kurulur.

---

## 2. Kullanıcı Rolleri ve Yetki Matrisi

Sistemde **en az 2 rol** (`player` ve `admin`) bulunur.

| Rol Adı | Rol Kodu | Yetkiler ve Erişebildiği İşlevler |
| :--- | :--- | :--- |
| **Oyuncu** | `player` | Kayıt olma, giriş/çıkış yapma, serbest veya günlük yarışma zindan koşusu başlatma, AJAX ile hamle yapma, oyunu kaydetme/devam etme, skor tablosunu görüntüleme. |
| **Yönetici** | `admin` | Tüm `player` yetkilerine ek olarak; kat bazlı başarı/ölüm raporlarını görüntüleme, kural ihlali yapan skorları silme, raporları CSV olarak indirme. |

---

## 3. Fonksiyonel Gereksinimler (FR)

* **FR-01 (Kayıt ve Giriş):** Kullanıcılar `password_hash()` ile özetlenmiş şifrelerle kayıt ve giriş yapabilir.
* **FR-02 (Tohumlu Harita Üretimi):** `mt_srand($seed)` fonksiyonu ile deterministik, herkes için aynı olan en az 5 katlı zindan üretilir. Günlük yarışma modu seçildiğinde `$seed = crc32(date('Y-m-d'))` formülü kullanılır.
* **FR-03 (BFS Erişilebilirlik Garantisi):** `SplQueue` ile Genişlik Öncelikli Arama (BFS) yapılarak başlangıç karesinden çıkış kapısına (`🚪`) yol olduğu %100 doğrulanır. Harita kapalıysa tekrar üretilir.
* **FR-04 (Factory ile Varlık Üretimi):** Canavarlar (`Goblin 👺`, `İskelet 💀`, `Ejderha 🐉`) ve Eşyalar (`İksir 🧪`, `Kılıç ⚔️`, `Kalkan 🛡️`) Factory Pattern ile zorluk seviyesine göre üretilir.
* **FR-05 (Görüş Alanı / Fog of War):** Oyuncuya sadece etrafındaki 2 birimlik görünür alanın verisi gönderilir, görüş yarıçapı dışındaki kareler sunucu tarafında maskelenir (`⬛`). Tarayıcı konsolundan hile yapılamaz.
* **FR-06 (Asenkron Savaş Motoru):** Sayfa yenilenmeden AJAX ile iletilen hamle sonucunda savaş çıkarsa, hasar formülü `Max(1, ATK - DEF)` olarak sunucuda hesaplanır ve sonuç döner.

---

## 4. Fonksiyonel Olmayan Gereksinimler (NFR)

* **NFR-01 (Güvenlik):** Veritabanı (WampServer MySQL) bağlantıları PDO kullanılarak yapılacak, tüm form ve AJAX isteklerinde CSRF (Cross-Site Request Forgery) koruması kullanılacaktır. Parolalar asla düz metin saklanmayacaktır.
* **NFR-02 (Eşzamanlılık ve Performans):** Canlı skor tablosu `parca.php` üzerinden 3 saniyede bir AJAX ile güncellenirken oyuncunun hamle yapmasını yavaşlatmamak (Session Locking) için, işlem anında `session_write_close()` fonksiyonu çağrılacaktır. Yanıt süresi 200ms'nin altında tutulacaktır.
* **NFR-03 (Kullanılabilirlik):** Oyuncu arayüzü Bootstrap 5 ile mobil uyumlu (responsive) olacak ve zindan içindeki hareketler sadece klavye ok tuşları (veya WASD) ile ekstra tıklama gerektirmeden hızlıca yapılabilecektir.

---

## 5. Görsel Use Case (Kullanım Senaryosu) Diyagramı

Aşağıdaki diyagram, sistemdeki aktörlerin hangi senaryolarla etkileşime girdiğini göstermektedir.

```mermaid
flowchart LR
    %% Aktörler
    Player((👤 Oyuncu))
    Admin((🛡️ Yönetici))

    %% Use Caseler
    UC1([UC-01: Kayıt/Giriş Yap])
    UC2([UC-02: Tohumlu Zindan Başlat])
    UC3([UC-03: AJAX Hamle / Savaş])
    UC4([UC-04: Oyunu Kaydet / Çık])
    UC5([UC-05: Canlı Skoru İzle])
    UC6([UC-06: Rapor Al / Hile Sil])

    %% İlişkiler
    Player --> UC1
    Player --> UC2
    Player --> UC3
    Player --> UC4
    Player --> UC5
    
    Admin --> UC1
    Admin --> UC5
    Admin --> UC6