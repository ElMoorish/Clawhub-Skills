---
name: airbnb-manager
version: 1.0.0
description: >
  Manage Airbnb listings: sync reservations, draft guest messages, update
  pricing and availability, track revenue, and pull market comps via Airbnb's
  official partner API and iCal sync. Also provides STR market analytics via
  Airbtics API. Use this skill whenever the user wants to manage their Airbnb
  hosting operations, check upcoming reservations, message guests, update
  nightly rates, review earnings, analyze competitor pricing, or research
  short-term rental markets. Trigger on phrases like "Airbnb reservations",
  "manage my listing", "message my guest", "nightly rate", "STR market",
  "Airbnb earnings", "occupancy rate", "short-term rental", or "Airbnb host".
metadata:
  clawdbot:
    requires:
      bins: ["curl", "python3"]
      env:
        - name: AIRBNB_CLIENT_ID
          description: "Airbnb API client ID (approved partners only — see setup)"
          optional: true
        - name: AIRBNB_ACCESS_TOKEN
          description: "Airbnb OAuth2 access token"
          optional: true
        - name: AIRBTICS_API_KEY
          description: "Airbtics API key for STR market analytics (airbtics.com)"
          optional: true
---

# Airbnb Manager Skill

Manage Airbnb host operations and access STR market data.

---

## API access landscape

| Method | Access | Use case |
|--------|--------|----------|
| **Official Airbnb API** (`developer.withairbnb.com`) | Approved partners only (PMS/channel managers) | Full listing + reservation management |
| **iCal sync** | All hosts — no API key | Calendar sync with other platforms |
| **Airbtics API** | Public API key, free trial | STR market data, comp analysis, revenue estimates |
| **Hostaway / Lodgify / Guesty** | Via their APIs (if user uses a PMS) | Management via property management system |

### Getting official Airbnb API access

The official Airbnb API is **restricted to approved software partners** (property management systems, channel managers). Individual hosts cannot get direct API access.

