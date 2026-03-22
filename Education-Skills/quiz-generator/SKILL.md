---
name: quiz-generator
version: 1.0.0
description: >
  Turn any document, URL, pasted text, or topic into a scored quiz with
  multiple-choice, true/false, short answer, and fill-in-the-blank questions.
  Use this skill whenever the user wants to test knowledge on a topic,
  generate a quiz from reading material, study for an exam, quiz their
  students, or self-assess understanding. Trigger on phrases like "make a
  quiz", "quiz me on", "test my knowledge", "generate questions from",
  "create a test", "study questions", "exam prep", or "assessment".
  Supports export to Markdown, JSON (Moodle/Canvas compatible), or Anki CSV.
---

# Quiz Generator Skill

Generate scored, multi-format quizzes from any source material with answer keys, explanations, and difficulty tagging.

---

## Input sources

Accept any of:
- **Pasted text** — directly in the conversation
- **File** — PDF, DOCX, TXT, MD (read via file tools)
- **URL** — fetch and extract main content
- **Topic** — generate from Claude's knowledge with specified scope

---

## Question types

| Type | When to use |
|---|---|
| Multiple choice (4 options) | Factual recall, definitions, distinctions |
| True/False | Conceptual checks, common misconceptions |
| Short answer | Application, explain-in-own-words |
| Fill-in-the-blank | Key terms, formulas, dates |
| Ordering | Processes, timelines, sequences |

Default mix for a 10-question quiz: 6 MC + 2 T/F + 2 short answer.

---

## Generation workflow

1. **Receive source** — text, URL, file, or topic.
2. **Determine scope** — ask if not specified:
   - How many questions? (default: 10)
   - Difficulty: easy / medium / hard / mixed? (default: mixed)
   - Question types preferred?
   - Subject tags for the quiz?
3. **Extract key facts** — identify: definitions, relationships, processes, dates, comparisons, cause/effect.
4. **Draft questions** following quality rules below.
5. **Run the quiz interactively** OR export to a file — ask the user which they prefer.

---

## Question quality rules

- **One defensible correct answer** — no trick questions, no ambiguity.
- **Distractors must be plausible** — wrong answers should represent common misconceptions, not obvious nonsense.
- **Bloom's taxonomy spread** — include recall (remember), comprehension (understand), and application (apply) levels.
- **No question should be answerable from its own text** — avoid "According to the passage, X is..." phrasing.
- **Avoid negative phrasing** ("Which is NOT...") unless testing for misconceptions specifically.
- Each question should have a **brief explanation** of why the correct answer is correct.

---

## Difficulty tagging

| Level | Criteria |
|---|---|
| Easy | Direct recall, single fact, present verbatim in source |
| Medium | Inference, comparison, requires understanding relationships |
| Hard | Application, synthesis, requires combining multiple concepts |

---

## Interactive quiz mode

Present one question at a time:

```
Question 3 of 10 [Medium] 🟡

What mechanism allows mRNA to be translated into a protein?

  A) DNA replication
  B) Ribosomes reading codons and assembling amino acids ← correct
  C) The nucleus transcribing DNA
  D) ATP synthesis

Your answer: _
```

After answer:
- ✅ Correct — show brief explanation.
- ❌ Incorrect — show correct answer + explanation.
- Track score silently; reveal at the end.

**End of quiz summary:**

```
Quiz complete!  Score: 7 / 10 (70%) — Good

Missed:  Q2, Q6, Q9
Topic weakness: Cell respiration (missed 2/3 questions)

Review these concepts:
  • Krebs cycle intermediates
  • Role of NADH in oxidative phosphorylation
```

---

## Export formats

### Markdown (default, human-readable)

```markdown
## Quiz: Cell Biology — 10 Questions

**Q1. [Easy] What organelle produces ATP?**
- A) Nucleus
- B) Ribosome
- C) Mitochondria ✓
- D) Vacuole

*Explanation: Mitochondria carry out oxidative phosphorylation to produce ATP.*
```

### JSON (LMS-compatible — Canvas/Moodle QTI-lite)

```json
{
  "title": "Cell Biology Quiz",
  "questions": [
    {
      "id": 1,
      "type": "multiple_choice",
      "difficulty": "easy",
      "text": "What organelle produces ATP?",
      "options": ["Nucleus", "Ribosome", "Mitochondria", "Vacuole"],
      "correct": 2,
      "explanation": "Mitochondria carry out oxidative phosphorylation..."
    }
  ]
}
```

### Anki CSV (import directly into Anki)

```
Front,Back,Tags
"What organelle produces ATP?","Mitochondria — carries out oxidative phosphorylation.","biology cell-bio easy"
```

---

## Special modes

### Exam prep mode

User says "Help me study for [exam]" → generate 20–30 questions weighted toward past exam patterns, then offer 3 timed mock sessions.

### Misconception buster

User says "Quiz me on common mistakes in [topic]" → craft distractors that target the most common misunderstandings; explain *why* each wrong answer is wrong.

### Adaptive follow-up

After a quiz, offer: "You struggled with [subtopic] — want a 5-question deep dive on just that?"

---

## Conversation flow example

```
User: "Quiz me on the French Revolution"
→ Ask: How many questions? Difficulty?
→ Generate and run interactively.
→ At end: show score + recommend review areas.
→ Offer: "Export this quiz?" or "Try again?"
```
