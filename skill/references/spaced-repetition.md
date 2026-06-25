# Spaced Repetition (SM-2 lite) — making it stick

Recurring errors and new vocabulary come back on a schedule so they move into long-term memory. You compute this in your head (no scripts needed) and store it in the progress file's review queue. Adapted from the SM-2 algorithm.

## What goes into the review queue
- **Error patterns** the learner repeated (e.g. "3rd-person -s", "depend on", "present perfect + since").
- **Vocabulary / phrases / idioms / phrasal verbs** you taught.
- (Optionally) pronunciation targets they're working on.

Each item is tracked in the progress file with: the item, an **ease** number (start 2.5), an **interval** in days, **reps** (times reviewed correctly in a row), and a **due date**.

## How review works (`/review`)
1. Pull items whose **due date ≤ today** from the progress file.
2. Sort by priority (recurring/meaning-blocking first), cap at ~10–15 per session.
3. For each, quiz **active-recall** style — make them produce, not recognize: "Give me a sentence using *look forward to*." / "Fix this: 'She have two cats.'" / "How do you say 'tenho 30 anos'?"
4. Grade their answer 0–5 (quality):
   - **5** perfect, instant · **4** correct after a beat · **3** correct with effort · **2** wrong but recognized when shown · **1** barely familiar · **0** blank.
5. Update the item's schedule (below) and write it back to the progress file.
6. End with a short report: items reviewed, how many "graduated", what's due next.

## Updating an item after a review (SM-2 lite)

```
if quality >= 3 (got it):
    reps = reps + 1
    if reps == 1: interval = 1 day
    elif reps == 2: interval = 6 days
    else:          interval = round(interval * ease)
else (missed it):
    reps = 0
    interval = 1 day        # see it again tomorrow

# update ease (how "easy" this item is for them)
ease = ease + (0.1 - (5 - quality) * (0.08 + (5 - quality) * 0.02))
if ease < 1.3: ease = 1.3

due_date = today + interval days
```

So: get it right repeatedly → it comes back less often (1 → 6 → ~15 → ~38 days…). Miss it → it resets to tomorrow. Items the learner finds hard (low ease) recur more often automatically.

## Mastery / graduation
- After the learner gets an item right several reviews in a row (reps ≥ 4–5 and interval is large), mark it **mastered** and retire it from the active queue. Celebrate: "You've locked in *present perfect + since* — retiring it from review. 🎉"
- If a "mastered" item resurfaces as an error later, re-add it (reps = 0).

## Keeping it light
- Don't make review feel like a chore. Keep it brisk, gamified, and encouraging.
- Weave a couple of due review items into normal `/chat` or `/drill` naturally when it fits — spaced practice doesn't always need a formal `/review`.
- The point is retention, not bookkeeping. If precise scheduling ever gets in the way of a good session, prioritize the learning.
