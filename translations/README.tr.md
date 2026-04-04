# steamworks-api-simplified

**Keşke olsaydı dediğimiz Steam API rehberi.**

🌍 *[English](../README.md) · [Deutsch](README.de.md)*

---

Oyununuzu Steam'de yayınladınız. Tebrikler! Şimdi veriye ihtiyacınız var: kaç kişi oynuyor, incelemeler ne diyor, satışlar nasıl gidiyor, istek listesi eklemeleri nereden geliyor.

Steam'in resmi API dökümanlarını açıyorsunuz ve... Kayboluyorsunuz. Üç farklı anahtar türü. İki farklı API sunucusu. Bazı endpoint'ler anahtar istiyor, bazıları istemiyor. Tarih formatları endpoint'ler arasında değişiyor. Finansal veriler sayı yerine metin olarak dönüyor. Dökümanlar her şeyi zaten bildiğinizi varsayıyor.

Biz de aynı şeyleri yaşadık. Bu rehber, öğrendiklerimizi birisinin bize anlatmasını istediğimiz şekilde düzenledik.

---

## İçindekiler

- [API'lere Yeni misiniz?](#apilere-yeni-misiniz)
- [Hızlı Başlangıç: Hangi Anahtara İhtiyacım Var?](#hızlı-başlangıç-hangi-anahtara-i̇htiyacım-var)
- [Üç Tür Steam API Anahtarı](#üç-tür-steam-api-anahtarı)
- [API Sunucuları](#api-sunucuları)
- [Endpoint'ler: Herkese Açık Veriler (Anahtar Gerekmez)](#endpointler-herkese-açık-veriler-anahtar-gerekmez)
- [Endpoint'ler: Satış ve Gelir](#endpointler-satış-ve-gelir-finansalpublisher-anahtar)
- [Endpoint'ler: Sadece Publisher Anahtar](#endpointler-sadece-publisher-anahtar)
- [Bölgesel Fiyat Sorgulama](#bölgesel-fiyat-sorgulama)
- [Steam Dil Kodları](#steam-dil-kodları-language-codes)
- [Paketler ve Bundle'lar](#paketler-ve-bundlelar-packages-and-bundles)
- [API ile Erişilemeyen Veriler](#api-ile-erişilemeyen-veriler-data-not-available-via-api)
- [Sık Yapılan Hatalar](#sık-yapılan-hatalar)
- [Hata Referansı](#hata-referansı)
- [Steamworks Kurulum Kontrol Listesi](#steamworks-kurulum-kontrol-listesi)
- [Resmi Kaynaklar](#resmi-kaynaklar)
- [Yapay Zeka Aracı Becerisi](#yapay-zeka-aracı-becerisi-ai-agent-skill)
- [Katkıda Bulunma](#katkıda-bulunma)
- [Lisans](#lisans)

---

<details>
<summary><b>API'lere Yeni misiniz? Buradan Başlayın</b></summary>

Şimdiye kadar sadece oyun motoruyla çalıştıysanız ve hiç web API çağırmadıysanız, sorun değil. Göründüğünden çok daha basit.

**API sadece bir URL'dir.** URL'yi ziyaret edersiniz ve web sayfası yerine ham veri (genellikle JSON, temelde yapılandırılmış metin) alırsınız.

**Şimdi deneyin.** Bu URL'yi kopyalayıp tarayıcınıza yapıştırın:

```
https://api.steampowered.com/ISteamUserStats/GetNumberOfCurrentPlayers/v1/?appid=3349960
```

Şuna benzer bir şey göreceksiniz:

```json
{ "response": { "player_count": 42, "result": 1 } }
```

İşte bu kadar. Az önce Steam API'yi çağırdınız. Bu URL, [okey.gg](https://store.steampowered.com/app/3349960?utm_source=steamworks_api_simplified) oyununu ([yilmaz.games](https://yilmaz.games?utm_source=steamworks_api_simplified) tarafından yapılan bir oyunu, merhaba, biz 👋) şu an kaç kişinin oynadığını döndürüyor.

**Kodunuzda** aynı şeyi yaparsınız: bir URL'ye istek atıp yanıtı okursunuz:

```javascript
// JavaScript / Node.js
const response = await fetch(
  "https://api.steampowered.com/ISteamUserStats/GetNumberOfCurrentPlayers/v1/?appid=UYGULAMA_ID"
);
const data = await response.json();
console.log(data.response.player_count); // 42
```

```python
# Python
import requests

response = requests.get(
    "https://api.steampowered.com/ISteamUserStats/GetNumberOfCurrentPlayers/v1/",
    params={"appid": "UYGULAMA_ID"}
)
data = response.json()
print(data["response"]["player_count"])  # 42
```

```csharp
// C# / Unity
using var client = new HttpClient();
var response = await client.GetStringAsync(
    "https://api.steampowered.com/ISteamUserStats/GetNumberOfCurrentPlayers/v1/?appid=UYGULAMA_ID"
);
// JSON string'ini parse ederek player_count değerini alın
```

```gdscript
# GDScript / Godot
var http = HTTPRequest.new()
add_child(http)
http.request("https://api.steampowered.com/ISteamUserStats/GetNumberOfCurrentPlayers/v1/?appid=UYGULAMA_ID")
# Yanıtı işlemek için request_completed sinyaline bağlanın
```

**API'leri keşfetmek için faydalı araçlar:**
- Tarayıcınız (cidden, sadece URL'leri yapıştırın)
- [Postman](https://www.postman.com/) veya [Insomnia](https://insomnia.rest/), API çağrıları oluşturmayı ve test etmeyi kolaylaştıran ücretsiz uygulamalar
- Terminalde `curl`: `curl "https://api.steampowered.com/..."`

Artık bu rehberin geri kalanı için hazırsınız. Aşağıdaki her endpoint aynı şekilde çalışır: bir URL, onu çağırırsınız, veri döner.
</details>

---

## Hızlı Başlangıç: Hangi Anahtara İhtiyacım Var?

Endpoint'lere dalmadan önce, her veri türü için neye ihtiyacınız olduğunu öğrenin:

| İstediğiniz veri | Anahtar türü | Nereye istek atılır |
|------------------|-------------|---------------------|
| Anlık oyuncu sayısı | Anahtar gerekmez | api.steampowered.com |
| Uygulama detayları, fiyatlandırma | Anahtar gerekmez | store.steampowered.com |
| İncelemeler | Anahtar gerekmez | store.steampowered.com |
| Haberler | Anahtar gerekmez | api.steampowered.com |
| Başarım yüzdeleri | Anahtar gerekmez | api.steampowered.com |
| **Satış ve gelir** | **Finansal anahtar** veya Publisher anahtar (Sales Data izni ile) | partner.steam-api.com |
| **İstek listesi verileri** | **Finansal anahtar** veya Publisher anahtar (Sales Data izni ile) | partner.steam-api.com |
| Oyun istatistikleri (tanımlıysa) | Publisher anahtar | partner.steam-api.com |
| Sıralama tabloları (tanımlıysa) | Publisher anahtar | partner.steam-api.com |
| Mikrotransaction raporları | Publisher anahtar (Microtransaction izni ile) | partner.steam-api.com |
| Yasaklanmış oyuncular | Publisher anahtar | partner.steam-api.com |

> **Şablonu yakalayın:** Ücretsiz/herkese açık veri → `api.steampowered.com` (anahtar yok). Kendi ticari verileriniz → `partner.steam-api.com` (anahtar gerekli). Yanlış sunucuyu kullanmak en sık yapılan 1 numaralı hatadır.

---

## Üç Tür Steam API Anahtarı

Steam'in üç farklı API anahtarı var. Evet, üç. İşte her birinin ne yaptığı ve nasıl alınacağı.

### 1. Web API Anahtarı

Steam hesabı olan herkesin alabileceği temel anahtar (Basic Key). Muhtemelen buna ihtiyacınız bile yok. Çoğu herkese açık endpoint hiç anahtar olmadan çalışıyor.

**Nasıl alınır:**
1. https://steamcommunity.com/dev/apikey adresine gidin
2. Steam hesabınızla giriş yapın
3. Bir alan adı girin ve kayıt olun
4. Anahtarınızı kopyalayın

**Ne yapabilir:** `api.steampowered.com` üzerindeki herkese açık endpoint'lere erişim. Kullanıcıya özel sorgulamalar (oyuncu profilleri, arkadaş listeleri vb.) için kullanışlıdır.

**Ne yapamaz:** Hiçbir partner veya publisher endpoint'ine erişemez. Satış verisi yok, istek listesi verisi yok, `partner.steam-api.com` üzerinde hiçbir şey yok.

### 2. Publisher Web API Anahtarı

Steamworks partner hesabınızdaki bir uygulama grubuna bağlı anahtar (Publisher Key). En esnek anahtar türü. Tam olarak hangi izinlere sahip olacağını siz seçersiniz.

**Nasıl alınır:**
1. https://partner.steamgames.com adresine giriş yapın
2. **Users & Permissions** → **Manage Groups** bölümüne gidin
3. Mevcut bir grup seçin veya yeni bir tane oluşturun
4. Uygulamalarınızı gruba atayın (henüz atanmamışlarsa)
5. Grup sayfasında **"Create WebAPI Key"** butonuna tıklayın
6. Hangi izinleri etkinleştireceğinizi seçin:
   - **Microtransactions**: işlem raporları ve yönetimi
   - **Sales Data**: satış, gelir ve istek listesi verileri (IPartnerFinancialsService)
   - **Economy**: Steam Envanter Servisi
   - **General API**: kimlik doğrulama, DLC sahiplik kontrolleri
7. İsteğe bağlı olarak IP beyaz listesi yapılandırın ([sık yapılan hatalar](#sık-yapılan-hatalar) bölümüne bakın)

**Ne yapabilir:** Web API anahtarının yapabildiği her şey, artı `partner.steam-api.com` üzerindeki partner endpoint'leri. Ancak sadece grubundaki uygulamalar için ve sadece etkinleştirdiğiniz izinlerle.

> ⚠️ **Satış/istek listesi verisi mi istiyorsunuz?** Anahtarı oluştururken **"Sales Data"** iznini etkinleştirmeniz gerekir. Bu olmadan hata mesajı olmadan boş yanıtlar alırsınız, sadece `{}`. Çok yaygın bir tuzak.

### 3. Finansal API Anahtarı

Sadece finansal veriler için özel amaçlı bir anahtar. TÜM uygulamalarınız için satış ve istek listesi verilerine kısıtlama olmadan erişim sağlar.

**Nasıl alınır:**
1. https://partner.steamgames.com adresine giriş yapın
2. **Users & Permissions** → **Manage Groups** bölümüne gidin
3. **"Create new group"** tıklayın ve **"Financial API Group"** seçin
4. Anahtar hemen grup sayfasında görünür

**Ne yapabilir:** Partner hesabınızdaki tüm uygulamalar için `IPartnerFinancialsService` endpoint'lerine (satış, istek listeleri) erişim, uygulama bazında kısıtlama yok.

**Publisher anahtarından farkı:** Financial API Grubu'nun kullanıcısı ve atanmış uygulaması yoktur. Tamamen finansal veri erişim anahtarı olarak var olur. "Sales Data" izinli bir publisher anahtarı aynı endpoint'lere erişebilir, ancak sadece kendi grubundaki uygulamalar için.

**Hangisini ne zaman kullanmalı:**
- **Tek oyun?** "Sales Data" izinli publisher anahtarı yeterli.
- **Farklı gruplarda birden fazla oyun?** Finansal anahtar (Financial Key) daha pratik. Tüm finansal veriler için tek anahtar.

---

## API Sunucuları

Steam'in iki ayrı API sunucusu var. Yanlış olanı kullanmak en sık yapılan hatadır.

| Sunucu | Kim kullanabilir | Protokol | Anahtar gerekli mi? |
|--------|-----------------|----------|---------------------|
| `api.steampowered.com` | Herkes | HTTP veya HTTPS | Genellikle hayır |
| `partner.steam-api.com` | Sadece Publisher/Finansal anahtarlar | **Sadece HTTPS** | Her zaman |

**Kural:** Herkese açık oyun verilerine bakıyorsanız (oyuncu sayıları, incelemeler, haberler), `api.steampowered.com` kullanın. Kendi ticari verilerinize bakıyorsanız (satışlar, istek listeleri, sahiplik), `partner.steam-api.com` kullanın.

Partner API IP'lerini beyaz listeye almanız gerekiyorsa (örn. kurumsal güvenlik duvarı için):
- `208.64.200.0/22`
- `155.133.239.0/24`

---

## Endpoint'ler: Herkese Açık Veriler (Anahtar Gerekmez)

Bu endpoint'ler herkese açıktır. API anahtarı gerekmez. Şu anda tarayıcınızda test edebilirsiniz.

### Anlık Oyuncu Sayısı

```
GET https://api.steampowered.com/ISteamUserStats/GetNumberOfCurrentPlayers/v1/?appid={appid}
```

Oyunda olan oyuncu sayısını döndürür.

```json
{ "response": { "player_count": 42, "result": 1 } }
```

> 💡 **Deneyin:** Bunu tarayıcınıza yapıştırın → [`https://api.steampowered.com/ISteamUserStats/GetNumberOfCurrentPlayers/v1/?appid=3349960`](https://api.steampowered.com/ISteamUserStats/GetNumberOfCurrentPlayers/v1/?appid=3349960). Bu [okey.gg](https://store.steampowered.com/app/3349960?utm_source=steamworks_api_simplified) için anlık oyuncu sayısı. `3349960` yerine kendi App ID'nizi yazın.

### Uygulama Detayları (Store API)

```
GET https://store.steampowered.com/api/appdetails?appids={appid}
```

Mağaza sayfasındaki her şeyi döndürür: ad, açıklama, fiyatlandırma, ekran görüntüleri, sistem gereksinimleri, çıkış tarihi, türler, kategoriler, geliştiriciler, yayıncılar.

Belirli bir bölge fiyatlandırması için `&cc={ülke_kodu}` ekleyin:

```
GET https://store.steampowered.com/api/appdetails?appids={appid}&cc=us
GET https://store.steampowered.com/api/appdetails?appids={appid}&cc=de
GET https://store.steampowered.com/api/appdetails?appids={appid}&cc=tr
GET https://store.steampowered.com/api/appdetails?appids={appid}&cc=cn
```

`price_overview` nesnesi şunları içerir:
- `currency`: örn. `"USD"`, `"EUR"`, `"TRY"`
- `initial`: kuruş cinsinden taban fiyat (örn. `999` = 9.99$)
- `final`: indirimden sonra kuruş cinsinden güncel fiyat
- `discount_percent`: aktif indirim yüdesi

> ⚠️ **Hız limiti (Rate Limit):** 5 dakikada ~200 istek. Agresif şekilde önbelleğe alın (cache). Mağaza sayfası verileri sık değişmez.

### İncelemeler

```
GET https://store.steampowered.com/appreviews/{appid}?json=1&filter=recent&language=all&num_per_page=100
```

**Parametreler:**
| Parametre | Değerler | Açıklama |
|-----------|---------|----------|
| `filter` | `recent`, `updated`, `all` | Sıralama |
| `language` | `all` veya dil kodu | Dile göre filtrele |
| `num_per_page` | 1–100 | Sayfa başına sonuç |
| `cursor` | `*` (ilk sayfa), sonra yanıttaki değer | Sayfalama |
| `review_type` | `all`, `positive`, `negative` | Duyguya göre filtrele |
| `purchase_type` | `all`, `steam`, `non_steam_purchase` | Satın alma kaynağına göre filtrele |

**Yanıt içerir:**
- `query_summary`: `total_positive`, `total_negative`, `total_reviews`, `review_score`, `review_score_desc`
- `reviews[]`: yazar bilgisi, oyun süresi, dil, metin, `voted_up`, zaman damgası, faydalılık oyları içeren bireysel incelemeler

### Haberler

```
GET https://api.steampowered.com/ISteamNews/GetNewsForApp/v2/?appid={appid}&count=10
```

| Parametre | Varsayılan | Açıklama |
|-----------|-----------|----------|
| `count` | 20 | Döndürülecek makale sayısı |
| `maxlength` | tam | İçeriği kısalt (0 = tam metin) |
| `feeds` | hepsi | Virgülle ayrılmış feed adları |

### Başarım Yüzdeleri

```
GET https://api.steampowered.com/ISteamUserStats/GetGlobalAchievementPercentagesForApp/v2/?gameid={appid}
```

Her başarım için global kilit açma yüzdelerini döndürür. Başarım zorluğunu dengelemek veya topluluk istatistiklerini göstermek için kullanışlıdır.

### Oyun Şeması (İstatistik ve Başarım Listesi)

```
GET https://api.steampowered.com/ISteamUserStats/GetSchemaForGame/v2/?appid={appid}&key={webApiKey}
```

Oyun için tanımlanmış tüm istatistik ve başarımları, görünen adları ve açıklamalarıyla döndürür. `GetGlobalStatsForGame` çağırmadan önce istatistik adlarını keşfetmek için buna ihtiyacınız var.

> Not: Web API anahtarından fayda gören tek herkese açık(ımsı) endpoint budur.

---

## Endpoint'ler: Satış ve Gelir (Finansal/Publisher Anahtar)

Aşağıdaki tüm endpoint'ler `partner.steam-api.com` kullanır ve Finansal anahtar veya **"Sales Data"** izni etkinleştirilmiş Publisher anahtar gerektirir.

> 🔒 **Önemli:** Bu çağrılar sunucudan yapılmalıdır. Publisher veya finansal anahtarınızı istemci tarafı kodda, oyun derlemelerinde veya herkese açık depolarda asla ifşa etmeyin.

### GetDetailedSales

```
GET https://partner.steam-api.com/IPartnerFinancialsService/GetDetailedSales/v001/
    ?key={key}
    &date={YYYY-MM-DD}
    &highwatermark_id=0
```

**Tek bir tarih için tüm uygulamalarınızdaki tüm satışları** döndürür. Uygulama başına endpoint yok. Kodunuzda `primary_appid`'ye göre filtrelemeniz gerekir.

**Parametreler:**
| Parametre | Gerekli | Açıklama |
|-----------|---------|----------|
| `key` | Evet | Finansal veya publisher anahtarınız |
| `date` | Evet | `YYYY-MM-DD`, **Pasifik Saati** (Pacific Time) olarak yorumlanır (UTC değil!) |
| `highwatermark_id` | Evet | `0` ile başlayın. Daha fazla veri varsa yanıt `max_id` içerir. Sayfalama (pagination) için kullanın |

**Yanıt örneği:**
```json
{
  "response": {
    "results": [
      {
        "date": "2026-04-03",
        "line_item_type": "Package",
        "packageid": 1234567,
        "package_sale_type": "Steam",
        "platform": "Windows",
        "country_code": "US",
        "base_price": "999",
        "sale_price": "999",
        "currency": "USD",
        "gross_units_sold": 1,
        "gross_units_returned": 0,
        "gross_sales_usd": "9.9900",
        "gross_returns_usd": "0.0000",
        "net_tax_usd": "0.0000",
        "primary_appid": 1234567,
        "net_units_sold": 1,
        "net_sales_usd": "9.9900"
      }
    ],
    "package_info": ["..."],
    "app_info": ["..."],
    "country_info": ["..."],
    "partner_info": ["..."],
    "max_id": "29557611440"
  }
}
```

Her satır öğesi, tarih, ülke, platform ve pakete göre ayrılmış bir satış veya iadedir.

> ⚠️ **Dikkat:** `gross_sales_usd`, `net_sales_usd` vb. alanların **string** (metin) olduğuna dikkat edin, sayı değil. Dönüştürmeniz gerekir: JavaScript'te `parseFloat(item.gross_sales_usd)`, Python'da `float(item["gross_sales_usd"])`.

### GetAppWishlistReporting

```
GET https://partner.steam-api.com/IPartnerFinancialsService/GetAppWishlistReporting/v001/
    ?key={key}
    &appid={appid}
    &date={YYYY-MM-DD}
```

Belirli bir uygulama için belirli bir tarihteki istek listesi aktivitesini döndürür.

**Parametreler:**
| Parametre | Gerekli | Açıklama |
|-----------|---------|----------|
| `key` | Evet | Finansal veya publisher anahtarınız |
| `appid` | Evet | Oyununuzun App ID'si |
| `date` | Evet | `YYYY-MM-DD`, **GMT** olarak (Pasifik Saati değil, evet, satışlardan farklı) |

> ⚠️ Dün, verisi olan en güncel tarihtir. Bugünün verisi henüz mevcut değil.

**Yanıt örneği:**
```json
{
  "response": {
    "appid": 1234567,
    "date": "2026-04-03",
    "wishlist_summary": {
      "wishlist_adds": 15,
      "wishlist_deletes": 3,
      "wishlist_purchases": 2,
      "wishlist_gifts": 0,
      "wishlist_adds_windows": 12,
      "wishlist_adds_mac": 2,
      "wishlist_adds_linux": 1
    },
    "country_summary": [
      {
        "country_code": "US",
        "country_name": "United States",
        "region": "North America",
        "summary_actions": { "wishlist_adds": 5 }
      }
    ],
    "language_summary": [
      {
        "language": 0,
        "language_name": "english",
        "summary_actions": { "wishlist_adds": 8 }
      }
    ],
    "app_min_date": "2025-01-15"
  }
}
```

`app_min_date` bu uygulama için verinin bulunduğu en eski tarihi belirtir. Bundan önceki tarihleri istemeyin. Sadece boş yanıt alırsınız.

### GetChangedDatesForPartner

```
GET https://partner.steam-api.com/IPartnerFinancialsService/GetChangedDatesForPartner/v001/
    ?key={key}
    &highwatermark=0
```

Yeni veya güncellenmiş finansal verisi olan tarihleri döndürür. Verimli senkronizasyon için kullanın. Her günü sorgulamak yerine sadece bu listede görünen tarihleri yeniden çekin.

```json
{
  "response": {
    "dates": ["2026/04/01", "2026/04/02", "2026/04/03"]
  }
}
```

> ⚠️ **Tarih formatı tuzağı:** Bu tarihler **eğik çizgi** kullanır (`YYYY/MM/DD`), ancak diğer tüm endpoint'ler **tire** kullanır (`YYYY-MM-DD`). Dönüştürmeniz gerekir: `date.replace(/\//g, '-')`.

---

## Endpoint'ler: Sadece Publisher Anahtar

Bunlar uygulamayla ilişkilendirilmiş bir Publisher anahtarı gerektirir. `partner.steam-api.com` kullanırlar.

### GetGlobalStatsForGame

```
GET https://partner.steam-api.com/ISteamUserStats/GetGlobalStatsForGame/v1/
    ?key={key}
    &appid={appid}
    &count=1
    &name[0]=istatistik_adi
```

İsteğe bağlı tarih aralığı filtreleme ile toplanmış global istatistikleri döndürür.

| Parametre | Açıklama |
|-----------|----------|
| `name[0]`, `name[1]`, vb. | Çekilecek istatistik adları |
| `count` | İstenen istatistik sayısı |
| `startdate`, `enddate` | Günlük toplamlar için isteğe bağlı Unix zaman damgaları |

> **Ön koşul:** Uygulamanızda **Steamworks → App Admin → Stats & Achievements** bölümünde istatistikler (stats) tanımlanmış olmalıdır. İstatistik adlarını bilmeniz gerekir. Keşfetmek için `GetSchemaForGame` kullanın.

### GetPartnerAppListForWebAPIKey

```
GET https://partner.steam-api.com/ISteamApps/GetPartnerAppListForWebAPIKey/v2/?key={key}
```

Publisher anahtarınızın erişebildiği tüm uygulamaları döndürür. Anahtarınızın doğru ayarlandığını doğrulamak için çok kullanışlıdır.

### GetPlayersBanned

```
GET https://partner.steam-api.com/ISteamApps/GetPlayersBanned/v1/?key={key}&appid={appid}
```

Oyununuzdaki yasaklanmış oyuncuların listesini döndürür.

### GetLeaderboardsForGame

```
GET https://partner.steam-api.com/ISteamLeaderboards/GetLeaderboardsForGame/v2/?key={key}&appid={appid}
```

Oyununuz için tanımlanmış tüm sıralama tablolarını döndürür.

> **Ön koşul:** Sıralama tabloları önce **Steamworks → App Admin → Leaderboards** bölümünde oluşturulmalıdır.

### ISteamMicroTxn/GetReport

```
GET https://partner.steam-api.com/ISteamMicroTxn/GetReport/v5/
    ?key={key}
    &appid={appid}
    &type=GAMESALES
    &time={RFC3339}
    &maxresults=1000
```

Mikrotransaction raporlarını döndürür. Rapor türleri: `GAMESALES`, `STEAMSTORESALES`, `SETTLEMENT`, `CHARGEBACK`, `SUBSCRIPTION`.

> **Ön koşul:** Uygulamanız Steam mikrotransaction'ları kullanıyor olmalıdır.

---

## Bölgesel Fiyat Sorgulama

Oyununuzun farklı bölgelerdeki fiyatını kontrol etmek için Store API'yi ülke koduyla çağırın:

```bash
# ABD (varsayılan)
curl "https://store.steampowered.com/api/appdetails?appids={appid}&cc=us"

# Avrupa Bölgesi
curl "https://store.steampowered.com/api/appdetails?appids={appid}&cc=de"

# Türkiye
curl "https://store.steampowered.com/api/appdetails?appids={appid}&cc=tr"

# Çin
curl "https://store.steampowered.com/api/appdetails?appids={appid}&cc=cn"

# Brezilya
curl "https://store.steampowered.com/api/appdetails?appids={appid}&cc=br"

# Japonya
curl "https://store.steampowered.com/api/appdetails?appids={appid}&cc=jp"
```

Yanıttaki `price_overview` nesnesi şunları içerir:
- `currency`: örn. `"USD"`, `"EUR"`, `"TRY"`, `"CNY"`
- `initial`: kuruş cinsinden taban fiyat
- `final`: kuruş cinsinden indirimli fiyat
- `discount_percent`: aktif indirim yüdesi

## Steam Dil Kodları (Language Codes)

Steam, API çağrıları ve mağaza sayfaları için kendi dil kodlarını kullanır. Bazıları standart dışıdır (`schinese`, `tchinese`, `brazilian`, `koreana`, `latam` gibi), bu yüzden ISO kodlarını doğrudan kullanamazsınız.

| İngilizce Ad | Yerel Ad | API Dil Kodu | Web API Kodu |
|---|---|---|---|
| Arabic | العربية | `arabic` | `ar` |
| Bulgarian | български език | `bulgarian` | `bg` |
| Chinese (Simplified) | 简体中文 | `schinese` | `zh-CN` |
| Chinese (Traditional) | 繁體中文 | `tchinese` | `zh-TW` |
| Czech | čeština | `czech` | `cs` |
| Danish | Dansk | `danish` | `da` |
| Dutch | Nederlands | `dutch` | `nl` |
| English | English | `english` | `en` |
| Finnish | Suomi | `finnish` | `fi` |
| French | Français | `french` | `fr` |
| German | Deutsch | `german` | `de` |
| Greek | Ελληνικά | `greek` | `el` |
| Hungarian | Magyar | `hungarian` | `hu` |
| Indonesian | Bahasa Indonesia | `indonesian` | `id` |
| Italian | Italiano | `italian` | `it` |
| Japanese | 日本語 | `japanese` | `ja` |
| Korean | 한국어 | `koreana` | `ko` |
| Norwegian | Norsk | `norwegian` | `no` |
| Polish | Polski | `polish` | `pl` |
| Portuguese | Português | `portuguese` | `pt` |
| Portuguese (Brazil) | Português-Brasil | `brazilian` | `pt-BR` |
| Romanian | Română | `romanian` | `ro` |
| Russian | Русский | `russian` | `ru` |
| Spanish (Spain) | Español-España | `spanish` | `es` |
| Spanish (Latin America) | Español-Latinoamérica | `latam` | `es-419` |
| Swedish | Svenska | `swedish` | `sv` |
| Thai | ไทย | `thai` | `th` |
| Turkish | Türkçe | `turkish` | `tr` |
| Ukrainian | Українська | `ukrainian` | `uk` |
| Vietnamese | Tiếng Việt | `vietnamese` | `vi` |

> **Kaynak:** [Steamworks Dil Dokümantasyonu](https://partner.steamgames.com/doc/store/localization/languages)
>
> **Not:** API dil kodları Steamworks istemci tarafı API'lerinde kullanılır. Web API dil kodları ise Steamworks Web API'sinde kullanılır. Ek diller (Afrikanca, Arnavutça, İbranice, Hintçe vb.) sadece mağaza sayfası dil seçimi için mevcuttur ve API'lerde desteklenmez.

> 💡 **İpucu:** URL'ye `?l=` parametresi ekleyerek herhangi bir Steam mağaza sayfasını farklı bir dilde önizleyebilirsiniz. Örneğin: [`store.steampowered.com/app/3349960/okeygg/?l=turkish`](https://store.steampowered.com/app/3349960/okeygg/?l=turkish&utm_source=steamworks_api_simplified) [okey.gg](https://store.steampowered.com/app/3349960?utm_source=steamworks_api_simplified) mağaza sayfasını Türkçe gösterir. Yukarıdaki tablodan API dil kodlarını kullanın (Web API kodlarını değil).

---

## Paketler ve Bundle'lar (Packages and Bundles)

Steam uygulamaları doğrudan satmaz. Her satın alma bir **paket** (package, "sub" olarak da bilinir) üzerinden gerçekleşir. Oyununuzun DLC'si, farklı sürümü veya bundle'ı olmasa bile, otomatik olarak oluşturulmuş en az bir varsayılan paketi vardır.

**Bu neden önemli:** Bazı Steamworks özellikleri (iade verileri gibi) App ID yerine **Paket ID'si** (Package ID) kullanır. İade istatistiklerinizi arıyorsanız ve App ID URL'de çalışmıyorsa, muhtemelen Paket ID'si gerekiyordur.

**Paket ID'nizi (Package ID) nasıl bulursunuz:**
1. [partner.steamgames.com/apps/associated/{appid}](https://partner.steamgames.com/apps/associated/3349960) adresine gidin
2. **"All Associated Packages"** (Tüm İlişkili Paketler) bölümüne bakın
3. Varsayılan paketiniz genellikle "{Oyun Adı} for Steam" veya benzeri bir isimle listelenir

Örneğin, [okey.gg](https://store.steampowered.com/app/3349960?utm_source=steamworks_api_simplified) App ID'si `3349960` ve varsayılan Paket ID'si `1185712`'dir.

**Bundle'lar** (Paket Grupları) paketlerden farklıdır. Bir bundle, birden fazla paketi indirimli olarak gruplar (örneğin temel oyun + tüm DLC'leri içeren bir "Tam Sürüm"). Bundle'ların kendi Bundle ID'leri vardır ve **Steamworks → Store Page → Bundles** bölümünden yönetilir. Bundle fiyatlandırması, birleşik paket fiyatlarından bundle indirimi çıkarılarak hesaplanır.

> Daha fazla bilgi: [Steamworks Paket Dokümantasyonu (Packages Documentation)](https://partner.steamgames.com/doc/store/application/packages)

---

## API ile Erişilemeyen Veriler (Data NOT Available via API)

Bazı veriler sadece Steamworks Partner Portal web arayüzünde mevcuttur. Bunlar için API endpoint'i yoktur. Valve bunları sunmamaktadır.

| Veri | Nerede bulunur |
|------|---------------|
| Mağaza sayfası trafiği (Store Page Traffic) | [partner.steamgames.com/apps/navtrafficstats/{appid}](https://partner.steamgames.com/apps/navtrafficstats/3349960) |
| UTM kampanya analitiği (UTM Analytics) | [partner.steamgames.com/apps/utmtrafficstats/{appid}](https://partner.steamgames.com/apps/utmtrafficstats/3349960) |
| İade detayları ve nedenleri (Refund Data) | [partner.steampowered.com/package/refunds/{packageid}/](https://partner.steampowered.com/package/refunds/1185712/) |

Bu portal sayfaları, veriyi başka yere aktarmanız gerekiyorsa CSV dışa aktarma (export) sunar.

> ⚠️ **İadeler App ID değil, Paket ID'si (Package ID) kullanır.** Steam satın almaları App ID'ye değil, paketlere (packages/subs) göre düzenler. Paket ID'nizi bulmak için [partner.steamgames.com/apps/associated/{appid}](https://partner.steamgames.com/apps/associated/3349960) adresine gidin. Örneğin, [okey.gg](https://store.steampowered.com/app/3349960?utm_source=steamworks_api_simplified) App ID'si `3349960` ama ana Paket ID'si `1185712`'dir.
>
> Daha fazla bilgi: [Steamworks Paket Dokümantasyonu (Packages Documentation)](https://partner.steamgames.com/doc/store/application/packages)

---

## Sık Yapılan Hatalar

Sizi uyarmadan tuzağa düşürecek şeyler. Kendinizi uyarılmış kabul edin.

### Tarih formatları tutarsız

| Endpoint | Tarih formatı | Saat dilimi |
|----------|--------------|-------------|
| `GetDetailedSales` | `YYYY-MM-DD` (tire) | Pasifik Saati |
| `GetAppWishlistReporting` | `YYYY-MM-DD` (tire) | GMT |
| `GetChangedDatesForPartner` | `YYYY/MM/DD` (eğik çizgi) | - |

Evet, satış verileri Pasifik Saati'ni, istek listesi verileri GMT'yi kullanıyor. Ve değişen tarihler eğik çizgi kullanırken diğer her şey tire kullanıyor. Steam'e hoş geldiniz.

### Finansal değerler string, sayı değil

`gross_sales_usd`, `net_sales_usd`, `base_price` gibi alanlar `"9.9900"` (string) olarak döner, `9.99` (sayı) olarak değil. Her zaman dönüştürün:

```javascript
// ❌ Yanlış: bu string karşılaştırır
if (item.gross_sales_usd > 0) { ... }

// ✅ Doğru
if (parseFloat(item.gross_sales_usd) > 0) { ... }
```

### Boş yanıt ≠ hata

Publisher anahtarınız geçerli ama "Sales Data" izni eksikse, Steam şunu döndürür:

```json
{ "response": {} }
```

Hata kodu yok. Hata mesajı yok. Sadece... hiçbir şey. `IPartnerFinancialsService`'den boş yanıtlar alıyorsanız, anahtarınızın izinlerini Steamworks'te kontrol edin.

### Serverless + IP beyaz listesi bir arada çalışmaz

Vercel, AWS Lambda, Cloudflare Workers veya herhangi bir serverless platforma deploy ediyorsanız, sunucunuzun IP adresi her istekte değişir. Steam anahtarınızda IP beyaz listesi etkinse, her istek başarısız olur.

**Çözüm:** Steamworks'te anahtarınızdan IP kısıtlamalarını kaldırın veya sabit IP'li bir proxy kullanın.

### Her tarihte satış verisi olmayabilir

Bazı günler oyununuz satılmaz. API bu tarihler için boş sonuç döndürür. Bu normal, hata değil. Her günü körlemesine sorgulamak yerine hangi tarihlerde gerçekten veri olduğunu bulmak için `GetChangedDatesForPartner` kullanın.

### İstek listesi verisinin minimum tarihi var

Her uygulamanın bir `app_min_date` değeri vardır (istek listesi yanıtında döner). Bu tarihten önceki veriler mevcut değildir. Ondan önceki tarihler için istek harcamayın.

---

## Hata Referansı

| Ne görüyorsunuz | Ne anlama geliyor | Nasıl düzeltilir |
|---|---|---|
| HTTP 403 + "Access is denied" HTML sayfası | Bu sunucu için yanlış anahtar türü | `partner.steam-api.com` için publisher/finansal anahtar kullanın. Normal Web API anahtarı çalışmaz. |
| HTTP 200 + `{"response":{}}` | Anahtar geçerli ama izin eksik | Steamworks'te publisher grubunda "Sales Data" iznini etkinleştirin |
| HTTP 200 + `{"response":{"result":8}}` | İstatistik/veri yapılandırılmamış | Önce Steamworks'te istatistik, başarım veya sıralama tablosu oluşturun |
| HTTP 429 | Hız limiti aşıldı | Yavaşlayın. Önbellekleme ekleyin. |
| Partner API'de bağlantı zaman aşımı | IP beyaz listesi tarafından engellendi | Anahtardaki IP kısıtlamalarını kaldırın veya sunucu IP'nizi ekleyin |

---

## Steamworks Kurulum Kontrol Listesi

Bazı endpoint'ler, ilgili özelliği Steamworks'te yapılandırana kadar hiçbir şey döndürmez. İşte ne kurulacak ve neyi etkinleştirecek:

| Özellik | Nerede yapılandırılır | Ne etkinleştirir |
|---------|----------------------|-----------------|
| İstatistikler | Steamworks → App Admin → Stats & Achievements → Stats | `GetGlobalStatsForGame`: zaman aralıklı toplanmış oyun istatistikleri |
| Başarımlar | Steamworks → App Admin → Stats & Achievements → Achievements | `GetGlobalAchievementPercentagesForApp`: kilit açma yüzdeleri |
| Sıralama Tabloları | Steamworks → App Admin → Leaderboards | `GetLeaderboardsForGame` + `GetLeaderboardEntries` |
| Mikrotransaction'lar | Steamworks → App Admin → Microtransaction Configuration | `ISteamMicroTxn/GetReport`: işlem raporları |
| Steam Envanteri | Steamworks → App Admin → Steam Inventory Service | `IInventoryService` endpoint'leri |

---

## Resmi Kaynaklar

- [Steamworks Web API Genel Bakış](https://partner.steamgames.com/doc/webapi_overview)
- [Web API Kimlik Doğrulama ve Anahtar Türleri](https://partner.steamgames.com/doc/webapi_overview/auth)
- [IPartnerFinancialsService](https://partner.steamgames.com/doc/webapi/IPartnerFinancialsService)
- [İstek Listesi Raporlama](https://partner.steamgames.com/doc/marketing/wishlist/reporting)
- [İstek Listesi Veri API Duyurusu](https://store.steampowered.com/news/group/4145017/view/499474120884358023)
- [Tam Arayüz Listesi](https://partner.steamgames.com/doc/webapi)

---

## Yapay Zeka Aracı Becerisi (AI Agent Skill)

Bu rehber aynı zamanda bir yapay zeka aracı becerisi olarak da mevcuttur. [Claude Code](https://claude.ai/code) veya benzeri yapay zeka kodlama araçları kullanıyorsanız, [`steamworks-api-specialist`](../steamworks-api-specialist/SKILL.md) becerisini yükleyerek yapay zeka asistanınıza Steam Web API hakkında derin bilgi verebilirsiniz — anahtar türleri, endpoint detayları, yaygın tuzaklar ve sorun giderme akışları — böylece entegrasyonunuzu daha hızlı yapabilirsiniz.

---

## Katkıda Bulunma

Bir hata mı buldunuz? Kaçırdığımız bir endpoint mi biliyorsunuz? Çeviri mi eklemek istiyorsunuz?

Yardımınızı çok isteriz. Detaylar için [Katkıda Bulunma Rehberi](../.github/CONTRIBUTING.md)'ne göz atın.

**Kısa versiyon:**
1. Bu depoyu fork'layın
2. Değişikliklerinizi yapın
3. PR gönderin

Çeviriler `translations/README.{dil-kodu}.md` formatında gider. Referans için [mevcut çevirilere](#) bakın.

Bu rehber size zaman kazandırdıysa, bir ⭐ vermeyi düşünün. Diğer indie geliştiricilerin de bulmasına yardımcı olur.

---

## Lisans

[CC0 1.0 Evrensel (Kamu Malı)](../LICENSE). Bununla istediğinizi yapın. Kopyalıyın, değiştirin, projenize dahil edin, etrafında bir kurs satın. Atıf gerekli değil (ama her zaman takdir edilir 🙏).

---

<p align="center">
  <br>
  ❤️ ile yapıldı, <a href="https://yilmaz.games?utm_source=steamworks_api_simplified">yilmaz.games</a> tarafından
  <br><br>
  Bu rehberi <a href="https://store.steampowered.com/app/3349960?utm_source=steamworks_api_simplified">okey.gg</a> oyunumuz için Steam API'lerini entegre ederken hazırladık.<br>
  İşinize yaradıysa, bir <a href="https://store.steampowered.com/app/3349960?utm_source=steamworks_api_simplified">istek listesi</a> eklemeniz küçük bir stüdyo için dünyalara bedel 🙏
  <br><br>
  <sub>Son güncelleme: Nisan 2026</sub>
</p>
