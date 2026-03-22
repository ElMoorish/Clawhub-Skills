---
name: showing-scheduler
version: 1.0.0
description: >
  Schedule and manage property showings using Google Calendar or Calendly.
  Generates showing request emails, availability links, confirmation messages,
  and showing feedback forms. Use this skill whenever the user wants to book
  a property showing, schedule open houses, send showing availability to
  prospects, track showing feedback, or manage a showing calendar. Trigger on
  phrases like "schedule a showing", "book a tour", "property viewing",
  "open house", "showing availability", "confirm showing", "showing feedback",
  "set up a tour", or "viewing appointment". Works with Google Calendar API
  and Calendly API.
metadata:
  clawdbot:
    requires:
      bins: ["curl", "python3"]
      env:
        - name: GOOGLE_CALENDAR_TOKEN
          description: "Google Calendar OAuth2 access token"
          optional: true
        - name: CALENDLY_API_KEY
          description: "Calendly personal access token (app.calendly.com/integrations/api_webhooks)"
          optional: true
---

# Showing Scheduler Skill

Schedule and manage property showings via Google Calendar or Calendly,
with automatic confirmations, reminders, and feedback tracking.

---

## Two integration paths

| Method | Best for | Setup |
|--------|---------|-------|
| **Google Calendar** | Agents managing a calendar directly | Google OAuth2 token |
| **Calendly** | Self-booking links for prospects | Calendly API key |
| **No-API mode** | Draft emails/messages without automation | None required |

---

## Path A — Google Calendar

### Setup

```bash
# Google Calendar requires OAuth2. Get a token via:
# 1. Google Cloud Console → APIs → Calendar API → Credentials → OAuth client ID
# 2. Authorize with scope: https://www.googleapis.com/auth/calendar
# 3. Exchange code for token
export GOOGLE_CALENDAR_TOKEN="ya29...."
export CALENDAR_ID="primary"  # or a specific calendar ID
```

### List existing showings

```bash
curl -s "https://www.googleapis.com/calendar/v3/calendars/$CALENDAR_ID/events" \
  -H "Authorization: Bearer $GOOGLE_CALENDAR_TOKEN" \
  -G \
  --data-urlencode "q=showing" \
  --data-urlencode "timeMin=$(date -u +%Y-%m-%dT%H:%M:%SZ)" \
  --data-urlencode "orderBy=startTime" \
  --data-urlencode "singleEvents=true" \
  --data-urlencode "maxResults=20" \
  | python3 -m json.tool
```

### Create a showing appointment

```bash
curl -s -X POST "https://www.googleapis.com/calendar/v3/calendars/$CALENDAR_ID/events" \
  -H "Authorization: Bearer $GOOGLE_CALENDAR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "summary": "Showing — 5500 Grand Lake Dr (John Smith)",
    "location": "5500 Grand Lake Dr, San Antonio, TX 78244",
    "description": "Prospect: John Smith\nPhone: 555-1234\nEmail: john@email.com\nNotes: Interested in 3bed/2bath, moving in May",
    "start": { "dateTime": "2026-03-28T10:00:00-06:00", "timeZone": "America/Chicago" },
    "end":   { "dateTime": "2026-03-28T10:30:00-06:00", "timeZone": "America/Chicago" },
    "attendees": [
      { "email": "john@email.com", "displayName": "John Smith" }
    ],
    "reminders": {
      "useDefault": false,
      "overrides": [
        { "method": "email", "minutes": 1440 },
        { "method": "popup", "minutes": 30 }
      ]
    },
    "colorId": "5"
  }' \
  | python3 -m json.tool
```

**colorId codes:** 1=Lavender, 2=Sage, 3=Grape, 5=Banana, 9=Blueberry, 11=Tomato

### Check availability (free/busy)

```bash
curl -s -X POST "https://www.googleapis.com/calendar/v3/freeBusy" \
  -H "Authorization: Bearer $GOOGLE_CALENDAR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "timeMin": "2026-03-28T00:00:00Z",
    "timeMax": "2026-03-28T23:59:59Z",
    "timeZone": "America/Chicago",
    "items": [{ "id": "primary" }]
  }' \
  | python3 -c "
import json, sys
data = json.load(sys.stdin)
busy = data['calendars']['primary']['busy']
print('Busy blocks:')
for b in busy:
    print(f'  {b[\"start\"][:16]} → {b[\"end\"][:16]}')
"
```

---

## Path B — Calendly

### Setup

```bash
# Get personal access token at: app.calendly.com/integrations/api_webhooks
export CALENDLY_API_KEY="your_token"

# Get your user URI (needed for most calls)
CALENDLY_USER=$(curl -s "https://api.calendly.com/users/me" \
  -H "Authorization: Bearer $CALENDLY_API_KEY" \
  | python3 -c "import json,sys; print(json.load(sys.stdin)['resource']['uri'])")
```

### List scheduled events (booked showings)

