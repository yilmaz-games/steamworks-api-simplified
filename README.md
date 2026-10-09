# steamworks-api-simplified

**The Steam API guide we wish existed.**

A practical guide to the Steamworks Web API covering sales data, wishlist reporting, reviews, player counts, achievements and partner endpoints. Focused on real-world usage, common pitfalls and clear examples for game developers.

🌍 *[Türkçe](translations/README.tr.md) · [Deutsch](translations/README.de.md)*

> 📢 **Heads up:** Steam turns off the old reviews endpoint (`store.steampowered.com/appreviews`) on **October 22, 2026**. If you fetch reviews, see [Migrating from /appreviews](#migrating-from-appreviews).

---

You shipped your game on Steam. Congrats! Now you need data: how many people are playing, what are they saying in reviews, how are sales going, where are your wishlists coming from.

You open Steam's official API docs and... it's a maze. Three different key types. Two different API hosts. Some endpoints need a key, some don't. Date formats change between endpoints. Financial data comes back as strings instead of numbers. The docs assume you already know everything.

We've been there. This guide is everything we learned, organized the way we wish someone had explained it to us.

---

## Table of Contents

- [New to APIs?](#new-to-apis)
- [Quick Start: Which Key Do I Need?](#quick-start-which-key-do-i-need)
- [The Three Types of Steam API Keys](#the-three-types-of-steam-api-keys)
- [API Hosts](#api-hosts)
- [Endpoints: Public Data (No Key Required)](#endpoints-public-data-no-key-required)
- [Endpoints: Sales & Revenue](#endpoints-sales--revenue-financialpublisher-key)
- [Endpoints: Publisher Key Only](#endpoints-publisher-key-only)
- [Regional Pricing Lookup](#regional-pricing-lookup)
- [Steam Language Codes](#steam-language-codes)
- [Packages and Bundles](#packages-and-bundles)
- [Data NOT Available via API](#data-not-available-via-api)
- [Common Gotchas](#common-gotchas)
- [Error Reference](#error-reference)
- [Steamworks Setup Checklist](#steamworks-setup-checklist)
- [Official References](#official-references)
- [AI Agent Skill](#ai-agent-skill)
- [Contributing](#contributing)
- [License](#license)

---

<a name="new-to-apis"></a>
<details>
<summary><b>New to APIs? Start here</b></summary>

If you've only ever worked in a game engine and never called a web API before, no worries. It's simpler than it sounds.

**An API is just a URL.** You visit the URL, and instead of a web page, you get back raw data (usually JSON, basically structured text).

**Try it right now.** Copy this URL and paste it into your browser:

```
https://api.steampowered.com/ISteamUserStats/GetNumberOfCurrentPlayers/v1/?appid=3349960
```

You'll see something like:

```json
{ "response": { "player_count": 42, "result": 1 } }
```

That's it. You just called the Steam API. That URL returns the number of people currently playing [okey.gg](https://store.steampowered.com/app/3349960?utm_source=steamworks_api_simplified) (a tile game by [yilmaz.games](https://yilmaz.games?utm_source=steamworks_api_simplified), hi, that's us 👋).

**In your code**, you'd do the same thing: fetch a URL and read the response. Here's what that looks like:

```javascript
// JavaScript / Node.js
const response = await fetch(
  "https://api.steampowered.com/ISteamUserStats/GetNumberOfCurrentPlayers/v1/?appid=YOUR_APP_ID"
);
const data = await response.json();
console.log(data.response.player_count); // 42
```

```python
# Python
import requests

response = requests.get(
    "https://api.steampowered.com/ISteamUserStats/GetNumberOfCurrentPlayers/v1/",
    params={"appid": "YOUR_APP_ID"}
)
data = response.json()
print(data["response"]["player_count"])  # 42
```

```csharp
// C# / Unity
using var client = new HttpClient();
var response = await client.GetStringAsync(
    "https://api.steampowered.com/ISteamUserStats/GetNumberOfCurrentPlayers/v1/?appid=YOUR_APP_ID"
);
// Parse the JSON string to get player_count
```

```gdscript
# GDScript / Godot
var http = HTTPRequest.new()
add_child(http)
http.request("https://api.steampowered.com/ISteamUserStats/GetNumberOfCurrentPlayers/v1/?appid=YOUR_APP_ID")
# Connect to request_completed signal to handle the response
```

**Useful tools for exploring APIs:**
- Your browser (seriously, just paste URLs)
- [Postman](https://www.postman.com/) or [Insomnia](https://insomnia.rest/), free apps that make it easy to build and test API calls
- `curl` in your terminal: `curl "https://api.steampowered.com/..."`

Now you're ready for the rest of this guide. Every endpoint below works the same way: it's a URL, you call it, you get data back.
</details>

---

## Quick Start: Which Key Do I Need?

Before diving into endpoints, here's what you need for each type of data:

| Data you want | Key type | Where to call |
|---------------|----------|---------------|
| Current online players | No key needed | api.steampowered.com |
| App details, pricing | No key needed | store.steampowered.com |
| Reviews | No key needed | api.steampowered.com |
| News | No key needed | api.steampowered.com |
| Achievement percentages | No key needed | api.steampowered.com |
| **Sales & revenue** | **Financial key** or Publisher key (with Sales Data perm) | partner.steam-api.com |
| **Wishlist data** | **Financial key** or Publisher key (with Sales Data perm) | partner.steam-api.com |
| Game stats (if defined) | Publisher key | partner.steam-api.com |
| Leaderboards (if defined) | Publisher key | partner.steam-api.com |
| Microtransaction reports | Publisher key (with Microtransaction perm) | partner.steam-api.com |
| Banned players | Publisher key | partner.steam-api.com |

> **Notice the pattern:** Free/public data → `api.steampowered.com` (no key). Your private business data → `partner.steam-api.com` (key required). Using the wrong host is the #1 most common mistake.

---

## The Three Types of Steam API Keys

Steam has three different API keys. Yes, three. Here's what each one does and how to get it.

### 1. Web API Key

The basic key anyone with a Steam account can get. You probably don't even need this one. Most public endpoints work without any key at all.

**How to get it:**
1. Go to https://steamcommunity.com/dev/apikey
2. Log in with your Steam account
3. Enter a domain name and register
4. Copy your key

**What it can do:** Access public endpoints on `api.steampowered.com`. Useful for user-specific lookups (player profiles, friend lists, etc.).

**What it can't do:** Access any partner or publisher endpoints. No sales data, no wishlist data, nothing on `partner.steam-api.com`.

### 2. Publisher Web API Key

A key tied to a group of apps in your Steamworks partner account. This is the most flexible key type. You choose exactly which permissions it has.

**How to get it:**
1. Log in to https://partner.steamgames.com
2. Go to **Users & Permissions** → **Manage Groups**
3. Select an existing group or create a new one
4. Assign your apps to the group (if they're not already there)
5. Click **"Create WebAPI Key"** on the group page
6. Select which permissions to enable:
   - **Microtransactions**: transaction reports and management
   - **Sales Data**: sales, revenue, and wishlist data (IPartnerFinancialsService)
   - **Economy**: Steam Inventory Service
   - **General API**: authentication, DLC ownership checks
7. Optionally configure IP whitelisting (see [gotchas](#common-gotchas) if you're on serverless)

**What it can do:** Everything the Web API key does, plus partner endpoints on `partner.steam-api.com`. But only for apps in its group, and only with the permissions you enabled.

> ⚠️ **Want sales/wishlist data?** You must enable the **"Sales Data"** permission when creating the key. Without it, you'll get empty responses with no error message, just `{}`. This is a very common trap.

### 3. Financial API Key

A special-purpose key for financial data only. Gives you unrestricted access to sales and wishlist data across ALL your apps.

**How to get it:**
1. Log in to https://partner.steamgames.com
2. Go to **Users & Permissions** → **Manage Groups**
3. Click **"Create new group"** and select **"Financial API Group"**
4. The key appears on the group page immediately

**What it can do:** Access `IPartnerFinancialsService` endpoints (sales, wishlists) for every app on your partner account, no per-app restrictions.

**How it differs from the publisher key:** A Financial API Group has no users and no apps assigned to it. It exists purely as a financial data access key. A publisher key with "Sales Data" permission can access the same endpoints, but only for apps in its specific group.

**When to use which:**
- **One game?** Publisher key with "Sales Data" permission is fine.
- **Multiple games across different groups?** Financial key is simpler. One key for all financial data.

---

## API Hosts

Steam has two separate API servers. Using the wrong one is the single most common mistake.

| Host | Who can use it | Protocol | Key required? |
|------|----------------|----------|---------------|
| `api.steampowered.com` | Anyone | HTTP or HTTPS | Usually no |
| `partner.steam-api.com` | Publisher/Financial keys only | **HTTPS only** | Always |

**Rule of thumb:** If you're looking at public game data (player counts, reviews, news), use `api.steampowered.com`. If you're looking at your own business data (sales, wishlists, ownership), use `partner.steam-api.com`.

If you need to whitelist partner API IPs (e.g., for a corporate firewall):
- `208.64.200.0/22`
- `155.133.239.0/24`

---

## Endpoints: Public Data (No Key Required)

These endpoints are open to everyone. No API key needed. You can test them in your browser right now.

### Current Online Players

```
GET https://api.steampowered.com/ISteamUserStats/GetNumberOfCurrentPlayers/v1/?appid={appid}
```

Returns the number of players currently in-game.

```json
{ "response": { "player_count": 42, "result": 1 } }
```

> 💡 **Try it:** Paste this in your browser → [`https://api.steampowered.com/ISteamUserStats/GetNumberOfCurrentPlayers/v1/?appid=3349960`](https://api.steampowered.com/ISteamUserStats/GetNumberOfCurrentPlayers/v1/?appid=3349960). That's the live player count for [okey.gg](https://store.steampowered.com/app/3349960?utm_source=steamworks_api_simplified). Replace `3349960` with your own App ID.

### App Details (Store API)

```
GET https://store.steampowered.com/api/appdetails?appids={appid}
```

Returns everything on the store page: name, description, pricing, screenshots, system requirements, release date, genres, categories, developers, publishers.

For pricing in a specific region, add `&cc={country_code}`:

```
GET https://store.steampowered.com/api/appdetails?appids={appid}&cc=us
GET https://store.steampowered.com/api/appdetails?appids={appid}&cc=de
GET https://store.steampowered.com/api/appdetails?appids={appid}&cc=tr
GET https://store.steampowered.com/api/appdetails?appids={appid}&cc=cn
```

The `price_overview` object contains:
- `currency`: e.g., `"USD"`, `"EUR"`, `"TRY"`
- `initial`: base price in cents (e.g., `999` = $9.99)
- `final`: current price in cents (after any discount)
- `discount_percent`: active discount percentage

> ⚠️ **Rate limit:** ~200 requests per 5 minutes. Cache aggressively. Store page data doesn't change often.

### Reviews

> ⚠️ **Changed in October 2026.** Steam disables the old `store.steampowered.com/appreviews/{appid}?json=1` endpoint on **October 22, 2026**. If your code still calls it, see [Migrating from /appreviews](#migrating-from-appreviews) below.

```
GET https://api.steampowered.com/IUserReviewsService/GetAppReviews/v1/?appid={appid}&filter=1&languages[0]=all&purchase_type=1&num_per_page=100
```

Returns a page of public reviews for any app, plus the review score summary. No key needed.

**Parameters:**
| Parameter | Values | Description |
|-----------|--------|-------------|
| `appid` | Your App ID | Required |
| `filter` | `0` Helpful (default), `1` Recent, `2` Updated, `3` Funny | Sort order. Helpful only returns reviews from the last `day_range` days (widened automatically if that window is empty), so use `1` or `2` to get every review |
| `day_range` | Days. Default 30, max 365, `0` = no limit | Helpful only. If the window has no reviews, Steam widens it. `day_range_used` in the response tells you what it actually searched |
| `languages[0]`, `languages[1]`, ... | `all` or [API language codes](#steam-language-codes) (`english`, `turkish`, ...) | **Defaults to English only.** Pass `languages[0]=all` to get every language |
| `num_per_page` | 1–100 | Results per page (default 20) |
| `cursor` | `*` (first page), then value from response | Pagination. **URL-encode it**, since it can contain `+`, `/` and `=` |
| `review_type` | `0` All (default), `1` Positive, `2` Negative | Filter by sentiment |
| `purchase_type` | `0` Steam (default), `1` All, `2` Non-Steam purchase | Filter by purchase source. The default skips reviews from Steam key activations |
| `date_range_start`, `date_range_end` | Unix timestamps | Only reviews created in this window. Ignored unless both are set |
| `playtime_min_hours`, `playtime_max_hours` | Hours | Filter by the author's playtime when they wrote the review |
| `filter_offtopic_activity` | `true` (default), `false` | Off-topic "review bomb" reviews are hidden by default. Pass `false` to include them |
| `key` | Publisher key | Optional. Gets you a higher rate limit (see below) |

There's also `display_language` (the language of `review_score_desc`), plus Steam Deck and hardware filters (`primarily_steam_deck`, `hardware_os`, `hardware_gpu`, ...). See the [official docs](https://partner.steamgames.com/doc/webapi/IUserReviewsService) for the full list.

**Response example:**
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

- The score fields (`review_score`, `review_score_desc`, `total_*`) only come back on the **first page**, and only when `review_type` is `0`. Save them from page 1.
- Playtimes are in **minutes**. Timestamps are Unix seconds.
- When you run out of reviews, the response has no `reviews` array at all and `num_reviews` is `0`.

> ⚠️ **Rate limit:** Calls without a key share a lower rate limit (HTTP 429 when you hit it), and responses may be cached for up to 10 minutes. For a higher limit, add `&key={publisherKey}` with a publisher key for your app and call `partner.steam-api.com` instead of `api.steampowered.com`. Do this from your server only.

> 💡 **`input_json`:** Steam's official docs pass the parameters as one URL-encoded JSON object: `?input_json={"appid":3349960,"filter":1,"languages":["all"]}`. Plain query parameters like the ones above work the same way, so use whichever is easier.

**Getting every review** (JavaScript):

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
<summary><b>Python version</b></summary>

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

#### Migrating from /appreviews

If you used the old store endpoint, here's how to switch.

**The quick version:** change the URL and the parameter values, then read the data from `response`.

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

**Everything that changed:**

| Old (`store.steampowered.com/appreviews`) | New (`IUserReviewsService/GetAppReviews`) |
|---|---|
| `/appreviews/{appid}?json=1` | `/IUserReviewsService/GetAppReviews/v1/?appid={appid}` on `api.steampowered.com` |
| `filter=all` / `recent` / `updated` | `filter=0` / `1` / `2` (numbers now, plus `3` for Funny) |
| `review_type=all` / `positive` / `negative` | `review_type=0` / `1` / `2` |
| `purchase_type=steam` / `all` / `non_steam_purchase` | `purchase_type=0` / `1` / `2` |
| `language=all` | `languages[0]=all` (it's a list now) |
| `"success": 1` in the response | Gone. Check the HTTP status instead |
| `query_summary`, `reviews`, `cursor` at the top level | Wrapped in a `response` object |
| `weighted_vote_score` sometimes a string | Always a number |
| `author.personaname`, `profile_url`, `avatar`, `persona_status`, `num_games_owned` | Removed. Use `author.steamid` |
| `reactions`, `app_release_date` | Removed |
| (not available) | New: `developer_response`, `timestamp_dev_responded`, `total_matching`, `day_range_used`, plus date, playtime, Steam Deck and hardware filters |

> Steam's migration notes also list `refunded` as a new field. The old endpoint already returned it in our tests, so if your code reads it, nothing changes.

> ⚠️ **Old parameters fail silently.** If you only swap the URL, `filter=recent` and `language=all` are ignored without an error. You'll get HTTP 200 with English-only reviews sorted by Helpful, usually from just the last 30 days. Convert every parameter.

Steam's own migration notes: [Migrating from /appreviews](https://partner.steamgames.com/doc/webapi/IUserReviewsService#migrating)

### News

```
GET https://api.steampowered.com/ISteamNews/GetNewsForApp/v2/?appid={appid}&count=10
```

| Parameter | Default | Description |
|-----------|---------|-------------|
| `count` | 20 | Number of articles to return |
| `maxlength` | full | Truncate content (0 = full text) |
| `feeds` | all | Comma-separated feed names to filter |

### Achievement Percentages

```
GET https://api.steampowered.com/ISteamUserStats/GetGlobalAchievementPercentagesForApp/v2/?gameid={appid}
```

Returns global unlock percentages for each achievement. Useful for balancing achievement difficulty or showing community stats.

### Game Schema (Stats & Achievements List)

```
GET https://api.steampowered.com/ISteamUserStats/GetSchemaForGame/v2/?appid={appid}&key={webApiKey}
```

Returns all stats and achievements defined for the game, with display names and descriptions. You'll need this to discover stat names before calling `GetGlobalStatsForGame`.

> Note: This is the one public-ish endpoint that benefits from having a Web API key.

---

## Endpoints: Sales & Revenue (Financial/Publisher Key)

All endpoints below use `partner.steam-api.com` and require either a Financial key or a Publisher key with **"Sales Data"** permission enabled.

> 🔒 **Important:** These calls must be made from a server. Never expose your publisher or financial key in client-side code, game builds, or public repositories.

### GetDetailedSales

```
GET https://partner.steam-api.com/IPartnerFinancialsService/GetDetailedSales/v001/
    ?key={key}
    &date={YYYY-MM-DD}
    &highwatermark_id=0
```

Returns **all sales across all your apps for a single date**. There is no per-app endpoint. You filter by `primary_appid` in your code.

**Parameters:**
| Parameter | Required | Description |
|-----------|----------|-------------|
| `key` | Yes | Your financial or publisher key |
| `date` | Yes | `YYYY-MM-DD`, interpreted as **Pacific Time** (not UTC!) |
| `highwatermark_id` | Yes | Start at `0`. If more data exists, response includes `max_id`. Use it to paginate |

**Response example:**
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

Each line item is a sale or refund, broken down by date, country, platform, and package.

> ⚠️ **Gotcha:** Notice that `gross_sales_usd`, `net_sales_usd`, etc. are **strings**, not numbers. You must convert them: `parseFloat(item.gross_sales_usd)` in JavaScript, `float(item["gross_sales_usd"])` in Python.

### GetAppWishlistReporting

```
GET https://partner.steam-api.com/IPartnerFinancialsService/GetAppWishlistReporting/v001/
    ?key={key}
    &appid={appid}
    &date={YYYY-MM-DD}
```

Returns wishlist activity for a specific app on a specific date.

**Parameters:**
| Parameter | Required | Description |
|-----------|----------|-------------|
| `key` | Yes | Your financial or publisher key |
| `appid` | Yes | Your game's App ID |
| `date` | Yes | `YYYY-MM-DD`, in **GMT** (not Pacific Time, yes, it's different from sales) |

> ⚠️ Yesterday is the most recent date with data. Today's data isn't available yet.

**Response example:**
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

`app_min_date` tells you the earliest date data exists for this app. Don't request dates before this. You'll just get empty responses.

### GetChangedDatesForPartner

```
GET https://partner.steam-api.com/IPartnerFinancialsService/GetChangedDatesForPartner/v001/
    ?key={key}
    &highwatermark=0
```

Returns dates that have new or updated financial data. Use this for efficient syncing. Only re-fetch dates that appear in this list instead of polling every day.

```json
{
  "response": {
    "dates": ["2026/04/01", "2026/04/02", "2026/04/03"]
  }
}
```

> ⚠️ **Date format gotcha:** These dates use **slashes** (`YYYY/MM/DD`), but all other endpoints use **dashes** (`YYYY-MM-DD`). You'll need to convert: `date.replace(/\//g, '-')`.

---

## Endpoints: Publisher Key Only

These require a Publisher key associated with the app. They use `partner.steam-api.com`.

### GetGlobalStatsForGame

```
GET https://partner.steam-api.com/ISteamUserStats/GetGlobalStatsForGame/v1/
    ?key={key}
    &appid={appid}
    &count=1
    &name[0]=stat_name
```

Returns aggregated global stats with optional date range filtering.

| Parameter | Description |
|-----------|-------------|
| `name[0]`, `name[1]`, etc. | Stat names to retrieve |
| `count` | Number of stats requested |
| `startdate`, `enddate` | Optional Unix timestamps for daily aggregates |

> **Prerequisite:** Your app must have stats defined in **Steamworks → App Admin → Stats & Achievements**. You need to know stat names. Use `GetSchemaForGame` to discover them.

### GetPartnerAppListForWebAPIKey

```
GET https://partner.steam-api.com/ISteamApps/GetPartnerAppListForWebAPIKey/v2/?key={key}
```

Returns all apps your publisher key has access to. Very useful for verifying your key is set up correctly.

### GetPlayersBanned

```
GET https://partner.steam-api.com/ISteamApps/GetPlayersBanned/v1/?key={key}&appid={appid}
```

Returns a list of banned players for your game.

### GetLeaderboardsForGame

```
GET https://partner.steam-api.com/ISteamLeaderboards/GetLeaderboardsForGame/v2/?key={key}&appid={appid}
```

Returns all leaderboards defined for your game.

> **Prerequisite:** Leaderboards must be created in **Steamworks → App Admin → Leaderboards** first.

### ISteamMicroTxn/GetReport

```
GET https://partner.steam-api.com/ISteamMicroTxn/GetReport/v5/
    ?key={key}
    &appid={appid}
    &type=GAMESALES
    &time={RFC3339}
    &maxresults=1000
```

Returns microtransaction reports. Report types: `GAMESALES`, `STEAMSTORESALES`, `SETTLEMENT`, `CHARGEBACK`, `SUBSCRIPTION`.

> **Prerequisite:** Your app must use Steam microtransactions.

---

## Regional Pricing Lookup

To check your game's price in different regions, call the Store API with a country code:

```bash
# US (default)
curl "https://store.steampowered.com/api/appdetails?appids={appid}&cc=us"

# Eurozone
curl "https://store.steampowered.com/api/appdetails?appids={appid}&cc=de"

# Türkiye
curl "https://store.steampowered.com/api/appdetails?appids={appid}&cc=tr"

# China
curl "https://store.steampowered.com/api/appdetails?appids={appid}&cc=cn"

# Brazil
curl "https://store.steampowered.com/api/appdetails?appids={appid}&cc=br"

# Japan
curl "https://store.steampowered.com/api/appdetails?appids={appid}&cc=jp"
```

The `price_overview` object in the response contains:
- `currency`: e.g., `"USD"`, `"EUR"`, `"TRY"`, `"CNY"`
- `initial`: base price in cents
- `final`: current price after discount, in cents
- `discount_percent`: active discount percentage

## Steam Language Codes

Steam uses its own language codes for API calls and store pages. Some of these are non-standard (like `schinese`, `tchinese`, `brazilian`, `koreana`, `latam`), so you can't just use ISO codes.

| English Name | Native Name | API Language Code | Web API Code |
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

> **Source:** [Steamworks Languages Documentation](https://partner.steamgames.com/doc/store/localization/languages)
>
> **Note:** API language codes are used with Steamworks client-side APIs. Web API language codes are used with the Steamworks Web API. One exception: the [Reviews](#reviews) endpoint (`GetAppReviews`) takes **API language codes** (`turkish`, not `tr`). Pass `tr` and you'll get zero reviews. Additional languages (Afrikaans, Albanian, Hebrew, Hindi, etc.) are available for store page language selection only and are not supported in APIs.

> 💡 **Pro tip:** You can preview any Steam store page in a different language by adding `?l=` to the URL. For example: [`store.steampowered.com/app/3349960/okeygg/?l=turkish`](https://store.steampowered.com/app/3349960/okeygg/?l=turkish&utm_source=steamworks_api_simplified) shows the [okey.gg](https://store.steampowered.com/app/3349960?utm_source=steamworks_api_simplified) store page in Turkish. Use the API language codes from the table above (not the Web API codes).

---

## Packages and Bundles

Steam doesn't sell apps directly. Every purchase goes through a **package** (also called a "sub"). Even if your game has no DLC, no editions, and no bundles, it still has at least one default package that was created automatically.

**Why this matters:** Some Steamworks features (like refund data) use **Package ID** instead of App ID. If you're looking for your refund stats and your App ID doesn't work in the URL, you probably need the Package ID instead.

**How to find your Package ID:**
1. Go to [partner.steamgames.com/apps/associated/{appid}](https://partner.steamgames.com/apps/associated/3349960)
2. Look under **"All Associated Packages"**
3. Your default package is usually named "{Game Name} for Steam" or similar

For example, [okey.gg](https://store.steampowered.com/app/3349960?utm_source=steamworks_api_simplified) has App ID `3349960` and its default package ID is `1185712`.

**Bundles** are different from packages. A bundle groups multiple packages together at a discount (like a "Complete Edition" that includes the base game + all DLC). Bundles have their own Bundle ID and are managed in **Steamworks → Store Page → Bundles**. Bundle pricing is calculated from the combined package prices minus the bundle discount.

> Learn more: [Steamworks Packages Documentation](https://partner.steamgames.com/doc/store/application/packages)

---

## Data NOT Available via API

Some data only exists in the Steamworks Partner Portal web interface. There are no API endpoints for these. Valve simply doesn't expose them.

| Data | Where to find it |
|------|------------------|
| Store page traffic | [partner.steamgames.com/apps/navtrafficstats/{appid}](https://partner.steamgames.com/apps/navtrafficstats/3349960) |
| UTM campaign analytics | [partner.steamgames.com/apps/utmtrafficstats/{appid}](https://partner.steamgames.com/apps/utmtrafficstats/3349960) |
| Refund details & reasons | [partner.steampowered.com/package/refunds/{packageid}/](https://partner.steampowered.com/package/refunds/1185712/) |

These portal pages do offer CSV export if you need the data elsewhere.

> ⚠️ **Refunds use Package ID, not App ID.** Steam organizes purchases by packages (also called "subs"), not by App ID. Every game has at least one package. To find your package ID, go to [partner.steamgames.com/apps/associated/{appid}](https://partner.steamgames.com/apps/associated/3349960) and look under "All Associated Packages". For example, [okey.gg](https://store.steampowered.com/app/3349960?utm_source=steamworks_api_simplified) has App ID `3349960` but its main package ID is `1185712`.
>
> Learn more about packages: [Steamworks Packages Documentation](https://partner.steamgames.com/doc/store/application/packages)

---

## Common Gotchas

Things that will trip you up if nobody warns you. Consider yourself warned.

### Date formats are inconsistent

| Endpoint | Date format | Timezone |
|----------|------------|----------|
| `GetDetailedSales` | `YYYY-MM-DD` (dashes) | Pacific Time |
| `GetAppWishlistReporting` | `YYYY-MM-DD` (dashes) | GMT |
| `GetChangedDatesForPartner` | `YYYY/MM/DD` (slashes) | - |

Yes, sales data uses Pacific Time and wishlist data uses GMT. And changed dates use slashes while everything else uses dashes. Welcome to Steam.

### Financial values are strings, not numbers

Fields like `gross_sales_usd`, `net_sales_usd`, `base_price` come back as `"9.9900"` (a string), not `9.99` (a number). Always convert:

```javascript
// ❌ Wrong: this compares strings
if (item.gross_sales_usd > 0) { ... }

// ✅ Correct
if (parseFloat(item.gross_sales_usd) > 0) { ... }
```

### Empty response ≠ error

When your publisher key is valid but missing the "Sales Data" permission, Steam returns:

```json
{ "response": {} }
```

No error code. No error message. Just... nothing. If you're getting empty responses from `IPartnerFinancialsService`, check your key's permissions in Steamworks.

### Serverless + IP whitelisting don't mix

If you deploy to Vercel, AWS Lambda, Cloudflare Workers, or any serverless platform, your server's IP address changes with every request. If your Steam key has IP whitelisting enabled, every request will fail.

**Fix:** Remove IP restrictions from your key in Steamworks, or use a fixed-IP proxy.

### Not all dates have sales data

Some days your game just doesn't sell. The API returns empty results for those dates. That's normal, not an error. Use `GetChangedDatesForPartner` to find which dates actually have data instead of blindly requesting every date.

### Wishlist data has a minimum date

Each app has an `app_min_date` (returned in the wishlist response). Data before that date doesn't exist. Don't waste requests on dates before it.

### The reviews API hides most reviews by default

`GetAppReviews` only returns **English** reviews from **Steam purchases** unless you say otherwise. If your game has players in other languages or you gave out keys, you'll see far fewer reviews than your store page shows. Always pass `languages[0]=all` and `purchase_type=1`.

It also fails silently on bad input. Old string values like `filter=recent` are ignored (you get the default Helpful sort), Web API language codes like `tr` return zero reviews, and an unencoded cursor with a `+` in it returns `{"response":{}}`.

---

## Error Reference

| What you see | What it means | How to fix |
|---|---|---|
| HTTP 403 + "Access is denied" HTML page | Wrong key type for this host | Use a publisher/financial key for `partner.steam-api.com`. A regular Web API key won't work. |
| HTTP 200 + `{"response":{}}` | Key is valid but missing permission | Enable "Sales Data" permission on the publisher group in Steamworks |
| HTTP 200 + `{"response":{"result":8}}` | No stats/data configured | Set up stats, achievements, or leaderboards in Steamworks first |
| Calls to `store.steampowered.com/appreviews` stopped working | Steam turned the endpoint off on October 22, 2026 | Switch to `GetAppReviews`. See [Migrating from /appreviews](#migrating-from-appreviews) |
| HTTP 200 + far fewer reviews than your store page shows | `GetAppReviews` defaults to English-only, Steam-purchase-only reviews | Add `languages[0]=all&purchase_type=1` |
| HTTP 200 + `{"response":{}}` on page 2+ of reviews | Cursor wasn't URL-encoded | Encode the cursor (`encodeURIComponent` / `URLSearchParams`) |
| HTTP 429 | Rate limited | Slow down. Add caching. |
| Connection timeout on partner API | IP blocked by whitelist | Remove IP restrictions from key, or add your server IP |

---

## Steamworks Setup Checklist

Some endpoints return nothing until you configure the corresponding feature in Steamworks. Here's what to set up and what it unlocks:

| Feature | Where to configure | What it unlocks |
|---------|-------------------|-----------------|
| Stats | Steamworks → App Admin → Stats & Achievements → Stats | `GetGlobalStatsForGame`: aggregated game stats with time ranges |
| Achievements | Steamworks → App Admin → Stats & Achievements → Achievements | `GetGlobalAchievementPercentagesForApp`: unlock percentages |
| Leaderboards | Steamworks → App Admin → Leaderboards | `GetLeaderboardsForGame` + `GetLeaderboardEntries` |
| Microtransactions | Steamworks → App Admin → Microtransaction Configuration | `ISteamMicroTxn/GetReport`: transaction reports |
| Steam Inventory | Steamworks → App Admin → Steam Inventory Service | `IInventoryService` endpoints |

---

## Official References

- [Steamworks Web API Overview](https://partner.steamgames.com/doc/webapi_overview)
- [Web API Authentication & Key Types](https://partner.steamgames.com/doc/webapi_overview/auth)
- [IPartnerFinancialsService](https://partner.steamgames.com/doc/webapi/IPartnerFinancialsService)
- [Wishlist Reporting](https://partner.steamgames.com/doc/marketing/wishlist/reporting)
- [Wishlist Data API Announcement](https://store.steampowered.com/news/group/4145017/view/499474120884358023)
- [IUserReviewsService](https://partner.steamgames.com/doc/webapi/IUserReviewsService)
- [User Reviews API Change Announcement](https://store.steampowered.com/news/group/4145017/view/676258795703241580)
- [Full Interface List](https://partner.steamgames.com/doc/webapi)

---

## AI Agent Skill

This guide is also available as an AI agent skill. If you use [Claude Code](https://claude.ai/code) or similar AI coding tools, you can install the [`steamworks-api-specialist`](steamworks-api-specialist/SKILL.md) skill to give your AI assistant deep knowledge of the Steam Web API — key types, endpoint details, common gotchas, and troubleshooting flows — so it can help you integrate faster.

---

## Contributing

Found an error? Know about an endpoint we missed? Want to add a translation?

We'd love your help. Check out our [Contributing Guide](.github/CONTRIBUTING.md) for details.

**Quick version:**
1. Fork this repo
2. Make your changes
3. Submit a PR

Translations go in `translations/README.{language-code}.md`. See [existing translations](translations/) for reference.

If this guide saved you time, consider giving it a ⭐. It helps other indie devs find it too.

---

## License

[CC0 1.0 Universal (Public Domain)](LICENSE). Do whatever you want with this. Copy it, modify it, include it in your project, sell a course around it. No attribution required (but it's always appreciated 🙏).

---

<p align="center">
  <br>
  Made with ❤️ by <a href="https://yilmaz.games?utm_source=steamworks_api_simplified">yilmaz.games</a>
  <br><br>
  We built this guide while integrating Steam APIs for our game <a href="https://store.steampowered.com/app/3349960?utm_source=steamworks_api_simplified">okey.gg</a>.<br>
  If it helped you out, a <a href="https://store.steampowered.com/app/3349960?utm_source=steamworks_api_simplified">wishlist</a> means the world to a small studio 🙏
  <br><br>
  <sub>Last updated: October 2026</sub>
</p>
