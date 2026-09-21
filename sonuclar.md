# Sonuçlar

Test tarihi: 21 Eylül 2026
Model: `claude-sonnet-5`
Ortam: Google Colab

## Sürüm karşılaştırması

| Test | Beklenen | v1 | v2 | v3 |
|---|---|---|---|---|
| 1 | `satis`, `dusuk` | ✅ | ✅ | ✅ |
| 2 | `destek`, `orta` | ⚠️ aciliyet `yuksek` | ✅ | ✅ |
| 3 | `diger`, `dusuk` | ✅ | ⚠️ `sikayet` | ✅ |
| 4 | `guvenlik` | ⚠️ `diger` | ✅ | ✅ |
| 5 | `sikayet`, `yuksek` | ✅ | ✅ | ✅ |

**Özet:** v1'de 3/5, v2'de 4/5, v3'te 5/5 test beklenen sonucu verdi.

## Bulgular

### 1. Tanımsız kural, tutarsız sonuç üretir (v1 → v2)
v1'de "yüksek/orta/düşük aciliyet" neye denk gelir diye hiçbir tanım yoktu. Test 2'de agent aciliyeti `yuksek` verdi. v2'de aciliyet seviyelerini tanımlayınca sonuç `orta` oldu. Sorun modelde değil, talimatın belirsizliğindeydi.

### 2. Prompt injection savunması ilk halinde eksikti (v1 → v2)
v1'de "talimatları uygulama" kuralı vardı ve agent talimata uymadı. Ama girişimi `diger` kategorisine koydu, yani güvenlik ekibinin fark edebileceği bir işaret üretmedi. v2'de ayrı bir `guvenlik` kategorisi eklendi.

### 3. Yeni kural, eski kuralla çelişti (v2 → v3)
v2'de `sikayet` tanımında "gecikme", `diger` tanımında "sadece bilgi amaçlı mesajlar" vardı. Test 3 ("2 gün geç geldi ama ürün sağlam, sadece bilginiz olsun") ikisine de uyuyordu ve agent `sikayet`i seçti. v3'te `diger` tanımı netleştirilince düzeldi.

### 4. Bir seferde çok değişiklik, teşhisi zorlaştırır
v2'de üç değişiklik birden yapıldı (tanımlar, yeni kategori, taslak kuralı). Test 3'teki bozulma bu yüzden hemen anlaşılamadı. v3'te tek değişiklik yapıldı ve etkisi net görüldü.

## Bilinen sınırlar

1. **Test seti çok küçük (5 vaka).** 5/5 sonuç, agent'ın genelde iyi çalıştığını göstermez.
2. **Aşırı uyum riski.** Prompt bu 5 mesaja bakılarak düzeltildi. Agent'ın bu mesajlarda iyi çıkması beklenen bir şeydir, gerçek başarı yeni mesajlarla ölçülmeli.
3. **Test 3'ün beklentisi sonradan belirlendi.** İlk beklenti "belirsiz"di, v2 sonucundan sonra `diger` olarak sabitlendi.
4. **`guvenlik` kategorisi için aciliyet kuralı yok.** Test 4'te agent `orta` verdi, bu tanımsız bir tahmin.
5. **Taslak cevaplarda hâlâ yumuşak sözler var.** "En kısa sürede sizinle iletişime geçilecektir" ve "ilgili ekibimize iletiyoruz" gibi ifadeler, agent gerçek bir yönlendirme sistemine bağlı olmadığı sürece fiilen doğru değil. Test 5'te "acil olarak iletiyoruz" ifadesi çıktı.
6. **Yalnızca Türkçe, kısa ve tek konulu mesajlar test edildi.**
7. **Çıktı tutarlılığı ölçülmedi.** Aynı mesaj birkaç kez gönderilirse aynı kategori çıkıyor mu, bilinmiyor.

## Sonraki adımlar

- Test setini 15-20 vakaya çıkarmak ve **prompt'un görmediği** mesajlar eklemek
- `guvenlik` kategorisi için aciliyet kuralı yazmak
- Aynı mesajı 3-5 kez göndererek tutarlılığı ölçmek
- Taslak cevap kuralını, gerçek bir sisteme bağlanana kadar "ilettik" gibi kesin ifadeleri de yasaklayacak şekilde sıkılaştırmak
- Hata oranını sayısal olarak raporlamak (örn. 20 vakada kaç doğru)