```bash
curl -s "https://api.calendly.com/scheduled_events" \
  -H "Authorization: Bearer $CALENDLY_API_KEY" \
  -G \
  --data-urlencode "user=$CALENDLY_USER" \
  --data-urlencode "min_start_time=$(date -u +%Y-%m-%dT%H:%M:%SZ)" \
  --data-urlencode "status=active" \
  --data-urlencode "count=20" \
  | python3 -m json.tool
```

### Get invitee details for a booked showing

```bash
EVENT_UUID="your_event_uuid"
curl -s "https://api.calendly.com/scheduled_events/$EVENT_UUID/invitees" \
  -H "Authorization: Bearer $CALENDLY_API_KEY" \
  | python3 -c "
import json, sys
data = json.load(sys.stdin)
for inv in data['collection']:
    print(f\"Name: {inv['name']}\")
    print(f\"Email: {inv['email']}\")
    qs = inv.get('questions_and_answers', [])
    for q in qs:
        print(f\"{q['question']}: {q['answer']}\")
"
```

### Get your booking link

```bash
curl -s "https://api.calendly.com/event_types" \
  -H "Authorization: Bearer $CALENDLY_API_KEY" \
  -G --data-urlencode "user=$CALENDLY_USER" \
  | python3 -c "
import json, sys
types = json.load(sys.stdin)['collection']
for t in types:
    if t['active']:
        print(f\"{t['name']}: {t['scheduling_url']}\")
"
```

---

## Path C — No-API mode (email/message drafts)

When no calendar integration is set up, generate ready-to-send communications.

### Initial showing availability email

```
Subject: Schedule a Showing — 5500 Grand Lake Dr, San Antonio

Hi [Name],

Thank you for your interest in 5500 Grand Lake Dr! I'd love to show you the property.

I'm available for showings at the following times:
  • [DAY], [DATE] at 10:00 AM, 12:00 PM, or 2:00 PM
  • [DAY], [DATE] at 11:00 AM or 3:00 PM

The showing typically takes 20–30 minutes.

[OPTIONAL: Book directly here: calendly.com/your-link]

To confirm, please reply with your preferred time, full name, and a phone number.
I'll send you a confirmation with directions and parking details.

Looking forward to meeting you!
[Your name]
```

### Showing confirmation (send after booking)

```
Subject: Confirmed: Showing at 5500 Grand Lake Dr — [DATE] at [TIME]

Hi [Name],

Your showing is confirmed! Here are the details:

📍 Address: 5500 Grand Lake Dr, San Antonio, TX 78244
📅 Date: [DAY], [DATE]
⏰ Time: [TIME] (please arrive on time)
⏱️ Duration: ~30 minutes
🚗 Parking: [driveway / street parking on Oak Ave]

What to bring: valid photo ID

If you need to reschedule, please contact me at least 24 hours in advance at
[EMAIL/PHONE].

See you soon!
[Your name]
```

### 24-hour reminder (auto-send)

```
Hi [Name] — just a reminder about your showing at 5500 Grand Lake Dr
tomorrow at [TIME]. Reply to confirm or let me know if anything changes!
```

### Post-showing feedback request

```
Hi [Name], thanks for touring 5500 Grand Lake Dr today!

Quick questions to help me assist you better:
1. Overall impression: ⭐⭐⭐⭐⭐ (reply 1–5)
2. Does it meet your needs?
3. Top concern or hesitation?

If you'd like to apply, I can send the application link.
Either way, happy to help you find the right fit!
```

---

## Showing calendar management

### Daily showing schedule format

```
Thursday, March 28 — 5500 Grand Lake Dr
──────────────────────────────────────────
10:00 AM  John Smith (john@email.com / 555-1234)
          Needs: 3bd, May move-in, budget $1,800
          Source: Zillow inquiry

11:30 AM  Maria Gonzalez (maria.g@gmail.com / 555-5678)
          Needs: 3bd, flexible timeline, pets (1 dog)
          Source: Facebook Marketplace

2:00 PM   [OPEN SLOT]

3:30 PM   Robert Chen (rob.chen@work.com / 555-9012)
          Needs: 3bd, employer relocation, June 1
          Source: Apartments.com
```

### Tracking showing-to-application conversion

Keep a simple log:

| Date | Prospect | Showed | Applied | Approved | Notes |
|------|---------|--------|---------|----------|-------|
| Mar 28 | J. Smith | ✅ | ✅ | ⏳ | Credit check pending |
| Mar 28 | M. Gonzalez | ✅ | ❌ | — | Wanted bigger yard |
| Mar 28 | R. Chen | ✅ | ✅ | ✅ | Move-in June 1 |

---

## Open house workflow

1. Create Google Calendar event (all-day or timed, public)
2. Generate sharing message for social media / email list
3. Set up Calendly event type for 15-min slots within open house window
4. After event: compile attendee list + send follow-up to all

**Open house social post template:**
```
🏠 OPEN HOUSE — [ADDRESS]
📅 [DATE] · [START TIME] – [END TIME]
🛏️ [BEDS] bed / 🛁 [BATHS] bath · [SQFT] sq ft
💰 $[PRICE]/mo (rent) or $[PRICE] (sale)

[2-line property highlight]

No appointment needed — stop by anytime!
[Optional: book a private tour: calendly.com/link]
```
