# Example — the §1 loop, walked end to end

One complete case, so you can see the shape. The work: **a nightly report job that has grown too slow to finish before the next one starts.**

It walks `§1 THE CORE LOOP` from top to bottom, leaning on `§2` (GPS), `§3` (past→present→future), `§15` (before/after, because we are changing existing behaviour), `§4` (dumb version + pseudocode) and `§14` (feedback point).

Deliberately **not** a domain-modelling example — no aggregates, no repositories, no framework. Those live in [TRACEABILITY.md](TRACEABILITY.md). This one is the shape of the loop itself, on a problem any stack has.

Two things to watch while you read: (1) no code appears until step 6, and (2) every step says where it came from.

---

```
╔══════════════════════════════════════════════╗
║ PHASE 3 — MAKE THE NIGHTLY JOB FIT           ║
║           IN ITS WINDOW                      ║
╚══════════════════════════════════════════════╝
```

## 1 · Where we are  → §2 + §3

```
🗺️ PROJECT

Goal: reporting that keeps up as the data grows

✓ 1. Find out where the time actually goes
✓ 2. Add an index on the timestamp column
✗ -- Scale the machine up      ← rejected in step 8, cost recurs
→ 3. Stop re-reading everything          ← 📍 HERE
○ 4. Alert when a run is late
○ 5. Backfill tooling
```

```
✓ BEFORE
  Step 1 measured it: 94 % of the six hours is spent
  reading rows we already read last night. The index in
  step 2 made each read faster but did not make them fewer.

📍 NOW
  We stop reading them at all.

WHY
  Because the index took us as far as "faster" goes.
  The remaining win is "less", and that is a different change.

→ NEXT
  Alerting — which only becomes meaningful once "late"
  stops being the normal state.
```

## 2 · The problem  → §1 + §15

```
🔵 BEFORE
   Every night, the job reads the entire table and
   recomputes every figure from scratch.

🔴 PROBLEM
   The table grows every day, so the run gets longer
   every day. It now takes six hours in an eight-hour
   window. In roughly five months it overruns, and a
   run starts while the previous one is still going.

🟢 AFTER
   Each run should do an amount of work proportional to
   what changed that day, not to everything that ever
   happened.

🔧 CHANGE
   Remember where the last run stopped, and start there.

➡️ CONSEQUENCE
   Runtime stops tracking total history and starts
   tracking daily volume. But "where we stopped" is a
   new thing that can be wrong — step 7.
```

## 3 · What we want, and why  → §1

We want the job's cost to depend on **how much happened today**, not on how long the company has existed.

That is the whole idea. Everything below is how.

## 4 · Dumb version  → §4

```
🧠 IN SIMPLE TERMS

1. Look up when we last finished successfully.
2. Read only the rows changed since then.
3. Update the figures with what we just read.
4. Write down the new "last finished" mark.
5. If anything failed, do NOT move the mark —
   so tomorrow picks up the same rows again.
6. First ever run, or no mark? Read everything once.
```

Point 5 is the one people skip. If the mark moves before the work is confirmed, a crash silently skips a day's data and **nothing ever notices** — the figures are just quietly wrong from then on.

## 5 · Pseudocode  → §4

```
READ THE LAST-FINISHED MARK
     ↓
IS THERE ONE?
 ├── NO  → READ EVERYTHING (first run)
 └── YES → READ ONLY WHAT CHANGED SINCE THE MARK
      ↓
UPDATE THE FIGURES
      ↓
DID EVERYTHING SUCCEED?
 ├── NO  → LEAVE THE MARK WHERE IT WAS
 │         (tomorrow redoes today's work)
 └── YES → MOVE THE MARK TO NOW
```

The two branches at the bottom are the whole safety property. Everything else is bookkeeping.

## 6 · Architecture and code  → §1 + §9

```
🧠 WHAT WE ACTUALLY DID

Although it touches four files, conceptually it is one thing:

"the job now asks 'what changed?' instead of 'what exists?'"

Schedule      →   Job logic       →   Data access
─────────         ──────────          ───────────
runner.py         report_job.py       queries.py
                       ↑
                       └── checkpoint.py (read/write the mark)
```

| File | What it does |
|---|---|
| `runner.py` | Starts the job on a schedule. Unchanged. |
| `report_job.py` | Steps 1-6 of the dumb version. |
| `queries.py` | Gains a "since this moment" filter. |
| `checkpoint.py` | Stores and reads the mark. New file. |

`checkpoint.py` is separate on purpose: **the mark has to survive a crash of the job itself**, so it cannot live in the job's own memory or in the same transaction as the work.

## 7 · Consequences  → §1

**We gain:** runtime goes from six hours to minutes, and stops growing with history.

**We accept in exchange:** a new way to be wrong. Rows that arrive *late* — recorded with yesterday's timestamp after we already passed that point — are now invisible forever. Before this change that was impossible, because we re-read everything every time. **We have traded a performance problem for a correctness risk**, and that is a real trade, not a free win.

The usual mitigation is to re-read a small overlapping window (say, the last hour before the mark) and accept processing a few rows twice. That only works if processing a row twice is harmless, which has to be checked, not assumed.

**It enables:** step 4, alerting. "The run was late" is only a useful signal once late has stopped being normal.

## 8 · Feedback point  → §14

Same block as `§7` — a feedback point is a decision placed where the response ends.

```
🧠 DECISION

PROBLEM      The run does not fit in its window, and the
             gap widens every day.

OPTIONS      A) checkpoint, process only what changed
             B) bigger machine

CHOICE       A.

WHY          Scaling buys time proportional to money and
             runs out again. The checkpoint changes what
             the cost depends on.

TRADE-OFF    Late-arriving rows can be missed, which the
             old version made impossible. We take a
             correctness risk we can bound over a
             performance ceiling we cannot.

CONSEQUENCE  If "silently missing a row" is unacceptable
             for these figures, this is the wrong change
             and we should scale instead. That is a
             business question, not a technical one.

HOW YOU'D KNOW THIS IS WRONG
             Rows exist whose timestamp is earlier than a
             checkpoint we already passed. One query over
             last week answers it, and it is worth running
             before we trust the first month of figures.
```

That is enough to answer "go on", "wait, how often do rows actually arrive late?", or "no — for these numbers we cannot miss anything, buy the bigger machine".

---

## What makes this work

- **The word "checkpoint" appears in step 6**, not step 3. By then you have read what it does three times, so the term lands as a label rather than a new concept.
- **Step 7 says what we lose.** The change swaps a performance problem for a correctness risk — without that sentence there is nothing to evaluate, only something to approve.
- **Step 1 repeats a finding from the previous phase** even though it was "already said". Without it, "stop re-reading everything" looks arbitrary; with it, it is the obvious next move after the index stopped helping.
- **Step 8 names the alternative that was not taken**, and says which question decides it. A decision with no visible alternative is an announcement, not a decision.
