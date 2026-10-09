---
name: steamworks-api-specialist
description: >
  Use this skill whenever the user wants to work with Steam Web APIs — getting API keys, fetching
  player counts, app details, reviews, news, sales/revenue data, wishlist metrics, achievements,
  leaderboards, regional pricing, or building custom applications on top of Steam data. Also use
  this skill when the user is debugging Steam API issues (empty responses, 403 errors, wrong data),
  choosing between key types, or figuring out which Steam API host to use. Triggers include
  "Steam API", "Steamworks", "Steam Web API", "Steam sales data", "Steam wishlist",
  "Steam player count", "Steam reviews", "Steam pricing", "partner.steam-api.com",
  "IPartnerFinancialsService", "Steam API key", "publisher key", "financial key",
  "appdetails", "GetDetailedSales", "Steam current players", "Steam achievements API",
  "Steam leaderboard API", "Steam microtransactions", "GetAppReviews", "IUserReviewsService",
  "appreviews". Use this even when the user doesn't
  say "API" explicitly but clearly needs Steam data programmatically — e.g., "how do I get
  my game's sales numbers in code" or "I want to build a dashboard for my Steam game."
---

# Steamworks Web API Specialist

Help users integrate with Steam's Web APIs. This skill covers authentication, all public and partner endpoints, and the gotchas that cause most developer frustration.

