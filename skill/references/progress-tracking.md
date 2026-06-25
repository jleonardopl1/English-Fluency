# Progress Tracking — the living `progress.md` file

Continuity is what turns scattered practice into real fluency. The `progress.md` file is the learner's memory between sessions: who they are, their level, their recurring errors, their review schedule, vocabulary learned, and session history.

## Where it lives (claude.ai / mobile friendly)
Best option: the learner keeps a **Claude Project** called "English Fluency" and stores `progress.md` in the project knowledge — then it's always in context. Otherwise, at session end you output an updated `progress.md` as a copy-paste code block, and the learner pastes it back at the start of next session. Either way:
- **Read it at session start** (or ask for it / offer to start fresh).
- **Update it at session end** and present the new version as a clean code block.
- Update silently mid-session in your working memory; only surface the file at the end (or if interrupted).

## The format
Use the template in `templates/progress.md`. Core sections:

1. **Profile:** name, motivation/goal, target, daily minutes, feedback directness (1–5), start date.
2. **Level:** current CEFR overall + per skill (speaking, grammar, vocab, naturalness, pronunciation), with last-test date.
3. **Streak & stats:** current streak (days/sessions), total sessions, last session date.
4. **Focus patterns (active):** the top recurring errors we're actively fixing.
5. **Review queue (SM-2):** each item with ease, interval, reps, due date (see `spaced-repetition.md`).
6. **Vocabulary learned:** words/phrases/idioms taught (with the date), feeding the review queue.
7. **Mastered:** patterns/vocab retired from active review (the win pile — grows over time).
8. **Session log:** date, what we did, highlights, what to do next.
9. **Coaching notes:** preferences and patterns a good coach remembers ("freezes in role-plays", "loves sports topics", "mornings are better", "wants to sound casual not formal").

## Updating rules
- **Every session:** bump streak/stats, append a session-log entry, update the level if a `/test` happened, refresh the review queue (advance due dates per SM-2), add new vocabulary, promote/demote focus patterns, capture any coaching notes.
- **New recurring error** appears → add to Focus patterns + Review queue.
- **Pattern mastered** (right several reviews running) → move to Mastered, remove from active.
- **Keep it lean.** If the session log or vocab list gets very long, summarize older entries (e.g. collapse months-old sessions into a one-line summary) so the file stays usable.

## Streaks (motivation)
- Count consecutive days (or sessions) with practice. Show it with 🔥. Use "day" for 1, "days" otherwise.
- Don't shame a broken streak — restart warmly: "New streak starts today. 💪"

## Privacy note for the learner
The file is just text the learner controls. Nothing is stored unless they save it. (Mention this once during onboarding so they understand how continuity works.)

## At session end — what to show
1. A short, motivating **session summary** (what we did, a specific win, the next step).
2. The updated **`progress.md`** as a copy-paste code block (unless it's already living in their Project, in which case just note what changed).
3. One clear recommendation for next time. No begging to continue.
