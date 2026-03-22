---
name: readability-analyzer
version: 1.0.0
description: >
  Analyze text readability using Flesch-Kincaid Grade Level, Flesch Reading
  Ease, Gunning Fog Index, SMOG, Coleman-Liau, and ARI scores. Use this skill
  whenever the user wants to check if text is appropriate for a target audience,
  simplify writing, grade reading level, assess educational material, or compare
  readability across documents. Trigger on phrases like "readability score",
  "reading level", "Flesch-Kincaid", "fog index", "is this too complex",
  "simplify this text", "what grade level is this", "check my writing level",
  or "analyze text complexity". Works on any pasted text, file, or URL.
metadata:
  clawdbot:
    requires:
      bins: ["python3"]
---

# Readability Analyzer Skill

Compute standard readability indices and provide actionable rewriting suggestions to hit a target reading level.

---

## Supported metrics

| Metric | Output | Target range |
|---|---|---|
| Flesch Reading Ease | 0–100 (higher = easier) | 60–70 for general audiences |
| Flesch-Kincaid Grade Level | US grade (e.g. 8.2) | ≤8 for mass-market, ≤12 for professional |
| Gunning Fog Index | Grade equivalent | ≤12 ideal for most readers |
| SMOG Index | Grade level | Best for health/medical text |
| Coleman-Liau Index | Grade level | Char-based, good for technical text |
| Automated Readability Index (ARI) | Grade level | Consistent with Flesch-Kincaid |
| Average grade level | Mean of all grade metrics | Use as overall summary |

---

## Python implementation

Run inline or save to `/tmp/readability.py`:

```python
#!/usr/bin/env python3
import re, math, sys

def tokenize(text):
    sentences = re.split(r'[.!?]+', text)
    sentences = [s.strip() for s in sentences if s.strip()]
    words     = re.findall(r"\b[a-zA-Z']+\b", text)
    return sentences, words

def count_syllables(word):
    word = word.lower().rstrip("es").rstrip("ed")
    vowels = "aeiouy"
    count = 0
    prev_vowel = False
    for ch in word:
        is_v = ch in vowels
        if is_v and not prev_vowel:
            count += 1
        prev_vowel = is_v
    return max(1, count)

def count_complex_words(words):
    """Words with 3+ syllables, excluding proper nouns and -ed/-es endings."""
    return sum(1 for w in words if count_syllables(w) >= 3)

def analyze(text):
    sentences, words = tokenize(text)
    if not sentences or not words:
        return {"error": "Text too short to analyze (need at least 2 sentences)"}

    n_sent = len(sentences)
    n_word = len(words)
    n_char = sum(len(w) for w in words)
    n_syll = sum(count_syllables(w) for w in words)
    n_comp = count_complex_words(words)

    asl = n_word / n_sent          # avg sentence length
    asw = n_syll / n_word          # avg syllables per word

    # Flesch Reading Ease
    fre = 206.835 - 1.015 * asl - 84.6 * asw

    # Flesch-Kincaid Grade Level
    fkgl = 0.39 * asl + 11.8 * asw - 15.59

    # Gunning Fog
    fog = 0.4 * (asl + 100 * n_comp / n_word)

    # SMOG (requires >= 30 sentences; approximate otherwise)
    smog = 3 + math.sqrt(n_comp * (30 / n_sent)) if n_sent >= 3 else None

    # Coleman-Liau
    L = (n_char / n_word) * 100
    S = (n_sent / n_word) * 100
    cli = 0.0588 * L - 0.296 * S - 15.8

    # ARI
    ari = 4.71 * (n_char / n_word) + 0.5 * asl - 21.43

    grades = [fkgl, fog, cli, ari]
    if smog: grades.append(smog)
    avg_grade = sum(grades) / len(grades)

    return {
        "words": n_word,
        "sentences": n_sent,
        "syllables": n_syll,
        "complex_words": n_comp,
        "avg_sentence_length": round(asl, 1),
        "avg_syllables_per_word": round(asw, 2),
        "flesch_reading_ease": round(fre, 1),
        "flesch_kincaid_grade": round(fkgl, 1),
        "gunning_fog": round(fog, 1),
        "smog_index": round(smog, 1) if smog else "N/A (< 30 sentences)",
        "coleman_liau": round(cli, 1),
        "ari": round(ari, 1),
        "avg_grade_level": round(avg_grade, 1)
    }

if __name__ == "__main__":
    text = sys.stdin.read()
    import json
    print(json.dumps(analyze(text), indent=2))
```

Usage:
```bash
echo "Your text here..." | python3 /tmp/readability.py
# or
cat document.txt | python3 /tmp/readability.py
```

---

## Interpreting scores

### Flesch Reading Ease

| Score | Reading level | Audience |
|---|---|---|
| 90–100 | Very easy | 5th grade |
| 70–90 | Easy | 6th grade |
| 60–70 | Standard | 7th–8th grade |
| 50–60 | Fairly difficult | High school |
| 30–50 | Difficult | College |
| 0–30 | Very difficult | Academic/professional |

### Grade level mapping

| Grade | Age | Examples |
|---|---|---|
| ≤6 | 11–12 | Children's books, health instructions |
| 7–8 | 12–14 | Popular magazines, casual web content |
| 9–10 | 14–16 | Newspaper articles, general non-fiction |
| 11–12 | 16–18 | Quality journalism, business writing |
| 13–15 | College | Technical journalism, trade publications |
| 16+ | Graduate | Academic papers, legal docs |

---

## Output format

Always show a clean summary:

```
Readability Analysis
─────────────────────────────
Words: 342    Sentences: 18    Avg sentence length: 19.0 words

Flesch Reading Ease:     48.3  → Fairly difficult
Flesch-Kincaid Grade:    12.4  → 12th grade
Gunning Fog:             14.1  → College level
Coleman-Liau:            11.9  → 12th grade
ARI:                     12.8  → College
SMOG:                    13.2  → College
─────────────────────────────
Overall estimated grade: 12.9  → College level

Top issues:
  • 23 long sentences (18+ words) — aim for ≤15 avg
  • 47 complex words (14%) — high; target ≤10%
  • Longest sentence: 42 words (flag it to user)
```

---

## Suggesting improvements

When the user wants to reduce complexity:

1. Identify the **5 longest sentences** — suggest splitting at conjunctions.
2. Flag **Tier 2/3 vocabulary** (3+ syllable words) — suggest simpler synonyms.
3. Highlight **passive voice** patterns — suggest active alternatives.
4. Target grade: ask the user, or default based on context:
   - Medical/health content: Grade 6–8 (plain language standard)
   - Web marketing: Grade 7–9
   - Academic: Grade 12–14 is acceptable
   - Legal: Grade 8–10 preferred (plain language laws)
5. Rewrite the most problematic paragraph as a demonstration.

---

## Multi-document comparison

If the user provides multiple texts (e.g., "compare my intro and conclusion"):

Run both through the analyzer and display side-by-side:

```
              Intro    Conclusion
Grade level:   14.2       9.8   ← inconsistent
Fog Index:     16.1      11.3
Avg sent len:  24.3      15.7
```

---

## Batch analysis from a file

```bash
# Split a document into paragraphs and score each:
python3 - <<'EOF'
import json, re
text = open("document.txt").read()
paras = [p.strip() for p in text.split("\n\n") if len(p.split()) > 20]
# run analyze() on each and rank by grade level
EOF
```
