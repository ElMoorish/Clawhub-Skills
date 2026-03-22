---
name: rentcast-agent
version: 1.0.0
description: >
  Pull rental comps, vacancy rates, rent estimates, property records, active
  listings, and market trends using the RentCast REST API. Use this skill
  whenever the user wants to analyze a rental property, estimate rent for an
  address, find comparable rentals, check market vacancy, search active rental
  listings, or research a real estate market by ZIP code or city. Trigger on
  phrases like "rental comps", "rent estimate", "how much can I charge for
  rent", "vacancy rate", "rental market", "comparable rentals", "property
  records", "investment analysis", "RentCast", or "what's the rent in [area]".
  Free tier includes 50 API calls/month.
metadata:
  clawdbot:
    requires:
      bins: ["curl", "python3"]
      env:
        - name: RENTCAST_API_KEY
          description: "RentCast API key from developers.rentcast.io (free tier available)"
---

# RentCast Agent Skill

Access 140M+ property records, rent estimates, rental comps, active listings,
and market statistics via the RentCast REST API.

---

## Setup

1. Sign up free at [rentcast.io](https://app.rentcast.io) → API Dashboard → Create key
2. Free tier: **50 calls/month** — sufficient for analysis tasks
3. `export RENTCAST_API_KEY="your_key_here"`

**Base URL:** `https://api.rentcast.io/v1`
**Auth header:** `X-Api-Key: $RENTCAST_API_KEY`

---

## Core helper

```bash
rc() {
  curl -s "https://api.rentcast.io/v1/$1" \
    -H "X-Api-Key: $RENTCAST_API_KEY" \
    -H "Accept: application/json" \
    "${@:2}"
}
```

---

## 1. Property records

```bash
# Single property lookup by address
rc "properties?address=5500+Grand+Lake+Dr&city=San+Antonio&state=TX&zipCode=78244"

# Bulk search by ZIP code (paginated, max 500/request)
rc "properties?zipCode=78244&propertyType=Single+Family&limit=20&offset=0"

# Filter parameters:
# propertyType: Single Family | Condo | Townhouse | Multi Family | Mobile Home
# bedrooms: 3  or range  bedrooms=2:4
# bathrooms: 2  or range  bathrooms=1:3
# squareFootage: 1000:2500
# yearBuilt: 2000:2015
```

**Response fields:** `id`, `formattedAddress`, `latitude`, `longitude`, `propertyType`,
`bedrooms`, `bathrooms`, `squareFootage`, `lotSize`, `yearBuilt`, `assessedValue`,
`taxAmount`, `owner`, `lastSaleDate`, `lastSalePrice`, `features`

---

## 2. Rent estimate (AVM)

```bash
# Rent estimate by address — returns comps + estimate
rc "avm/rent?address=5500+Grand+Lake+Dr&city=San+Antonio&state=TX&zipCode=78244"

# By coordinates
rc "avm/rent?latitude=29.4241&longitude=-98.4936&propertyType=Single+Family&bedrooms=3&bathrooms=2"
```

**Key response fields:**
```json
{
  "rent": 1850,
  "rentRangeLow": 1700,
  "rentRangeHigh": 2000,
  "subjectProperty": {
    "address": "5500 Grand Lake Dr, San Antonio, TX 78244",
    "bedrooms": 3,
    "bathrooms": 2,
    "squareFootage": 1450
  },
  "comparables": [
    {
      "formattedAddress": "5234 Sunstone Dr, San Antonio, TX",
      "bedrooms": 3,
      "bathrooms": 2,
      "squareFootage": 1380,
      "price": 1795,
      "distance": 0.4,
      "daysOld": 12,
      "listingType": "rental"
    }
  ]
}
```

Optional params: `compCount=15` (default 15 comps), `maxRadius=1.5` (miles)

---

## 3. Home value estimate

```bash
# Property value AVM
rc "avm/value?address=5500+Grand+Lake+Dr&city=San+Antonio&state=TX&zipCode=78244"
```

Returns: `price`, `priceRangeLow`, `priceRangeHigh`, plus sale comps.

---

## 4. Active listings search

```bash
# Active rental listings in an area
rc "listings/rental/long-term?city=San+Antonio&state=TX&propertyType=Single+Family&bedrooms=3&limit=20"

# Active for-sale listings
rc "listings/sale?zipCode=78244&propertyType=Single+Family&maxPrice=300000&limit=20"

# Listing filters:
# minPrice / maxPrice
# minBedrooms / maxBedrooms  (or bedrooms=2:4)
# minBathrooms / maxBathrooms
# minSquareFootage / maxSquareFootage
# daysOld (max days since listed)
```

---

## 5. Market statistics by ZIP code

```bash
# Rental market stats
rc "markets?zipCode=78244&dataType=rental"

# Sale market stats
rc "markets?zipCode=78244&dataType=sale"

# Supported geographies: zipCode, city+state, county+state
```

**Response includes:**
```json
{
  "averageRent": 1820,
  "minRent": 950,
  "maxRent": 3200,
  "medianRent": 1750,
  "averageDaysOnMarket": 24,
  "rentalVacancyRate": 0.047,
  "totalListings": 183,
  "listingCountsByBedrooms": {
    "1": 42,
    "2": 67,
    "3": 58,
    "4": 16
  },
  "averageRentByBedrooms": {
    "1": 1150,
    "2": 1520,
    "3": 1850,
    "4": 2300
  },
  "averageRentPerSqFt": 1.26,
  "historicalAverageRent": [
    { "month": "2026-02", "rent": 1800 },
    { "month": "2026-01", "rent": 1785 }
  ]
}
```

---

## Displaying results

### Rent estimate report

```
Rent Estimate — 5500 Grand Lake Dr, San Antonio, TX 78244
──────────────────────────────────────────────────────────
Property:   3 bed / 2 bath  ·  1,450 sq ft  ·  Single Family

Estimated rent:  $1,850/mo
Range:           $1,700 – $2,000/mo
Rent per sq ft:  $1.28

Top 3 comps (within 0.5 miles):
  1. 5234 Sunstone Dr     $1,795/mo  ·  3/2  ·  1,380 sqft  ·  12 days ago
  2. 5801 Creek Bend      $1,875/mo  ·  3/2  ·  1,500 sqft  ·  5 days ago
  3. 5112 Rock Creek Dr   $1,825/mo  ·  3/2  ·  1,410 sqft  ·  21 days ago

Quick investment check (at $280K purchase):
  Gross yield:   7.9%   (rent × 12 / price)
  1% rule:       0.66%  ← below 1% threshold
```

### Market snapshot

```
78244 ZIP Code — Rental Market Snapshot
────────────────────────────────────────
Avg rent:         $1,820/mo  (+3.2% vs. 6mo ago)
Median rent:      $1,750/mo
Vacancy rate:     4.7%  (healthy — <5% = tight market)
Avg days listed:  24 days
Active listings:  183

By bedroom:
  1 bed:  $1,150/mo  (42 listings)
  2 bed:  $1,520/mo  (67 listings)
  3 bed:  $1,850/mo  (58 listings)
  4 bed:  $2,300/mo  (16 listings)
```

---

## Pagination for bulk queries

When results exceed `limit` (max 500):

```bash
# Get total count first
rc "properties?zipCode=78244&includeTotalCount=true&limit=1" | python3 -c "
import sys, json
r = json.load(sys.stdin)
# Total in X-Total-Count header (pass -I to curl to see headers)
"

# Then paginate
for offset in 0 500 1000; do
  rc "properties?zipCode=78244&limit=500&offset=$offset" >> all_properties.jsonl
done
```

---

## Error handling

| Code | Meaning | Fix |
|------|---------|-----|
| `401` | Invalid API key | Check `RENTCAST_API_KEY` |
| `402` | Plan limit reached | Upgrade plan at rentcast.io |
| `404` | Address not found | Try with just `zipCode` + property type |
| `429` | Rate limit | 10 req/sec max; add `sleep 0.1` between calls |
| `suppressLogging=true` | Add to sensitive queries | Prevents RentCast internal logging |