Source: [steamworks-api-simplified](https://github.com/yilmaz-games/steamworks-api-simplified)

---

## How to help the user

### Start by understanding what they need

Steam API questions fall into three buckets. Figure out which one the user is in:

1. **"I want to get data"** — They know what data they want but not how to get it. Guide them to the right endpoint, key type, and host.
2. **"Something isn't working"** — They're calling an API and getting errors or empty data. Use the diagnostic flow below.
3. **"I want to build something"** — They want a dashboard, bot, or integration. Help them plan which endpoints they need, then walk through each one.

### The three mistakes that cause 90% of frustration

Before diving into specifics, internalize these. They're the reason most developers get stuck:

1. **Wrong host.** Public data uses `api.steampowered.com`. Business data (sales, wishlists) uses `partner.steam-api.com`. Using the wrong one either returns public data when they wanted private data, or gives a 403.
2. **Missing "Sales Data" permission.** A publisher key without this permission returns `{"response":{}}` for financial endpoints — no error, no message, just empty. This is extremely confusing and the user may not realize what's wrong.
3. **Financial values are strings.** Fields like `gross_sales_usd` return `"9.9900"` not `9.99`. Comparing strings instead of numbers causes silent bugs.

Always check for these first when something goes wrong.

---

## Diagnostic flow: when the user has a problem

Follow this sequence when a user reports an API issue:

```
What's the symptom?
│
├─ HTTP 403 + HTML "Access is denied"
│  → They're using a regular Web API Key on partner.steam-api.com
│  → Fix: They need a Publisher or Financial key
│
├─ HTTP 200 + {"response":{}}  (empty, no error)
│  → Key is valid but missing permissions
│  → Ask: "Did you enable 'Sales Data' permission when creating the publisher key?"
│  → Fix: Recreate or edit the key in Steamworks → Users & Permissions → Manage Groups
│
├─ HTTP 200 + {"response":{"result":8}}
│  → The feature isn't configured in Steamworks yet
│  → Ask: "Have you set up stats/achievements/leaderboards in Steamworks App Admin?"
│
├─ Connection timeout on partner API
│  → IP whitelist is blocking them
│  → Ask: "Are you on a serverless platform (Vercel, Lambda, Cloudflare Workers)?"
│  → Fix: Remove IP restrictions from the key, or use a fixed-IP proxy
│
├─ HTTP 429
│  → Rate limited. Suggest caching.
│  → Store API limit: ~200 requests per 5 minutes
│
├─ Old reviews endpoint (store.steampowered.com/appreviews) stopped working
│  → Steam disabled it on October 22, 2026
│  → Fix: Migrate to IUserReviewsService/GetAppReviews (see Reviews below)
│
├─ Reviews: far fewer than the store page shows, or English only
│  → GetAppReviews defaults to English-only and Steam-purchase-only
│  → Fix: Add languages[0]=all and purchase_type=1
│  → Also check: Old string params (filter=recent, language=all) are silently ignored
│  → Also check: Web API language codes (tr) return nothing; use API codes (turkish)
│
├─ Reviews: {"response":{}} on page 2+
│  → Cursor wasn't URL-encoded (it can contain +, /, =)
│
├─ Getting data but it seems wrong or empty for certain dates
│  → Check: Are they using the right timezone? (Sales = Pacific Time, Wishlists = GMT)
│  → Check: Are they requesting a date before app_min_date for wishlist data?
│  → Check: Some days genuinely have no sales — use GetChangedDatesForPartner to find valid dates
│
└─ Getting data but numbers seem wrong
   → Check: Are they parsing financial strings as numbers? ("9.9900" → parseFloat)
   → Check: Are they using the right date format? (dashes for most, slashes for GetChangedDatesForPartner)
```

---

## Key type decision guide

When the user needs to choose an API key:

| They want... | Key type | How to get it |
|---|---|---|
| Public data (players, reviews, news, pricing) | No key needed | Just call the endpoint |
| Public data + player profiles/friend lists | Web API Key | steamcommunity.com/dev/apikey |
| Sales, revenue, or wishlist data for one game | Publisher Key (with "Sales Data" permission) | Steamworks → Users & Permissions → Manage Groups → Create WebAPI Key |
| Sales/wishlist data across ALL their games | Financial API Key | Steamworks → Manage Groups → Create Financial API Group |
| Stats, leaderboards, bans, microtransactions | Publisher Key (with relevant permission) | Same as above, different permission checkboxes |

**Important nuance:** A Financial API Group has no users and no apps — it's purely a financial data access key. A Publisher key with "Sales Data" can access the same financial endpoints, but only for apps in its group.

---

## API hosts

| Host | When to use | Protocol | Key? |
|------|-------------|----------|------|
| `api.steampowered.com` | Public game data | HTTP or HTTPS | Usually no |
| `store.steampowered.com` | Store page data (app details, pricing) | HTTPS | No |
| `partner.steam-api.com` | Business data (sales, wishlists, stats) | **HTTPS only** | Always |

Partner API IP ranges (for firewall whitelisting): `208.64.200.0/22`, `155.133.239.0/24`

---

## Endpoint reference

### Public endpoints (no key required)

**Current online players**
```
GET https://api.steampowered.com/ISteamUserStats/GetNumberOfCurrentPlayers/v1/?appid={appid}
```
Returns: `{ "response": { "player_count": N, "result": 1 } }`

**App details & pricing**
```
GET https://store.steampowered.com/api/appdetails?appids={appid}
GET https://store.steampowered.com/api/appdetails?appids={appid}&cc={country_code}
```
Returns store page data. `price_overview` has `currency`, `initial` (cents), `final` (cents), `discount_percent`.
Rate limit: ~200 req / 5 min — cache aggressively.

**Reviews** (`IUserReviewsService/GetAppReviews`)
```
GET https://api.steampowered.com/IUserReviewsService/GetAppReviews/v1/?appid={appid}&filter=1&languages[0]=all&purchase_type=1&num_per_page=100
```
- Replaces `store.steampowered.com/appreviews/{appid}?json=1`, which Steam disabled on **October 22, 2026**
- Params are **numbers**, not strings: `filter` (0=Helpful default, 1=Recent, 2=Updated, 3=Funny), `review_type` (0=All, 1=Positive, 2=Negative), `purchase_type` (0=Steam default, 1=All, 2=Non-Steam)
- `languages[0]`, `languages[1]`, ... take API language codes (`english`, `turkish`) or `all`. **Default is English only**
- `num_per_page` (1-100, default 20), `cursor` (`*` first, then from response, **URL-encode it**)
- Extra filters: `date_range_start`/`date_range_end` (Unix, set both), `playtime_min_hours`/`playtime_max_hours`, `filter_offtopic_activity=false` to include review bombs, `display_language` (language of `review_score_desc`), Steam Deck and hardware filters
- Helpful (`filter=0`) only returns reviews from the last `day_range` days (default 30, max 365, 0 = no limit). Steam widens the window only if it's empty, and reports it in `day_range_used`. Use `filter=1` or `2` to get every review
- Response is wrapped in `response`. No `success` field, so check HTTP status. Score fields (`review_score`, `total_*`) only on page 1 when `review_type=0`. Empty page = no `reviews` key
- No key needed. Anonymous calls share a lower rate limit (429) and may be cached for 10 min. For a higher limit, add a publisher key and call `partner.steam-api.com` from a server
- Migration from old endpoint: `language=all` → `languages[0]=all`, string enums → numbers, `author.personaname`/`avatar`/`profile_url`/`num_games_owned` removed (use `author.steamid`), `weighted_vote_score` is always a number, new `developer_response` and `total_matching` fields. Steam's notes list `refunded` as new, but the old endpoint already returned it

**News**
```
GET https://api.steampowered.com/ISteamNews/GetNewsForApp/v2/?appid={appid}&count=10
```
Params: `count`, `maxlength` (0=full), `feeds` (comma-separated).

**Achievement percentages**
```
GET https://api.steampowered.com/ISteamUserStats/GetGlobalAchievementPercentagesForApp/v2/?gameid={appid}
```

**Game schema** (stat/achievement definitions — Web API Key helps here)
```
GET https://api.steampowered.com/ISteamUserStats/GetSchemaForGame/v2/?appid={appid}&key={key}
```

### Financial endpoints (Financial or Publisher Key + "Sales Data" permission)

**GetDetailedSales** — all sales across all apps for one date
```
GET https://partner.steam-api.com/IPartnerFinancialsService/GetDetailedSales/v001/?key={key}&date={YYYY-MM-DD}&highwatermark_id=0
```
- Returns ALL apps. Filter by `primary_appid` in code.
- `date` uses **Pacific Time**
- Paginate: start with `highwatermark_id=0`, use `max_id` from response for next page
- Financial values are **strings** — always convert to numbers

**GetAppWishlistReporting** — wishlist activity for one app on one date
```
GET https://partner.steam-api.com/IPartnerFinancialsService/GetAppWishlistReporting/v001/?key={key}&appid={appid}&date={YYYY-MM-DD}
```
- `date` uses **GMT** (different from sales!)
- Yesterday is the most recent available date
- Response includes `app_min_date` — don't request before it

**GetChangedDatesForPartner** — which dates have new/updated financial data
```
GET https://partner.steam-api.com/IPartnerFinancialsService/GetChangedDatesForPartner/v001/?key={key}&highwatermark=0
```
- Uses **slashes** in dates (`YYYY/MM/DD`) — convert to dashes for other endpoints

### Publisher-only endpoints

**GetGlobalStatsForGame**
```
GET https://partner.steam-api.com/ISteamUserStats/GetGlobalStatsForGame/v1/?key={key}&appid={appid}&count=1&name[0]=stat_name
```
Prereq: Stats defined in Steamworks. Use `GetSchemaForGame` to discover stat names.

**GetPartnerAppListForWebAPIKey** — verify key setup
```
GET https://partner.steam-api.com/ISteamApps/GetPartnerAppListForWebAPIKey/v2/?key={key}
```

**GetPlayersBanned**
```
GET https://partner.steam-api.com/ISteamApps/GetPlayersBanned/v1/?key={key}&appid={appid}
```

**GetLeaderboardsForGame**
```
GET https://partner.steam-api.com/ISteamLeaderboards/GetLeaderboardsForGame/v2/?key={key}&appid={appid}
```
Prereq: Leaderboards created in Steamworks.

**ISteamMicroTxn/GetReport**
```
GET https://partner.steam-api.com/ISteamMicroTxn/GetReport/v5/?key={key}&appid={appid}&type=GAMESALES&time={RFC3339}&maxresults=1000
```
Types: GAMESALES, STEAMSTORESALES, SETTLEMENT, CHARGEBACK, SUBSCRIPTION.

---

## Date format cheat sheet

This inconsistency causes real bugs. Reference this when helping with any date-related code:

| Endpoint | Format | Timezone |
|----------|--------|----------|
| GetDetailedSales | `YYYY-MM-DD` (dashes) | Pacific Time |
| GetAppWishlistReporting | `YYYY-MM-DD` (dashes) | GMT |
| GetChangedDatesForPartner | `YYYY/MM/DD` (slashes) | — |

---

## Data that doesn't exist in any API

If the user asks for these, tell them they're only available in the Steamworks web portal (with CSV export):

| Data | Portal URL pattern |
|------|--------------------|
| Store page traffic | `partner.steamgames.com/apps/navtrafficstats/{appid}` |
| UTM campaign analytics | `partner.steamgames.com/apps/utmtrafficstats/{appid}` |
| Refund details & reasons | `partner.steampowered.com/package/refunds/{packageid}/` |

Refund URLs use **Package ID**, not App ID. Every game has at least one package. Find it at `partner.steamgames.com/apps/associated/{appid}`.

---

## Steamworks prerequisites

These endpoints return empty until the feature is configured. If the user gets empty data, ask if they've done the setup:

| Feature | Configure at | Unlocks |
|---------|-------------|---------|
| Stats | App Admin → Stats & Achievements → Stats | GetGlobalStatsForGame |
| Achievements | App Admin → Stats & Achievements → Achievements | GetGlobalAchievementPercentagesForApp |
| Leaderboards | App Admin → Leaderboards | GetLeaderboardsForGame |
| Microtransactions | App Admin → Microtransaction Config | ISteamMicroTxn/GetReport |
| Inventory | App Admin → Steam Inventory Service | IInventoryService |

---

## Security reminder

Never help the user put publisher or financial API keys in client-side code, game builds, or public repositories. These keys access business data and must stay server-side. If the user's code exposes a key, flag it immediately.

---

## Steam language codes

Steam uses non-standard language codes. The surprising ones: `schinese` (Simplified Chinese), `tchinese` (Traditional Chinese), `brazilian` (Brazilian Portuguese), `koreana` (Korean), `latam` (Latin American Spanish). `GetAppReviews` uses these API codes, not ISO codes (`tr` returns nothing). Full table is in the [source guide](https://github.com/yilmaz-games/steamworks-api-simplified#steam-language-codes).

---

## Official references

- [Web API Overview](https://partner.steamgames.com/doc/webapi_overview)
- [Authentication & Key Types](https://partner.steamgames.com/doc/webapi_overview/auth)
- [IPartnerFinancialsService](https://partner.steamgames.com/doc/webapi/IPartnerFinancialsService)
- [IUserReviewsService](https://partner.steamgames.com/doc/webapi/IUserReviewsService)
- [Full Interface List](https://partner.steamgames.com/doc/webapi)
- [Wishlist Reporting](https://partner.steamgames.com/doc/marketing/wishlist/reporting)
- [Packages Documentation](https://partner.steamgames.com/doc/store/application/packages)
