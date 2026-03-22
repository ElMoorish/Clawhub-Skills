---
name: zillow-search
version: 1.0.0
description: >
  Search property listings, fetch Zestimates (automated home value estimates),
  and pull housing market trends using the Zillow/Bridge Interactive API and
  supplementary data sources. Use this skill whenever the user asks about
  property values, wants to search homes for sale or rent, requests a Zestimate
  for an address, or wants market data for a ZIP code or city. Trigger on
  phrases like "Zestimate", "Zillow search", "what's this house worth",
  "homes for sale in", "property value", "housing market trends", "sold prices
  near me", or "how much is [address] worth".
metadata:
  clawdbot:
    requires:
      bins: ["curl", "python3"]
      env:
        - name: BRIDGE_API_KEY
          description: "Bridge Interactive API key for Zestimate & public records (bridgedataoutput.com)"
          optional: true
        - name: RAPIDAPI_KEY
          description: "RapidAPI key for Zillow scraper fallback (rapidapi.com)"
          optional: true
---

# Zillow Search Skill

Access Zillow property data — listings, Zestimates, and market trends — via the
Bridge Interactive API (official Zillow data partner) and RapidAPI Zillow scrapers.

---

## API landscape

Zillow deprecated its original public API in 2021. Data is now available through:

| Source | What it provides | Access |
|--------|-----------------|--------|
| **Bridge Interactive** (`bridgedataoutput.com`) | Official Zestimates, public records, market data | Free account; apply at bridgedataoutput.com/register |
| **RapidAPI Zillow scraper** | Listings, photos, property details, Zestimate | RapidAPI key; several providers, ~$0.001–0.01/req |
| **Zillow Research Data** (`zillow.com/research/data`) | Free CSV market stats by ZIP/metro | No key — public downloads |

