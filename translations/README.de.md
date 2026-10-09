# steamworks-api-simplified

**Der Steam-API-Guide, den wir uns gewünscht hätten.**

Ein praktischer Guide zur Steamworks Web API mit Verkaufsdaten, Wunschlisten-Reporting, Reviews, Spielerzahlen, Achievements und Partner-Endpoints. Fokus auf reale Anwendungsfälle, häufige Stolperfallen und klare Beispiele für Spieleentwickler.

🌍 *[English](../README.md) · [Türkçe](README.tr.md)*

> 📢 **Achtung:** Steam schaltet den alten Reviews-Endpoint (`store.steampowered.com/appreviews`) am **22. Oktober 2026** ab. Wenn du Reviews abrufst, sieh dir [Migration von /appreviews](#migration-von-appreviews) an.

---

Du hast dein Spiel auf Steam veröffentlicht. Glückwunsch! Jetzt brauchst du Daten: wie viele Leute spielen gerade, was sagen die Reviews, wie laufen die Verkäufe, woher kommen die Wunschlisten-Einträge.

Du öffnest die offizielle Steam-API-Doku und... Labyrinth. Drei verschiedene Schlüsseltypen. Zwei verschiedene API-Hosts. Manche Endpoints brauchen einen Schlüssel, manche nicht. Datumsformate wechseln zwischen Endpoints. Finanzdaten kommen als Strings statt als Zahlen. Die Doku setzt voraus, dass du schon alles weißt.

Kennen wir. Dieser Guide ist alles, was wir gelernt haben, so aufbereitet, wie wir es uns gewünscht hätten, dass es uns jemand erklärt.

---

## Inhaltsverzeichnis

- [Neu bei APIs?](#neu-bei-apis)
- [Schnellstart: Welchen Schlüssel brauche ich?](#schnellstart-welchen-schlüssel-brauche-ich)
- [Die drei Arten von Steam-API-Schlüsseln](#die-drei-arten-von-steam-api-schlüsseln)
- [API-Hosts](#api-hosts)
- [Endpoints: Öffentliche Daten (kein Schlüssel nötig)](#endpoints-öffentliche-daten-kein-schlüssel-nötig)
- [Endpoints: Verkäufe & Umsatz](#endpoints-verkäufe--umsatz-financialpublisher-schlüssel)
- [Endpoints: Nur Publisher-Schlüssel](#endpoints-nur-publisher-schlüssel)
- [Regionale Preisabfrage](#regionale-preisabfrage)
- [Steam-Sprachcodes](#steam-sprachcodes-language-codes)
- [Pakete und Bundles](#pakete-und-bundles-packages-and-bundles)
- [Daten, die NICHT per API verfügbar sind](#daten-die-nicht-per-api-verfügbar-sind-data-not-available-via-api)
- [Häufige Stolperfallen](#häufige-stolperfallen)
- [Fehler-Referenz](#fehler-referenz)
- [Steamworks-Einrichtungs-Checkliste](#steamworks-einrichtungs-checkliste)
- [Offizielle Quellen](#offizielle-quellen)
- [KI-Agenten-Skill](#ki-agenten-skill)
- [Mitwirken](#mitwirken)
- [Lizenz](#lizenz)

---

<a name="neu-bei-apis"></a>
<details>
<summary><b>Neu bei APIs? Hier anfangen</b></summary>

Wenn du bisher nur in einer Game Engine gearbeitet und noch nie eine Web-API aufgerufen hast, kein Problem. Es ist einfacher als es klingt.

**Eine API ist einfach eine URL.** Du rufst die URL auf und bekommst statt einer Webseite Rohdaten zurück (meistens JSON, im Grunde strukturierter Text).

**Probier es jetzt aus.** Kopiere diese URL und füge sie in deinen Browser ein:

```
https://api.steampowered.com/ISteamUserStats/GetNumberOfCurrentPlayers/v1/?appid=3349960
```

Du siehst dann so etwas:

```json
{ "response": { "player_count": 42, "result": 1 } }
```

Das war's. Du hast gerade die Steam-API aufgerufen. Diese URL gibt die Anzahl der Spieler zurück, die gerade [okey.gg](https://store.steampowered.com/app/3349960?utm_source=steamworks_api_simplified) spielen (ein Spiel von [yilmaz.games](https://yilmaz.games?utm_source=steamworks_api_simplified), hi, das sind wir 👋).

**In deinem Code** machst du genau dasselbe: eine URL aufrufen und die Antwort lesen:

```javascript
// JavaScript / Node.js
const response = await fetch(
  "https://api.steampowered.com/ISteamUserStats/GetNumberOfCurrentPlayers/v1/?appid=DEINE_APP_ID"
);
const data = await response.json();
console.log(data.response.player_count); // 42
```

```python
# Python
import requests

response = requests.get(
    "https://api.steampowered.com/ISteamUserStats/GetNumberOfCurrentPlayers/v1/",
    params={"appid": "DEINE_APP_ID"}
)
data = response.json()
print(data["response"]["player_count"])  # 42
```

```csharp
// C# / Unity
using var client = new HttpClient();
var response = await client.GetStringAsync(
    "https://api.steampowered.com/ISteamUserStats/GetNumberOfCurrentPlayers/v1/?appid=DEINE_APP_ID"
);
// JSON-String parsen, um player_count zu erhalten
```

```gdscript
# GDScript / Godot
var http = HTTPRequest.new()
add_child(http)
http.request("https://api.steampowered.com/ISteamUserStats/GetNumberOfCurrentPlayers/v1/?appid=DEINE_APP_ID")
# Mit dem request_completed Signal verbinden, um die Antwort zu verarbeiten
```

**Nützliche Tools zum Erkunden von APIs:**
- Dein Browser (ernsthaft, einfach URLs einfügen)
- [Postman](https://www.postman.com/) oder [Insomnia](https://insomnia.rest/), kostenlose Apps, die API-Aufrufe einfach machen
- `curl` im Terminal: `curl "https://api.steampowered.com/..."`

Jetzt bist du bereit für den Rest dieses Guides. Jeder Endpoint unten funktioniert gleich: eine URL, du rufst sie auf, du bekommst Daten zurück.
</details>

---

## Schnellstart: Welchen Schlüssel brauche ich?

Bevor du in die Endpoints eintauchst, hier eine Übersicht, was du für welche Daten brauchst:

| Gewünschte Daten | Schlüsseltyp | Wohin die Anfrage geht |
|------------------|-------------|------------------------|
| Aktuelle Online-Spieler | Kein Schlüssel nötig | api.steampowered.com |
| App-Details, Preise | Kein Schlüssel nötig | store.steampowered.com |
| Reviews | Kein Schlüssel nötig | api.steampowered.com |
| News | Kein Schlüssel nötig | api.steampowered.com |
| Achievement-Prozentsätze | Kein Schlüssel nötig | api.steampowered.com |
| **Verkäufe & Umsatz** | **Financial-Schlüssel** oder Publisher-Schlüssel (mit Sales Data Berechtigung) | partner.steam-api.com |
| **Wunschlisten-Daten** | **Financial-Schlüssel** oder Publisher-Schlüssel (mit Sales Data Berechtigung) | partner.steam-api.com |
| Spielstatistiken (falls definiert) | Publisher-Schlüssel | partner.steam-api.com |
| Bestenlisten (falls definiert) | Publisher-Schlüssel | partner.steam-api.com |
| Mikrotransaktions-Berichte | Publisher-Schlüssel (mit Microtransaction Berechtigung) | partner.steam-api.com |
| Gesperrte Spieler | Publisher-Schlüssel | partner.steam-api.com |

> **Erkennst du das Muster?** Kostenlose/öffentliche Daten → `api.steampowered.com` (kein Schlüssel). Deine privaten Geschäftsdaten → `partner.steam-api.com` (Schlüssel erforderlich). Den falschen Host zu verwenden ist der häufigste Fehler überhaupt.

---

## Die drei Arten von Steam-API-Schlüsseln

Steam hat drei verschiedene API-Schlüssel. Ja, drei. Hier ist, was jeder kann und wie du ihn bekommst.

### 1. Web API Key

Der einfache Schlüssel (Basic Key), den jeder mit einem Steam-Account bekommen kann. Du brauchst ihn wahrscheinlich gar nicht. Die meisten öffentlichen Endpoints funktionieren komplett ohne Schlüssel.

**So bekommst du ihn:**
1. Gehe zu https://steamcommunity.com/dev/apikey
2. Logge dich mit deinem Steam-Account ein
3. Gib einen Domainnamen ein und registriere
4. Kopiere deinen Schlüssel

**Was er kann:** Zugriff auf öffentliche Endpoints auf `api.steampowered.com`. Nützlich für benutzerspezifische Abfragen (Spielerprofile, Freundeslisten usw.).

**Was er nicht kann:** Kein Zugriff auf Partner- oder Publisher-Endpoints. Keine Verkaufsdaten, keine Wunschlistendaten, nichts auf `partner.steam-api.com`.

### 2. Publisher Web API Key

Ein Schlüssel (Publisher Key), der an eine App-Gruppe in deinem Steamworks-Partner-Account gebunden ist. Der flexibelste Schlüsseltyp. Du wählst genau, welche Berechtigungen er hat.

**So bekommst du ihn:**
1. Logge dich ein auf https://partner.steamgames.com
2. Gehe zu **Users & Permissions** → **Manage Groups**
3. Wähle eine bestehende Gruppe oder erstelle eine neue
4. Weise deine Apps der Gruppe zu (falls noch nicht geschehen)
5. Klicke auf **"Create WebAPI Key"** auf der Gruppenseite
6. Wähle die Berechtigungen:
   - **Microtransactions**: Transaktionsberichte und -verwaltung
   - **Sales Data**: Verkaufs-, Umsatz- und Wunschlistendaten (IPartnerFinancialsService)
   - **Economy**: Steam Inventory Service
   - **General API**: Authentifizierung, DLC-Besitzprüfungen
7. Optional IP-Whitelist konfigurieren (siehe [Stolperfallen](#häufige-stolperfallen) bei Serverless)

**Was er kann:** Alles was der Web API Key kann, plus Partner-Endpoints auf `partner.steam-api.com`. Aber nur für Apps in seiner Gruppe und nur mit den aktivierten Berechtigungen.

> ⚠️ **Verkaufs-/Wunschlistendaten gewünscht?** Du musst die **"Sales Data"**-Berechtigung beim Erstellen des Schlüssels aktivieren. Ohne bekommst du leere Antworten ohne Fehlermeldung, nur `{}`. Eine sehr häufige Falle.

### 3. Financial API Key

Ein Spezial-Schlüssel nur für Finanzdaten. Gibt dir uneingeschränkten Zugriff auf Verkaufs- und Wunschlistendaten für ALLE deine Apps.

**So bekommst du ihn:**
1. Logge dich ein auf https://partner.steamgames.com
2. Gehe zu **Users & Permissions** → **Manage Groups**
3. Klicke auf **"Create new group"** und wähle **"Financial API Group"**
4. Der Schlüssel erscheint sofort auf der Gruppenseite

**Was er kann:** Zugriff auf `IPartnerFinancialsService`-Endpoints (Verkäufe, Wunschlisten) für jede App auf deinem Partner-Account, keine App-Einschränkungen.

**Unterschied zum Publisher Key:** Eine Financial API Group hat keine Benutzer und keine zugewiesenen Apps. Sie existiert ausschließlich als Finanzdaten-Zugriffsschlüssel. Ein Publisher Key mit "Sales Data"-Berechtigung kann auf dieselben Endpoints zugreifen, aber nur für Apps in seiner spezifischen Gruppe.

**Welchen wann verwenden:**
- **Ein Spiel?** Publisher Key mit "Sales Data"-Berechtigung reicht.
- **Mehrere Spiele in verschiedenen Gruppen?** Financial Key ist einfacher. Ein Schlüssel für alle Finanzdaten.

---

## API-Hosts

Steam hat zwei separate API-Server. Den falschen zu verwenden ist der allerhäufigste Fehler.

| Host | Wer kann ihn nutzen | Protokoll | Schlüssel nötig? |
|------|---------------------|-----------|------------------|
| `api.steampowered.com` | Jeder | HTTP oder HTTPS | Meistens nein |
| `partner.steam-api.com` | Nur Publisher/Financial Keys | **Nur HTTPS** | Immer |

**Faustregel:** Öffentliche Spieldaten (Spielerzahlen, Reviews, News) → `api.steampowered.com`. Deine eigenen Geschäftsdaten (Verkäufe, Wunschlisten, Besitz) → `partner.steam-api.com`.

Falls du Partner-API-IPs whitelisten musst (z.B. für eine Unternehmens-Firewall):
- `208.64.200.0/22`
- `155.133.239.0/24`

---

## Endpoints: Öffentliche Daten (kein Schlüssel nötig)

Diese Endpoints sind für jeden offen. Kein API-Schlüssel nötig. Du kannst sie jetzt direkt im Browser testen.

### Aktuelle Online-Spieler

```
GET https://api.steampowered.com/ISteamUserStats/GetNumberOfCurrentPlayers/v1/?appid={appid}
```

Gibt die Anzahl der aktuell im Spiel befindlichen Spieler zurück.

```json
{ "response": { "player_count": 42, "result": 1 } }
```

> 💡 **Probiere es aus:** Füge das in deinen Browser ein → [`https://api.steampowered.com/ISteamUserStats/GetNumberOfCurrentPlayers/v1/?appid=3349960`](https://api.steampowered.com/ISteamUserStats/GetNumberOfCurrentPlayers/v1/?appid=3349960). Das ist die Live-Spielerzahl für [okey.gg](https://store.steampowered.com/app/3349960?utm_source=steamworks_api_simplified). Ersetze `3349960` mit deiner eigenen App ID.

### App-Details (Store API)

```
GET https://store.steampowered.com/api/appdetails?appids={appid}
```

Gibt alles von der Store-Seite zurück: Name, Beschreibung, Preise, Screenshots, Systemanforderungen, Erscheinungsdatum, Genres, Kategorien, Entwickler, Publisher.

Für Preise in einer bestimmten Region füge `&cc={ländercode}` hinzu:

```
GET https://store.steampowered.com/api/appdetails?appids={appid}&cc=us
GET https://store.steampowered.com/api/appdetails?appids={appid}&cc=de
GET https://store.steampowered.com/api/appdetails?appids={appid}&cc=tr
GET https://store.steampowered.com/api/appdetails?appids={appid}&cc=cn
```

Das `price_overview`-Objekt enthält:
- `currency`: z.B. `"USD"`, `"EUR"`, `"TRY"`
- `initial`: Grundpreis in Cent (z.B. `999` = 9,99 $)
- `final`: aktueller Preis in Cent (nach Rabatt)
- `discount_percent`: aktiver Rabattprozentsatz

> ⚠️ **Rate Limit:** ~200 Anfragen pro 5 Minuten. Aggressiv cachen. Store-Daten ändern sich selten.

### Reviews

> ⚠️ **Geändert im Oktober 2026.** Steam schaltet den alten Endpoint `store.steampowered.com/appreviews/{appid}?json=1` am **22. Oktober 2026** ab. Wenn dein Code ihn noch aufruft, sieh dir unten [Migration von /appreviews](#migration-von-appreviews) an.

```
GET https://api.steampowered.com/IUserReviewsService/GetAppReviews/v1/?appid={appid}&filter=1&languages[0]=all&purchase_type=1&num_per_page=100
```

Gibt eine Seite mit öffentlichen Reviews für eine beliebige App zurück, plus die Zusammenfassung des Review-Scores. Kein Schlüssel nötig.

**Parameter:**
| Parameter | Werte | Beschreibung |
|-----------|-------|-------------|
| `appid` | Deine App ID | Pflicht |
| `filter` | `0` Helpful (Standard), `1` Recent, `2` Updated, `3` Funny | Sortierung. Helpful liefert nur Reviews der letzten `day_range` Tage (wird automatisch erweitert, wenn der Zeitraum leer ist), nutze also `1` oder `2`, um alle Reviews zu bekommen |
| `day_range` | Tage. Standard 30, max. 365, `0` = kein Limit | Nur für Helpful. Wenn der Zeitraum keine Reviews enthält, erweitert Steam ihn. `day_range_used` in der Antwort zeigt, welcher Zeitraum tatsächlich durchsucht wurde |
| `languages[0]`, `languages[1]`, ... | `all` oder [API-Sprachcodes](#steam-sprachcodes-language-codes) (`english`, `german`, ...) | **Standard ist nur Englisch.** Übergib `languages[0]=all` für alle Sprachen |
| `num_per_page` | 1–100 | Ergebnisse pro Seite (Standard 20) |
| `cursor` | `*` (erste Seite), dann Wert aus Antwort | Paginierung. **URL-encoden**, da er `+`, `/` und `=` enthalten kann |
| `review_type` | `0` All (Standard), `1` Positive, `2` Negative | Nach Stimmung filtern |
| `purchase_type` | `0` Steam (Standard), `1` All, `2` Non-Steam purchase | Nach Kaufquelle filtern. Der Standard lässt Reviews von Steam-Key-Aktivierungen weg |
| `date_range_start`, `date_range_end` | Unix-Zeitstempel | Nur Reviews aus diesem Zeitraum. Setze beide, sonst greift der Filter nicht |
| `playtime_min_hours`, `playtime_max_hours` | Stunden | Nach der Spielzeit des Autors zum Zeitpunkt des Reviews filtern |
| `filter_offtopic_activity` | `true` (Standard), `false` | Off-Topic-Reviews ("Review Bombs") sind standardmäßig ausgeblendet. Übergib `false`, um sie einzuschließen |
| `key` | Publisher Key | Optional. Bringt dir ein höheres Rate Limit (siehe unten) |

Es gibt außerdem `display_language` (Sprache von `review_score_desc`) sowie Steam-Deck- und Hardware-Filter (`primarily_steam_deck`, `hardware_os`, `hardware_gpu`, ...). Die vollständige Liste findest du in der [offiziellen Doku](https://partner.steamgames.com/doc/webapi/IUserReviewsService).

**Antwort-Beispiel:**
```json
{
  "response": {
    "query_summary": {
      "num_reviews": 100,
      "review_score": 8,
      "review_score_desc": "Very Positive",
      "total_positive": 112,
      "total_negative": 14,
      "total_reviews": 126
    },
    "reviews": [
      {
        "recommendationid": "123456789",
        "author": {
          "steamid": "76561198000000000",
          "num_reviews": 4,
          "playtime_forever": 610,
          "playtime_last_two_weeks": 15,
          "playtime_at_review": 480,
          "last_played": 1791224192
        },
        "language": "english",
        "review": "Great game, would play again.",
        "timestamp_created": 1791224243,
        "timestamp_updated": 1791225342,
        "voted_up": true,
        "votes_up": 3,
        "votes_funny": 0,
        "weighted_vote_score": 0.52,
        "comment_count": 1,
        "steam_purchase": true,
        "received_for_free": false,
        "written_during_early_access": false,
        "developer_response": "Thanks for playing!",
        "timestamp_dev_responded": 1791552591,
        "primarily_steam_deck": false,
        "refunded": false
      }
    ],
    "cursor": "AoJwop++/5YDdOvi6QU=",
    "total_matching": 126
  }
}
```

- Die Score-Felder (`review_score`, `review_score_desc`, `total_*`) kommen nur auf der **ersten Seite** zurück, und nur wenn `review_type` `0` ist. Speichere sie von Seite 1.
- Spielzeiten sind in **Minuten**. Zeitstempel sind Unix-Sekunden.
- Wenn keine Reviews mehr kommen, fehlt das `reviews`-Array komplett und `num_reviews` ist `0`.

> ⚠️ **Rate Limit:** Aufrufe ohne Schlüssel teilen sich ein niedrigeres Rate Limit (HTTP 429, wenn du es erreichst), und Antworten können bis zu 10 Minuten gecacht sein. Für ein höheres Limit hängst du `&key={publisherKey}` mit einem Publisher Key für deine App an und rufst `partner.steam-api.com` statt `api.steampowered.com` auf. Mach das nur von deinem Server aus.

> 💡 **`input_json`:** Steams offizielle Doku übergibt die Parameter als ein URL-encodetes JSON-Objekt: `?input_json={"appid":3349960,"filter":1,"languages":["all"]}`. Normale Query-Parameter wie oben funktionieren genauso, nimm einfach, was für dich einfacher ist.

**Alle Reviews abrufen** (JavaScript):

```javascript
async function getAllReviews(appid) {
  const reviews = [];
  let cursor = "*";
  while (true) {
    const params = new URLSearchParams({
      appid,
      filter: 1,            // 1 = Recent (use 1 or 2 when paging through everything)
      "languages[0]": "all",
      purchase_type: 1,     // 1 = All (the default, 0, is Steam purchases only)
      num_per_page: 100,
      cursor,               // URLSearchParams handles the encoding for you
    });
    const res = await fetch(
      `https://api.steampowered.com/IUserReviewsService/GetAppReviews/v1/?${params}`
    );
    if (!res.ok) throw new Error(`HTTP ${res.status}`); // no "success" field anymore
    const { response } = await res.json();
    if (!response.reviews?.length) break; // empty page = you're done
    reviews.push(...response.reviews);
    cursor = response.cursor;
  }
  return reviews;
}
```

<details>
<summary><b>Python-Version</b></summary>

```python
import requests

def get_all_reviews(appid):
    reviews = []
    cursor = "*"
    while True:
        response = requests.get(
            "https://api.steampowered.com/IUserReviewsService/GetAppReviews/v1/",
            params={
                "appid": appid,
                "filter": 1,              # 1 = Recent (use 1 or 2 when paging through everything)
                "languages[0]": "all",
                "purchase_type": 1,       # 1 = All (the default, 0, is Steam purchases only)
                "num_per_page": 100,
                "cursor": cursor,         # requests handles the encoding for you
            },
        )
        response.raise_for_status()       # no "success" field anymore
        data = response.json()["response"]
        if not data.get("reviews"):
            break                         # empty page = you're done
        reviews.extend(data["reviews"])
        cursor = data["cursor"]
    return reviews
```

</details>

#### Migration von /appreviews

Wenn du den alten Store-Endpoint verwendet hast, so stellst du um.

**Die Kurzfassung:** Tausche die URL und die Parameterwerte aus und lies die Daten dann aus `response`.

```diff
- https://store.steampowered.com/appreviews/{appid}?json=1&filter=recent&language=all&purchase_type=all&num_per_page=100
+ https://api.steampowered.com/IUserReviewsService/GetAppReviews/v1/?appid={appid}&filter=1&languages[0]=all&purchase_type=1&num_per_page=100
```

```diff
- const { success, query_summary, reviews, cursor } = await res.json();
- if (success !== 1) throw new Error("Steam error");
+ if (!res.ok) throw new Error(`HTTP ${res.status}`);
+ const { query_summary, reviews = [], cursor } = (await res.json()).response;
```

**Alles, was sich geändert hat:**

| Alt (`store.steampowered.com/appreviews`) | Neu (`IUserReviewsService/GetAppReviews`) |
|---|---|
| `/appreviews/{appid}?json=1` | `/IUserReviewsService/GetAppReviews/v1/?appid={appid}` auf `api.steampowered.com` |
| `filter=all` / `recent` / `updated` | `filter=0` / `1` / `2` (jetzt Zahlen, plus `3` für Funny) |
| `review_type=all` / `positive` / `negative` | `review_type=0` / `1` / `2` |
| `purchase_type=steam` / `all` / `non_steam_purchase` | `purchase_type=0` / `1` / `2` |
| `language=all` | `languages[0]=all` (jetzt eine Liste) |
| `"success": 1` in der Antwort | Entfernt. Prüfe stattdessen den HTTP-Status |
| `query_summary`, `reviews`, `cursor` auf oberster Ebene | In ein `response`-Objekt verpackt |
| `weighted_vote_score` manchmal ein String | Immer eine Zahl |
| `author.personaname`, `profile_url`, `avatar`, `persona_status`, `num_games_owned` | Entfernt. Nutze `author.steamid` |
| `reactions`, `app_release_date` | Entfernt |
| (nicht verfügbar) | Neu: `developer_response`, `timestamp_dev_responded`, `total_matching`, `day_range_used`, plus Datums-, Spielzeit-, Steam-Deck- und Hardware-Filter |

> Steams Migrationshinweise führen `refunded` ebenfalls als neues Feld auf. Der alte Endpoint hat es in unseren Tests bereits zurückgegeben. Wenn dein Code es liest, ändert sich also nichts.

> ⚠️ **Alte Parameter scheitern lautlos.** Wenn du nur die URL austauschst, werden `filter=recent` und `language=all` ohne Fehler ignoriert. Du bekommst HTTP 200 mit ausschließlich englischen Reviews, sortiert nach Helpful und meist nur aus den letzten 30 Tagen. Konvertiere jeden Parameter.

Steams eigene Migrationshinweise: [Migrating from /appreviews](https://partner.steamgames.com/doc/webapi/IUserReviewsService#migrating)

### News

```
GET https://api.steampowered.com/ISteamNews/GetNewsForApp/v2/?appid={appid}&count=10
```

| Parameter | Standard | Beschreibung |
|-----------|----------|-------------|
| `count` | 20 | Anzahl der Artikel |
| `maxlength` | voll | Inhalt kürzen (0 = vollständig) |
| `feeds` | alle | Komma-getrennte Feed-Namen zum Filtern |

### Achievement-Prozentsätze

```
GET https://api.steampowered.com/ISteamUserStats/GetGlobalAchievementPercentagesForApp/v2/?gameid={appid}
```

Gibt globale Freischaltungsprozentsätze für jedes Achievement zurück. Nützlich zum Balancing der Achievement-Schwierigkeit oder zum Anzeigen von Community-Statistiken.

### Spielschema (Statistiken & Achievement-Liste)

```
GET https://api.steampowered.com/ISteamUserStats/GetSchemaForGame/v2/?appid={appid}&key={webApiKey}
```

Gibt alle für das Spiel definierten Statistiken und Achievements mit Anzeigenamen und Beschreibungen zurück. Du brauchst das, um Statistik-Namen zu entdecken, bevor du `GetGlobalStatsForGame` aufrufst.

> Hinweis: Das ist der einzige öffentlich(ish) Endpoint, der von einem Web API Key profitiert.

---

## Endpoints: Verkäufe & Umsatz (Financial/Publisher-Schlüssel)

Alle Endpoints unten verwenden `partner.steam-api.com` und benötigen entweder einen Financial Key oder einen Publisher Key mit aktivierter **"Sales Data"**-Berechtigung.

> 🔒 **Wichtig:** Diese Aufrufe müssen von einem Server aus erfolgen. Exponiere deinen Publisher- oder Financial Key niemals in Client-Code, Spiel-Builds oder öffentlichen Repositories.

### GetDetailedSales

```
GET https://partner.steam-api.com/IPartnerFinancialsService/GetDetailedSales/v001/
    ?key={key}
    &date={YYYY-MM-DD}
    &highwatermark_id=0
```

Gibt **alle Verkäufe über alle deine Apps für ein einzelnes Datum** zurück. Es gibt keinen pro-App-Endpoint. Du filterst nach `primary_appid` in deinem Code.

**Parameter:**
| Parameter | Erforderlich | Beschreibung |
|-----------|-------------|-------------|
| `key` | Ja | Dein Financial oder Publisher Key |
| `date` | Ja | `YYYY-MM-DD`, wird als **Pacific Time** interpretiert (nicht UTC!) |
| `highwatermark_id` | Ja | Starte bei `0`. Bei mehr Daten enthält die Antwort `max_id`. Zum Paginieren verwenden |

**Antwort-Beispiel:**
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

Jede Zeile ist ein Verkauf oder eine Rückerstattung, aufgeschlüsselt nach Datum, Land, Plattform und Paket.

> ⚠️ **Achtung:** Beachte, dass `gross_sales_usd`, `net_sales_usd` usw. **Strings** sind, keine Zahlen. Du musst sie konvertieren: `parseFloat(item.gross_sales_usd)` in JavaScript, `float(item["gross_sales_usd"])` in Python.

### GetAppWishlistReporting

```
GET https://partner.steam-api.com/IPartnerFinancialsService/GetAppWishlistReporting/v001/
    ?key={key}
    &appid={appid}
    &date={YYYY-MM-DD}
```

Gibt Wunschlisten-Aktivität für eine bestimmte App an einem bestimmten Datum zurück.

**Parameter:**
| Parameter | Erforderlich | Beschreibung |
|-----------|-------------|-------------|
| `key` | Ja | Dein Financial oder Publisher Key |
| `appid` | Ja | Die App ID deines Spiels |
| `date` | Ja | `YYYY-MM-DD`, in **GMT** (nicht Pacific Time, ja, anders als bei Verkäufen) |

> ⚠️ Gestern ist das aktuellste Datum mit Daten. Die heutigen Daten sind noch nicht verfügbar.

**Antwort-Beispiel:**
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

`app_min_date` zeigt das früheste Datum an, für das Daten vorhanden sind. Frage keine früheren Daten an. Du bekommst nur leere Antworten.

### GetChangedDatesForPartner

```
GET https://partner.steam-api.com/IPartnerFinancialsService/GetChangedDatesForPartner/v001/
    ?key={key}
    &highwatermark=0
```

Gibt Daten zurück, die neue oder aktualisierte Finanzdaten haben. Nutze das für effiziente Synchronisation. Hole nur Daten ab, die in dieser Liste erscheinen, statt jeden Tag blind abzufragen.

```json
{
  "response": {
    "dates": ["2026/04/01", "2026/04/02", "2026/04/03"]
  }
}
```

> ⚠️ **Datumsformat-Falle:** Diese Daten verwenden **Schrägstriche** (`YYYY/MM/DD`), aber alle anderen Endpoints verwenden **Bindestriche** (`YYYY-MM-DD`). Du musst konvertieren: `date.replace(/\//g, '-')`.

---

## Endpoints: Nur Publisher-Schlüssel

Diese benötigen einen Publisher Key, der mit der App verbunden ist. Sie verwenden `partner.steam-api.com`.

### GetGlobalStatsForGame

```
GET https://partner.steam-api.com/ISteamUserStats/GetGlobalStatsForGame/v1/
    ?key={key}
    &appid={appid}
    &count=1
    &name[0]=stat_name
```

Gibt aggregierte globale Statistiken mit optionaler Datumsbereichs-Filterung zurück.

| Parameter | Beschreibung |
|-----------|-------------|
| `name[0]`, `name[1]`, usw. | Statistik-Namen zum Abrufen |
| `count` | Anzahl angeforderter Statistiken |
| `startdate`, `enddate` | Optionale Unix-Zeitstempel für tägliche Aggregate |

> **Voraussetzung:** Deine App muss Statistiken (Stats) in **Steamworks → App Admin → Stats & Achievements** definiert haben. Du musst die Statistik-Namen kennen. Verwende `GetSchemaForGame` um sie zu entdecken.

### GetPartnerAppListForWebAPIKey

```
GET https://partner.steam-api.com/ISteamApps/GetPartnerAppListForWebAPIKey/v2/?key={key}
```

Gibt alle Apps zurück, auf die dein Publisher Key Zugriff hat. Sehr nützlich um zu überprüfen, ob dein Schlüssel richtig eingerichtet ist.

### GetPlayersBanned

```
GET https://partner.steam-api.com/ISteamApps/GetPlayersBanned/v1/?key={key}&appid={appid}
```

Gibt eine Liste der gesperrten Spieler für dein Spiel zurück.

### GetLeaderboardsForGame

```
GET https://partner.steam-api.com/ISteamLeaderboards/GetLeaderboardsForGame/v2/?key={key}&appid={appid}
```

Gibt alle definierten Bestenlisten für dein Spiel zurück.

> **Voraussetzung:** Bestenlisten müssen zuerst in **Steamworks → App Admin → Leaderboards** erstellt werden.

### ISteamMicroTxn/GetReport

```
GET https://partner.steam-api.com/ISteamMicroTxn/GetReport/v5/
    ?key={key}
    &appid={appid}
    &type=GAMESALES
    &time={RFC3339}
    &maxresults=1000
```

Gibt Mikrotransaktions-Berichte zurück. Berichtstypen: `GAMESALES`, `STEAMSTORESALES`, `SETTLEMENT`, `CHARGEBACK`, `SUBSCRIPTION`.

> **Voraussetzung:** Deine App muss Steam-Mikrotransaktionen verwenden.

---

## Regionale Preisabfrage

Um den Preis deines Spiels in verschiedenen Regionen zu prüfen, rufe die Store API mit einem Ländercode auf:

```bash
# USA (Standard)
curl "https://store.steampowered.com/api/appdetails?appids={appid}&cc=us"

# Eurozone
curl "https://store.steampowered.com/api/appdetails?appids={appid}&cc=de"

# Türkiye
curl "https://store.steampowered.com/api/appdetails?appids={appid}&cc=tr"

# China
curl "https://store.steampowered.com/api/appdetails?appids={appid}&cc=cn"

# Brasilien
curl "https://store.steampowered.com/api/appdetails?appids={appid}&cc=br"

# Japan
curl "https://store.steampowered.com/api/appdetails?appids={appid}&cc=jp"
```

Das `price_overview`-Objekt in der Antwort enthält:
- `currency`: z.B. `"USD"`, `"EUR"`, `"TRY"`, `"CNY"`
- `initial`: Grundpreis in Cent
- `final`: aktueller Preis nach Rabatt, in Cent
- `discount_percent`: aktiver Rabattprozentsatz

## Steam-Sprachcodes (Language Codes)

Steam verwendet eigene Sprachcodes für API-Aufrufe und Store-Seiten. Einige davon sind nicht standardkonform (`schinese`, `tchinese`, `brazilian`, `koreana`, `latam`), daher kannst du nicht einfach ISO-Codes verwenden.

| Englischer Name | Lokaler Name | API-Sprachcode | Web-API-Code |
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

> **Quelle:** [Steamworks Sprach-Dokumentation (Languages Documentation)](https://partner.steamgames.com/doc/store/localization/languages)
>
> **Hinweis:** API-Sprachcodes werden mit den clientseitigen Steamworks-APIs verwendet. Web-API-Sprachcodes werden mit der Steamworks Web API verwendet. Eine Ausnahme: Der [Reviews](#reviews)-Endpoint (`GetAppReviews`) verwendet **API-Sprachcodes** (`german`, nicht `de`). Übergibst du `de`, bekommst du null Reviews. Weitere Sprachen (Afrikaans, Albanisch, Hebräisch, Hindi usw.) sind nur für die Store-Seiten-Sprachauswahl verfügbar und werden in APIs nicht unterstützt.

> 💡 **Profi-Tipp:** Du kannst jede Steam-Store-Seite in einer anderen Sprache anzeigen, indem du `?l=` an die URL anhängst. Zum Beispiel: [`store.steampowered.com/app/3349960/okeygg/?l=turkish`](https://store.steampowered.com/app/3349960/okeygg/?l=turkish&utm_source=steamworks_api_simplified) zeigt die [okey.gg](https://store.steampowered.com/app/3349960?utm_source=steamworks_api_simplified) Store-Seite auf Türkisch. Verwende die API-Sprachcodes aus der Tabelle oben (nicht die Web-API-Codes).

---

## Pakete und Bundles (Packages and Bundles)

Steam verkauft Apps nicht direkt. Jeder Kauf läuft über ein **Paket** (Package, auch "Sub" genannt). Auch wenn dein Spiel kein DLC, keine Editionen und keine Bundles hat, hat es mindestens ein Standard-Paket, das automatisch erstellt wurde.

**Warum das wichtig ist:** Einige Steamworks-Funktionen (wie Rückerstattungsdaten) verwenden die **Paket-ID** (Package ID) statt der App ID. Wenn du nach deinen Rückerstattungsstatistiken suchst und deine App ID in der URL nicht funktioniert, brauchst du wahrscheinlich die Paket-ID.

**So findest du deine Paket-ID (Package ID):**
1. Gehe zu [partner.steamgames.com/apps/associated/{appid}](https://partner.steamgames.com/apps/associated/3349960)
2. Schaue unter **"All Associated Packages"** (Alle zugehörigen Pakete)
3. Dein Standard-Paket heißt normalerweise "{Spielname} for Steam" oder ähnlich

Zum Beispiel hat [okey.gg](https://store.steampowered.com/app/3349960?utm_source=steamworks_api_simplified) die App ID `3349960` und die Standard-Paket-ID `1185712`.

**Bundles** sind anders als Pakete. Ein Bundle gruppiert mehrere Pakete mit Rabatt zusammen (z.B. eine "Complete Edition" mit Basisspiel + allen DLCs). Bundles haben eine eigene Bundle-ID und werden unter **Steamworks → Store Page → Bundles** verwaltet. Die Bundle-Preise berechnen sich aus den kombinierten Paketpreisen minus dem Bundle-Rabatt.

> Mehr dazu: [Steamworks Paket-Dokumentation (Packages Documentation)](https://partner.steamgames.com/doc/store/application/packages)

---

## Daten, die NICHT per API verfügbar sind (Data NOT Available via API)

Manche Daten gibt es nur im Steamworks Partner Portal. Es gibt keine API-Endpoints dafür. Valve stellt sie einfach nicht bereit.

| Daten | Wo zu finden |
|-------|-------------|
| Store-Seiten-Traffic | [partner.steamgames.com/apps/navtrafficstats/{appid}](https://partner.steamgames.com/apps/navtrafficstats/3349960) |
| UTM-Kampagnen-Analytik | [partner.steamgames.com/apps/utmtrafficstats/{appid}](https://partner.steamgames.com/apps/utmtrafficstats/3349960) |
| Rückerstattungsdetails & Gründe (Refund Data) | [partner.steampowered.com/package/refunds/{packageid}/](https://partner.steampowered.com/package/refunds/1185712/) |

Diese Portal-Seiten bieten CSV-Export (CSV Export), falls du die Daten woanders brauchst.

> ⚠️ **Rückerstattungen verwenden die Paket-ID (Package ID), nicht die App ID.** Steam organisiert Käufe nach Paketen (Packages/Subs), nicht nach App ID. Um deine Paket-ID zu finden, gehe zu [partner.steamgames.com/apps/associated/{appid}](https://partner.steamgames.com/apps/associated/3349960). Zum Beispiel hat [okey.gg](https://store.steampowered.com/app/3349960?utm_source=steamworks_api_simplified) die App ID `3349960`, aber die Paket-ID ist `1185712`.
>
> Mehr dazu: [Steamworks Paket-Dokumentation (Packages Documentation)](https://partner.steamgames.com/doc/store/application/packages)

---

## Häufige Stolperfallen

Dinge, die dich erwischen, wenn dich niemand vorwarnt. Betrachte dich als gewarnt.

### Datumsformate sind inkonsistent

| Endpoint | Datumsformat | Zeitzone |
|----------|-------------|----------|
| `GetDetailedSales` | `YYYY-MM-DD` (Bindestriche) | Pacific Time |
| `GetAppWishlistReporting` | `YYYY-MM-DD` (Bindestriche) | GMT |
| `GetChangedDatesForPartner` | `YYYY/MM/DD` (Schrägstriche) | - |

Ja, Verkaufsdaten verwenden Pacific Time und Wunschlistendaten verwenden GMT. Und geänderte Daten verwenden Schrägstriche, während alles andere Bindestriche verwendet. Willkommen bei Steam.

### Finanzwerte sind Strings, keine Zahlen

Felder wie `gross_sales_usd`, `net_sales_usd`, `base_price` kommen als `"9.9900"` (String) zurück, nicht als `9.99` (Zahl). Immer konvertieren:

```javascript
// ❌ Falsch: vergleicht Strings
if (item.gross_sales_usd > 0) { ... }

// ✅ Richtig
if (parseFloat(item.gross_sales_usd) > 0) { ... }
```

### Leere Antwort ≠ Fehler

Wenn dein Publisher Key gültig ist, aber die "Sales Data"-Berechtigung fehlt, gibt Steam zurück:

```json
{ "response": {} }
```

Kein Fehlercode. Keine Fehlermeldung. Einfach... nichts. Wenn du leere Antworten von `IPartnerFinancialsService` bekommst, prüfe die Berechtigungen deines Schlüssels in Steamworks.

### Serverless + IP-Whitelist passen nicht zusammen

Wenn du auf Vercel, AWS Lambda, Cloudflare Workers oder einer anderen Serverless-Plattform deployst, ändert sich die IP deines Servers mit jeder Anfrage. Wenn bei deinem Steam Key IP-Whitelisting aktiviert ist, schlägt jede Anfrage fehl.

**Lösung:** Entferne IP-Einschränkungen von deinem Schlüssel in Steamworks, oder verwende einen Proxy mit fester IP.

### Nicht jedes Datum hat Verkaufsdaten

An manchen Tagen verkauft sich dein Spiel einfach nicht. Die API gibt leere Ergebnisse für diese Daten zurück. Das ist normal, kein Fehler. Verwende `GetChangedDatesForPartner` um herauszufinden, welche Daten tatsächlich Daten haben, statt blind jeden Tag abzufragen.

### Wunschlistendaten haben ein Mindestdatum

Jede App hat ein `app_min_date` (wird in der Wunschlisten-Antwort zurückgegeben). Daten vor diesem Datum existieren nicht. Verschwende keine Anfragen auf frühere Daten.

### Die Reviews-API versteckt standardmäßig die meisten Reviews

`GetAppReviews` gibt nur **englische** Reviews von **Steam-Käufen** zurück, wenn du nichts anderes angibst. Wenn dein Spiel Spieler in anderen Sprachen hat oder du Keys verteilt hast, siehst du viel weniger Reviews als auf deiner Store-Seite. Übergib immer `languages[0]=all` und `purchase_type=1`.

Außerdem scheitert sie bei falschen Eingaben lautlos. Alte String-Werte wie `filter=recent` werden ignoriert (du bekommst die Standard-Sortierung Helpful), Web-API-Sprachcodes wie `de` liefern null Reviews, und ein nicht encodeter Cursor mit einem `+` darin liefert `{"response":{}}`.

---

## Fehler-Referenz

| Was du siehst | Was es bedeutet | Wie du es behebst |
|---|---|---|
| HTTP 403 + "Access is denied" HTML-Seite | Falscher Schlüsseltyp für diesen Host | Verwende einen Publisher/Financial Key für `partner.steam-api.com`. Ein normaler Web API Key funktioniert nicht. |
| HTTP 200 + `{"response":{}}` | Schlüssel ist gültig, aber Berechtigung fehlt | Aktiviere "Sales Data"-Berechtigung in der Publisher-Gruppe in Steamworks |
| HTTP 200 + `{"response":{"result":8}}` | Keine Statistiken/Daten konfiguriert | Erstelle zuerst Statistiken, Achievements oder Bestenlisten in Steamworks |
| Aufrufe an `store.steampowered.com/appreviews` funktionieren nicht mehr | Steam hat den Endpoint am 22. Oktober 2026 abgeschaltet | Wechsle zu `GetAppReviews`. Siehe [Migration von /appreviews](#migration-von-appreviews) |
| HTTP 200 + viel weniger Reviews als auf deiner Store-Seite | `GetAppReviews` liefert standardmäßig nur englische Reviews von Steam-Käufen | Füge `languages[0]=all&purchase_type=1` hinzu |
| HTTP 200 + `{"response":{}}` ab Seite 2 der Reviews | Cursor wurde nicht URL-encodet | Encode den Cursor (`encodeURIComponent` / `URLSearchParams`) |
| HTTP 429 | Rate Limit erreicht | Langsamer machen. Caching hinzufügen. |
| Verbindungs-Timeout bei Partner API | IP durch Whitelist blockiert | Entferne IP-Einschränkungen vom Schlüssel, oder füge deine Server-IP hinzu |

---

## Steamworks-Einrichtungs-Checkliste

Manche Endpoints geben nichts zurück, bis du das entsprechende Feature in Steamworks konfiguriert hast. Hier ist was einzurichten ist und was es freischaltet:

| Feature | Wo konfigurieren | Was es freischaltet |
|---------|-----------------|---------------------|
| Statistiken | Steamworks → App Admin → Stats & Achievements → Stats | `GetGlobalStatsForGame`: aggregierte Spielstatistiken mit Zeitbereichen |
| Achievements | Steamworks → App Admin → Stats & Achievements → Achievements | `GetGlobalAchievementPercentagesForApp`: Freischaltungsprozentsätze |
| Bestenlisten | Steamworks → App Admin → Leaderboards | `GetLeaderboardsForGame` + `GetLeaderboardEntries` |
| Mikrotransaktionen | Steamworks → App Admin → Microtransaction Configuration | `ISteamMicroTxn/GetReport`: Transaktionsberichte |
| Steam Inventory | Steamworks → App Admin → Steam Inventory Service | `IInventoryService`-Endpoints |

---

## Offizielle Quellen

- [Steamworks Web API Übersicht](https://partner.steamgames.com/doc/webapi_overview)
- [Web API Authentifizierung & Schlüsseltypen](https://partner.steamgames.com/doc/webapi_overview/auth)
- [IPartnerFinancialsService](https://partner.steamgames.com/doc/webapi/IPartnerFinancialsService)
- [Wunschlisten-Berichterstattung](https://partner.steamgames.com/doc/marketing/wishlist/reporting)
- [Wunschlisten-Daten-API-Ankündigung](https://store.steampowered.com/news/group/4145017/view/499474120884358023)
- [IUserReviewsService](https://partner.steamgames.com/doc/webapi/IUserReviewsService)
- [Ankündigung zur Änderung der User-Reviews-API](https://store.steampowered.com/news/group/4145017/view/676258795703241580)
- [Vollständige Interface-Liste](https://partner.steamgames.com/doc/webapi)

---

## KI-Agenten-Skill

Dieser Guide ist auch als KI-Agenten-Skill verfügbar. Wenn du [Claude Code](https://claude.ai/code) oder ähnliche KI-Coding-Tools verwendest, kannst du den [`steamworks-api-specialist`](../steamworks-api-specialist/SKILL.md) Skill installieren, um deinem KI-Assistenten tiefes Wissen über die Steam Web API zu geben — Schlüsseltypen, Endpoint-Details, häufige Stolperfallen und Fehlerbehebungsabläufe — damit er dir bei der Integration schneller helfen kann.

---

## Mitwirken

Einen Fehler gefunden? Einen Endpoint entdeckt, den wir übersehen haben? Eine Übersetzung beisteuern?

Wir freuen uns über Hilfe. Schau dir unseren [Mitwirkungs-Guide](../.github/CONTRIBUTING.md) für Details an.

**Kurzversion:**
1. Forke dieses Repository
2. Nimm deine Änderungen vor
3. Erstelle einen Pull Request

Übersetzungen kommen nach `translations/README.{sprachcode}.md`. Schau dir [bestehende Übersetzungen](./) als Referenz an.

Wenn dir dieser Guide Zeit gespart hat, überleg dir einen ⭐ zu geben. Das hilft anderen Indie-Devs, ihn zu finden.

---

## Lizenz

[CC0 1.0 Universal (Public Domain)](../LICENSE). Mach damit was du willst. Kopieren, ändern, in dein Projekt einbauen, einen Kurs darum herum verkaufen. Keine Namensnennung erforderlich (wird aber immer geschätzt 🙏).

---

<p align="center">
  <br>
  Mit ❤️ erstellt von <a href="https://yilmaz.games?utm_source=steamworks_api_simplified">yilmaz.games</a>
  <br><br>
  Wir haben diesen Guide erstellt, während wir Steam-APIs für unser Spiel <a href="https://store.steampowered.com/app/3349960?utm_source=steamworks_api_simplified">okey.gg</a> integriert haben.<br>
  Wenn er dir geholfen hat, bedeutet ein <a href="https://store.steampowered.com/app/3349960?utm_source=steamworks_api_simplified">Wunschlisten-Eintrag</a> einem kleinen Studio die Welt 🙏
  <br><br>
  <sub>Letzte Aktualisierung: Oktober 2026</sub>
</p>