To apply: go to [developer.withairbnb.com](https://developer.withairbnb.com) → register a developer account → apply to an API program (Homes API). Approval is based on business type and intended use.

**If the user has API access**, proceed with the Official API section below.
**If not**, use iCal sync + Airbtics for market data — covers 80% of common host needs.

---

## Option A — Official Airbnb API (approved partners)

**Base URL:** `https://api.airbnb.com/v2/`
**Auth:** `Authorization: Bearer $AIRBNB_ACCESS_TOKEN`

### Get listings

```bash
curl -s "https://api.airbnb.com/v2/listings" \
  -H "Authorization: Bearer $AIRBNB_ACCESS_TOKEN" \
  -H "Accept: application/json"
```

### Get reservations

```bash
# Upcoming reservations for a listing
curl -s "https://api.airbnb.com/v2/reservations?listing_id=LISTING_ID&include_pending=true" \
  -H "Authorization: Bearer $AIRBNB_ACCESS_TOKEN"
```

Key fields: `confirmation_code`, `guest_user.first_name`, `checkin_date`, `checkout_date`,
`nights`, `total_paid_amount`, `status`, `guest_count`

### Update nightly price

```bash
curl -s -X PUT "https://api.airbnb.com/v2/calendars/LISTING_ID/YYYY-MM-DD" \
  -H "Authorization: Bearer $AIRBNB_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"daily_price": 149, "availability": "available", "min_nights": 2}'
```

### Send message to guest

```bash
curl -s -X POST "https://api.airbnb.com/v2/threads/THREAD_ID/messages" \
  -H "Authorization: Bearer $AIRBNB_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"message": "Hi! Your check-in code is 1234. Let me know if you need anything!"}'
```

### Update calendar availability (bulk)

```bash
curl -s -X PUT "https://api.airbnb.com/v2/calendars/LISTING_ID" \
  -H "Authorization: Bearer $AIRBNB_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "calendar_operations": [
      {"start_date": "2026-04-01", "end_date": "2026-04-07", "availability": "blocked"},
      {"start_date": "2026-04-08", "end_date": "2026-04-30", "availability": "available"}
    ]
  }'
```

---

## Option B — iCal sync (all hosts, no API key)

Every Airbnb listing has an iCal URL for calendar export/import.

**Get your iCal URL:**
Airbnb → Manage Listings → [Listing] → Availability → Export Calendar → Copy link

```bash
# Download and parse your Airbnb calendar
ICAL_URL="https://www.airbnb.com/calendar/ical/LISTING_ID.ics?s=TOKEN"

curl -s "$ICAL_URL" | python3 - <<'EOF'
import sys
from datetime import datetime

lines = sys.stdin.read().splitlines()
events = []
event = {}
for line in lines:
    if line == "BEGIN:VEVENT":
        event = {}
    elif line == "END:VEVENT":
        if event:
            events.append(event)
    elif line.startswith("DTSTART"):
        event["start"] = line.split(":")[1][:8]
    elif line.startswith("DTEND"):
        event["end"] = line.split(":")[1][:8]
    elif line.startswith("SUMMARY"):
        event["summary"] = line.split(":", 1)[1]

today = datetime.now().strftime("%Y%m%d")
upcoming = [e for e in events if e.get("start", "") >= today]
upcoming.sort(key=lambda e: e["start"])

for e in upcoming[:10]:
    start = datetime.strptime(e["start"], "%Y%m%d").strftime("%b %d")
    end   = datetime.strptime(e["end"],   "%Y%m%d").strftime("%b %d")
    print(f"  {start} → {end}  {e.get('summary', 'Reserved')}")
EOF
```

**Import external calendar into Airbnb** (to block dates from VRBO, Booking.com, etc.):
Airbnb → Availability → Sync calendars → Import → paste external iCal URL

---

## Option C — STR market analytics via Airbtics

[Airbtics](https://airbtics.com) provides market-level STR data via a public API.

```bash
export AIRBTICS_API_KEY="your_key"

# Market statistics for a city
curl -s "https://api.airbtics.com/api/v1/market/summary" \
  -H "Authorization: Bearer $AIRBTICS_API_KEY" \
  -G \
  --data-urlencode "city=Austin" \
  --data-urlencode "state=TX" \
  --data-urlencode "country=US" \
  | python3 -m json.tool

# Comparable listings near an address
curl -s "https://api.airbtics.com/api/v1/comps" \
  -H "Authorization: Bearer $AIRBTICS_API_KEY" \
  -G \
  --data-urlencode "lat=30.2672" \
  --data-urlencode "lng=-97.7431" \
  --data-urlencode "bedrooms=2" \
  --data-urlencode "radius=2" \
  | python3 -m json.tool
```

**Market response fields:** `averageOccupancy`, `averageDailyRate` (ADR), `revPAR`,
`averageMonthlyRevenue`, `activeListings`, `seasonalityIndex`, `topAmenities`

---

## Guest message templates

When asked to draft guest messages, use these as starting points:

**Check-in instructions:**
```
Hi [Name]! Excited to host you. Here are your check-in details:
📍 Address: [address]
🔑 Door code: [code] (active from [time] on [date])
🚗 Parking: [parking instructions]
📶 WiFi: [network] / [password]
Let me know if you have any questions!
```

**Late check-out request response:**
```
Hi [Name]! Happy to help. I can offer a late check-out until [time] for an additional $[X],
or until [time] at no charge if availability allows. I'll confirm by [time] tomorrow!
```

**5-star review request (send after check-out):**
```
Hi [Name]! Hope you had a wonderful stay. If you have a moment, we'd love a review —
it means the world to small hosts. I've already left one for you! 🙏
```

---

## Revenue tracker (no API needed)

Parse iCal + manual inputs to track monthly revenue:

```python
def monthly_summary(reservations, nightly_rate=150):
    from collections import defaultdict
    from datetime import datetime, timedelta

    monthly = defaultdict(lambda: {"nights": 0, "revenue": 0, "bookings": 0})
    for r in reservations:
        if r.get("summary", "").lower() in ["reserved", "airbnb (not available)"]:
            start = datetime.strptime(r["start"], "%Y%m%d")
            end   = datetime.strptime(r["end"],   "%Y%m%d")
            nights = (end - start).days
            month_key = start.strftime("%Y-%m")
            monthly[month_key]["nights"]   += nights
            monthly[month_key]["revenue"]  += nights * nightly_rate
            monthly[month_key]["bookings"] += 1

    for month, data in sorted(monthly.items()):
        days_in_month = 30
        occ = data["nights"] / days_in_month * 100
        print(f"{month}  {data['nights']:2d} nights  "
              f"{occ:.0f}% occ  "
              f"${data['revenue']:,} rev  "
              f"{data['bookings']} bookings")
```

---

## Pricing strategy guidance

When the user asks about pricing:

| Occupancy last 30d | Signal | Action |
|-------------------|--------|--------|
| > 90% | Underpriced | Raise rates 10–20% |
| 70–90% | Healthy | Maintain; adjust weekends up |
| 50–70% | Soft | Lower mid-week rates by 10–15% |
| < 50% | Pricing issue or seasonality | Drop 15–25%; check comps via Airbtics |

- Enable **Smart Pricing** in Airbnb as a baseline; override manually for holidays and local events.
- Set **minimum stay** to 2–3 nights on weekends to reduce turnover cost.
- Check local **event calendars** — rates can go 2–3× during festivals, conferences, sports.
