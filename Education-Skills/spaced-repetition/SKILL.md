---
name: spaced-repetition
version: 1.0.0
description: >
  Implement SM-2 spaced repetition scheduling for any text-based study
  material. Use this skill whenever the user wants to study using spaced
  repetition, manage a review schedule, track recall quality on flashcards,
  calculate next review dates, or build a personal knowledge retention system
  without Anki. Trigger on phrases like "spaced repetition", "SM-2",
  "review schedule", "study schedule", "when should I review this",
  "space out my studying", or "memory retention". Works with plain text,
  markdown files, JSON, or CSV card lists.
metadata:
  clawdbot:
    requires:
      bins: ["python3"]
---

# Spaced Repetition (SM-2) Skill

Implement the SM-2 algorithm to schedule reviews for any text-based content — no external service required. Stores state in a local JSON file.

---

## The SM-2 algorithm

For each card after a review session, the learner rates recall quality **0–5**:

| Grade | Meaning |
|---|---|
| 5 | Perfect recall, immediate response |
| 4 | Correct recall with slight hesitation |
| 3 | Correct recall with serious difficulty |
| 2 | Incorrect, but correct answer felt familiar |
| 1 | Incorrect, barely remembered |
| 0 | Total blackout |

**Update rules after rating `q`:**

```
if q >= 3:
    if n == 0:
        interval = 1
    elif n == 1:
        interval = 6
    else:
        interval = round(interval * ease_factor)
    n += 1
else:
    n = 0
    interval = 1

ease_factor = max(1.3, ease_factor + 0.1 - (5-q) * (0.08 + (5-q) * 0.02))
next_review = today + interval (days)
```

Starting values: `ease_factor = 2.5`, `n = 0`, `interval = 1`.

---

## Card store format (`cards.json`)

```json
{
  "cards": [
    {
      "id": "card_001",
      "front": "What is the Pythagorean theorem?",
      "back": "a² + b² = c², where c is the hypotenuse.",
      "tags": ["math", "geometry"],
      "n": 0,
      "ease_factor": 2.5,
      "interval": 1,
      "next_review": "2026-03-24",
      "history": []
    }
  ]
}
```

Default path: `~/.spaced-repetition/cards.json`. Create if it doesn't exist.

---

## Core Python implementation

Save as `~/.spaced-repetition/sm2.py` on first use:

```python
#!/usr/bin/env python3
"""SM-2 spaced repetition engine."""
import json, sys, os
from datetime import date, timedelta

STORE = os.path.expanduser("~/.spaced-repetition/cards.json")

def load():
    os.makedirs(os.path.dirname(STORE), exist_ok=True)
    if not os.path.exists(STORE):
        return {"cards": []}
    with open(STORE) as f:
        return json.load(f)

def save(data):
    with open(STORE, "w") as f:
        json.dump(data, f, indent=2, default=str)

def sm2_update(card, q):
    ef = card["ease_factor"]
    n  = card["n"]
    iv = card["interval"]
    if q >= 3:
        if   n == 0: iv = 1
        elif n == 1: iv = 6
        else:        iv = round(iv * ef)
        n += 1
    else:
        n, iv = 0, 1
    ef = max(1.3, ef + 0.1 - (5 - q) * (0.08 + (5 - q) * 0.02))
    next_r = (date.today() + timedelta(days=iv)).isoformat()
    card.update({"n": n, "ease_factor": round(ef, 4),
                 "interval": iv, "next_review": next_r})
    card.setdefault("history", []).append(
        {"date": date.today().isoformat(), "grade": q, "interval": iv})
    return card

def due_cards(data):
    today = date.today().isoformat()
    return [c for c in data["cards"] if c["next_review"] <= today]

def add_card(front, back, tags=None, deck=None):
    data = load()
    import uuid
    card = {"id": str(uuid.uuid4())[:8], "front": front, "back": back,
            "tags": tags or [], "deck": deck or "default",
            "n": 0, "ease_factor": 2.5, "interval": 1,
            "next_review": date.today().isoformat(), "history": []}
    data["cards"].append(card)
    save(data)
    return card

if __name__ == "__main__":
    cmd = sys.argv[1] if len(sys.argv) > 1 else "due"
    data = load()
    if cmd == "due":
        due = due_cards(data)
        print(json.dumps(due, indent=2))
    elif cmd == "stats":
        today = date.today().isoformat()
        due = [c for c in data["cards"] if c["next_review"] <= today]
        upcoming = sorted(
            [c for c in data["cards"] if c["next_review"] > today],
            key=lambda c: c["next_review"])[:5]
        print(json.dumps({"total": len(data["cards"]),
                          "due_today": len(due),
                          "upcoming": upcoming}, indent=2, default=str))
```

---

## Workflows

### Adding cards

When the user provides content (text, notes, a topic):

1. Parse into atomic Q&A pairs (one concept per card).
2. Assign tags from the topic.
3. Call `add_card()` for each pair or bulk-insert into `cards.json`.
4. Confirm: "Added N cards to your [tag] deck. First review scheduled for today."

### Starting a review session

```bash
python3 ~/.spaced-repetition/sm2.py due
```

For each due card:
1. Show the **front** only.
2. Ask the user to try to recall the answer (pause — don't show immediately).
3. Reveal the **back**.
4. Ask the user to rate their recall: `0–5` (or map natural language: "got it" → 5, "sort of" → 3, "no idea" → 0).
5. Call `sm2_update(card, q)` and save.
6. Continue to the next due card.
7. At the end: show session summary — cards reviewed, average grade, next due date.

### Stats overview

```bash
python3 ~/.spaced-repetition/sm2.py stats
```

Display:
```
Total cards: 142
Due today: 8
Next 5 upcoming reviews:
  2026-03-25 — "What is entropy?" (interval: 3d)
  2026-03-26 — "Define photosynthesis" (interval: 6d)
  ...
```

---

## Importing from CSV

If the user has a CSV (`front, back, tags`):

```python
import csv
with open("cards.csv") as f:
    for row in csv.DictReader(f):
        add_card(row["front"], row["back"], row.get("tags","").split("|"))
```

---

## Tips to share with the user

- Keep cards **atomic** — one fact per card. Large cards hurt retention.
- Review every day if possible — even 5 cards beats skipping.
- Grade honestly — overrating yourself defeats the algorithm.
- Mature card = interval > 21 days. That's the goal.
- Add cards for things you *almost* remembered, not just things you failed.