**Recommended setup:**
1. Register at [bridgedataoutput.com](https://bridgedataoutput.com/register) for the official Zestimate + records API.
2. Get a [RapidAPI key](https://rapidapi.com) for listing search (free tier available).

---

## Bridge Interactive API — Zestimates & Public Records

**Base URL:** `https://api.bridgedataoutput.com/api/v2/`

**Auth:** Bearer token in header: `Authorization: Bearer $BRIDGE_API_KEY`

### Get Zestimate by address

```bash
# First look up the ZPID (Zillow Property ID) for an address
curl -s "https://api.bridgedataoutput.com/api/v2/zestimates?access_token=$BRIDGE_API_KEY&address=1234+Main+St&citystatezip=Seattle+WA" \
  | python3 -m json.tool
```

Response key fields:
```json
{
  "bundle": [{
    "zpid": "12345678",
    "zestimate": 485000,
    "rentzestimate": 2400,
    "valuationRange": { "low": 461750, "high": 508250 },
    "zestimateChangeOneYear": 12500,
    "percentileWithin30Days": 72,
    "lastUpdated": "2026-03-20"
  }]
}
```

### Get property public record

```bash
curl -s "https://api.bridgedataoutput.com/api/v2/pub/parcels?access_token=$BRIDGE_API_KEY&address=1234+Main+St&city=Seattle&state=WA" \
  | python3 -m json.tool
```

Returns: lot size, year built, beds/baths, property type, tax assessment, owner info, sales history.

### Market data by ZIP code

```bash
curl -s "https://api.bridgedataoutput.com/api/v2/zestimatesMarketData?access_token=$BRIDGE_API_KEY&zipcode=98101" \
  | python3 -m json.tool
```

Returns: median list price, median sale price, inventory, days-on-market, price per sq ft — historical monthly series.

---

## RapidAPI Zillow — Listing Search

Sign up at [rapidapi.com](https://rapidapi.com) → search "Zillow" → subscribe to a provider (e.g. `zillow-com1` or `realty-mole-property-api`).

```bash
# Search for-sale listings in a city
curl -s "https://zillow-com1.p.rapidapi.com/propertyExtendedSearch" \
  -H "X-RapidAPI-Key: $RAPIDAPI_KEY" \
  -H "X-RapidAPI-Host: zillow-com1.p.rapidapi.com" \
  -G \
  --data-urlencode "location=Seattle, WA" \
  --data-urlencode "status_type=ForSale" \
  --data-urlencode "home_type=Houses" \
  --data-urlencode "minPrice=400000" \
  --data-urlencode "maxPrice=700000" \
  --data-urlencode "bedsMin=3" \
  | python3 -m json.tool
```

**`status_type` values:** `ForSale` | `ForRent` | `RecentlySold`

**`home_type` values:** `Houses` | `Condos` | `Townhomes` | `MultiFamily` | `Apartments`

### Get property details by ZPID

```bash
curl -s "https://zillow-com1.p.rapidapi.com/property" \
  -H "X-RapidAPI-Key: $RAPIDAPI_KEY" \
  -H "X-RapidAPI-Host: zillow-com1.p.rapidapi.com" \
  -G --data-urlencode "zpid=12345678" \
  | python3 -m json.tool
```

Returns: full listing detail, photos, price history, tax history, school ratings, walk score, Zestimate if available.

---

## Free market data — no API key required

Zillow publishes free CSV research data at `zillow.com/research/data`. Download and parse directly:

```python
import pandas as pd

# Median sale price by ZIP (monthly, national)
url = "https://files.zillowstatic.com/research/public_csvs/medianvaluepersqft_zip/Metro_medvaluepersqft_uc_sfrcondo_sm_sa_month.csv"
df = pd.read_csv(url)

# Filter to a metro
metro = df[df['RegionName'] == 'Seattle, WA']
# Last 12 months
recent = metro.iloc[:, -12:].T
recent.columns = ['value_per_sqft']
print(recent.tail(12).to_string())
```

Available datasets: median sale price, price cuts, days on market, inventory, rental index — all by ZIP, city, metro, or state.

---

## Displaying results

### Property value summary

```
Property: 1234 Main St, Seattle, WA 98101
────────────────────────────────────────────
Zestimate:     $485,000  (range: $462K–$508K)
Rent estimate: $2,400/mo
1-year change: +$12,500  (+2.6%)

Details: 3 bed / 2 bath · 1,850 sq ft · Built 1998
Lot: 5,200 sq ft · Single Family
Last sold: Mar 2019 for $410,000

Zillow listing: https://www.zillow.com/homedetails/ZPID_zpid/
```

### Listing search results

```
Homes for sale in Seattle, WA · $400K–$700K · 3+ bed
────────────────────────────────────────────────────
1. 4521 Oak Ave N               $525,000
   3 bed / 2 bath · 1,720 sq ft · Listed 5 days ago
   Zestimate: $518,000  ▲ Priced above estimate

2. 892 Lakeview Blvd E          $449,000
   3 bed / 1.5 bath · 1,450 sq ft · Listed 12 days ago
   Zestimate: $461,000  ▼ Below estimate (good value signal)

3. 210 N 45th St                $679,500
   4 bed / 3 bath · 2,100 sq ft · Listed 2 days ago
   Zestimate: $695,000  ▼ Below estimate
```

### Market trend summary

```
Seattle, WA (98101) — Market Snapshot
────────────────────────────────────────
Median sale price:    $589,000  (+4.2% YoY)
Median $/sq ft:       $378      (+3.1% YoY)
Inventory:            842 active listings
Avg days on market:   18 days   (Hot market)
Price cut %:          22% of listings reduced
```

---

## Error handling

| Error | Meaning | Fix |
|-------|---------|-----|
| `401 Unauthorized` | Invalid/expired Bridge token | Re-register or check key |
| `No results` | Address not found | Try ZIP-only or nearby city |
| `429 Too Many Requests` | Rate limit hit | Bridge: 2 req/sec; RapidAPI: per-plan |
| RapidAPI `403` | Subscription inactive | Upgrade plan or switch provider |

---

## Important notes on data use

- Zillow Zestimates are **estimates only**, not appraisals. Accuracy varies ±5–10%.
- Bridge API terms **prohibit storing Zestimate data locally** — display dynamically only.
- For investment decisions, supplement with RentCast comps (see `rentcast-agent` skill) and local MLS data.
