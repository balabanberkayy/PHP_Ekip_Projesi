# Yapay Zekâ Kullanım Günlüğü (`ai-gunlugu.md`)

**Proje:** Tohumlu Rastgele Zindan Keşif Oyunu  
**Aşama:** 1. Hafta (Projelendirme ve Tasarım)  

---

## 1. Hafta: Mimari ve Algoritma Danışmanlık Kayıtları

| Tarih | Danışılan Konu / İstem | Sağlanan Öneri / Çözüm | Karar ve Uygulama |
| :--- | :--- | :--- | :--- |
| **10.10.2026** | "Aşama 2 JSON ve Aşama 3 MySQL geçişini iş katmanında kod değiştirmeden nasıl sağlarız?" | `RunRepositoryInterface` arayüzü tanımlanarak `JsonRunRepository` ve `MySqlRunRepository` sınıflarının bu arayüzden türetilmesi önerildi. | `tasarim.md` belgesine **Repository Sınıf Şeması** eklendi. |
| **12.10.2026** | "Zindan haritasında erişilebilirlik (BFS) kontrolünde hangi yapı daha verimlidir?" | PHP'nin yerleşik `SplQueue` sınıfının FIFO işlemi yaptığı açıklandı. | BFS için `SplQueue` kullanımı standartlaştırıldı. |