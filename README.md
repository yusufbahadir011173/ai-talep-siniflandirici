# AI Talep Sınıflandırıcı

Gelen müşteri mesajlarını kategoriye ayıran, aciliyet belirleyen ve taslak cevap üreten mini AI agent. Claude API ve Google Colab ile geliştirildi.

> **Durum:** Prototip (v3). 5 vakalık test setinde beklenen sonuçları veriyor. Test seti henüz küçük, sınırlar `sonuclar.md` dosyasında açıkça yazılı.

## Problem

Kurumsal firmalarda destek kutusuna gelen mesajlar genellikle elle okunup ilgili ekibe yönlendirilir. Bu yavaştır, kişiden kişiye tutarsızdır ve acil vakalar (para kaybı, hukuki tehdit, iptal tehdidi) sıradaki mesajların arasında kaybolabilir.

## Çözüm

Agent her mesajı okur ve şu yapıda bir çıktı üretir:

```json
{
  "kategori": "sikayet",
  "aciliyet": "yuksek",
  "ozet": "Müşteri çift ücret kesintisi nedeniyle iade talep ediyor.",
  "taslak_cevap": "..."
}
```

| Kategori | Anlamı |
|---|---|
| `sikayet` | Memnuniyetsizlik, gecikme, yanlış ücret, iade talebi |
| `satis` | Fiyat, teklif, ürün bilgisi, satın alma niyeti |
| `destek` | Teknik sorun, kullanım yardımı, hesap sorunu |
| `guvenlik` | Sistem talimatlarını öğrenmeye veya agent'ı yönlendirmeye çalışan mesajlar |
| `diger` | Diğer mesajlar ve talep içermeyen bilgilendirmeler |

## Nasıl çalışır

```
Müşteri mesajı → Sistem prompt'u + Claude → JSON çıktı → (insan onayı) → ilgili ekip
```

1. Mesaj, sistem prompt'uyla birlikte Claude'a gönderilir.
2. Claude yalnızca JSON formatında cevap verir.
3. Kod, JSON'u okuyup kategori, aciliyet, özet ve taslak cevabı ayırır.
4. Taslak cevaplar **otomatik gönderilmez**; bir insanın onayına sunulması önerilir.

## Tasarım kararları

- **Sistem prompt'u ile talimat:** Agent'ın görevi kullanıcı mesajından ayrı, sistem mesajında tanımlı.
- **Prompt injection savunması:** Mesajın içindeki talimatlar uygulanmaz, sadece sınıflandırılır. Bu tür girişimler `guvenlik` kategorisine ayrılır.
- **Söz verme yasağı:** Agent'ın sistemlere erişimi olmadığı için taslak cevaplarda somut süre, çözüm veya iade sözü verilmez.
- **Tanımlı kurallar:** Kategori ve aciliyet için açık tanımlar var. İlk sürümde tanım olmadığında model tutarsız tahmin yaptı (bkz. `sonuclar.md`).
- **Bir seferde bir değişiklik:** Prompt iyileştirmeleri tek tek yapıldı ki hangi değişikliğin neyi etkilediği görülebilsin.

## Repo yapısı

| Dosya | İçerik |
|---|---|
| `README.md` | Proje özeti |
| `prompt.md` | Sistem prompt'u ve sürüm geçmişi (v1 → v3) |
| `test-ornekleri.md` | Test mesajları ve beklenen çıktılar |
| `sonuclar.md` | Sürüm karşılaştırması, bulgular, bilinen sınırlar |
| `siniflandirici.ipynb` | Çalışan Colab notebook'u |

## Nasıl çalıştırılır

1. `siniflandirici.ipynb` dosyasını Google Colab'de aç.
2. Anthropic Console'dan bir API anahtarı al.
3. Colab'de sol menüdeki **Secrets (🔑)** bölümüne `ANTHROPIC_API_KEY` adıyla ekle ve notebook erişimini aç.
4. Hücreleri sırayla çalıştır.

**Güvenlik notu:** API anahtarı hiçbir zaman koda yazılmaz veya repoya yüklenmez. Anahtar yalnızca Colab Secrets'ta durur.

## Sonraki adımlar

- [ ] Test setini 15-20 vakaya çıkarmak (yeni, prompt'un görmediği mesajlarla)
- [ ] `guvenlik` kategorisi için aciliyet kuralı eklemek
- [ ] Taslak cevaplardaki "ilettik" gibi ifadeleri, agent gerçek bir sisteme bağlanana kadar daha temkinli hale getirmek
- [ ] Uzun, çok konulu ve İngilizce mesajlarla test etmek
