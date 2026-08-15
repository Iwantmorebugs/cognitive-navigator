# Example — the §1 loop, walked end to end

One complete case, so you can see the shape. The work: **an integration that keeps failing intermittently, and we want to add retries.**

It walks `§1 THE CORE LOOP` from top to bottom, leaning on `§2` (GPS), `§3` (past→present→future), `§15` (before/after, because we are changing existing behaviour), `§4` (dumb version + pseudocode) and `§14` (feedback point).

Deliberately **not** a domain-modelling example — no aggregates, no repositories, no framework. Those live in [TRACEABILITY.md](TRACEABILITY.md). This one is the shape of the loop itself, on a problem any stack has.

Two things to watch while you read: (1) no code appears until step 6, and (2) every step says where it came from.

---

```
╔══════════════════════════════════════════════╗
║ PHASE 3 — MAKE THE INTEGRATION SURVIVE       ║
║           A FLAKY PARTNER API                ║
╚══════════════════════════════════════════════╝
```

## 1 · Where we are  → §2 + §3

```
🗺️ PROJECT

Goal: sync appointments with the partner system

✓ 1. Read their API docs
✓ 2. Map their fields to ours
→ 3. Survive their outages          ← 📍 HERE
○ 4. Alerting
○ 5. Backfill for missed windows
```

```
✓ BEFORE
  In step 2 we decided the sync is one-way: we only ever
  read from them and write into our side.

📍 NOW
  We make that read survive their API going down.

WHY
  Because "one-way" is what makes retrying safe at all —
  see the decision in step 8.

→ NEXT
  Alerting, so a permanent outage is not silent.
```

## 2 · The problem  → §1 + §15

```
🔵 BEFORE
   One call. If it fails, the whole sync run fails.

🔴 PROBLEM
   Their API returns 503 a few times a day for a few seconds.
   A 4-second blip kills a run that then waits an hour.

🟢 AFTER
   A short outage should cost seconds, not a whole cycle.

🔧 CHANGE
   Retry the call instead of giving up on the first failure.

➡️ CONSEQUENCE
   Blips stop being incidents. But "retry" is not free —
   step 8 is where that bill comes due.
```

## 3 · What we want, and why  → §1

We want the sync to **distinguish between "they are down for a moment" and "something is actually wrong"**, and only give up on the second.

That is the whole idea. Everything below is how.

## 4 · Dumb version  → §4

```
🧠 IN SIMPLE TERMS

1. Ask the partner for the appointments.
2. Did it work? → done.
3. Did it fail?
4. Is it the kind of failure that might fix itself?
   (they are overloaded, connection dropped)
   → wait a moment and ask again
5. Is it the kind that will never fix itself?
   (our credentials are wrong, the URL is wrong)
   → stop immediately, do not retry
6. Still failing after a few tries?
   → give up and report it
7. Each wait is longer than the last, so we do not
   pile onto a system that is already struggling.
```

Point 5 is the one people skip. Retrying a wrong password is not resilience — it is a slow way to get locked out.

## 5 · Pseudocode  → §4

```
CALL PARTNER API
     ↓
DID IT WORK?
 ├── YES → RETURN THE DATA
 └── NO
      ↓
   IS THIS FAILURE TEMPORARY?
    ├── NO  → STOP NOW, REPORT IT
    └── YES
         ↓
      HAVE WE TRIED TOO MANY TIMES?
       ├── YES → GIVE UP, REPORT IT
       └── NO
            ↓
         WAIT (longer each time)
            ↓
         TRY AGAIN ──┐
                     │
     ┌───────────────┘
     ↓
  (back to the top)
```

That "wait longer each time" is what will be called **exponential backoff** in step 6. It has a name, but the behaviour is exactly the line above — the name adds nothing you have not already read.

## 6 · Architecture and code  → §1 + §9

```
🧠 WHAT WE ACTUALLY DID

Although it touches four files, conceptually it is one thing:

"a failed call gets a few more chances, unless the failure
 is one that more chances cannot fix."

Caller           →   Retry policy      →   HTTP client
─────────            ────────────          ───────────
sync_job.py          retry.py              partner_api.py
                     ↑
                     └── config.py (how many, how long)
```

| File | What it does |
|---|---|
| `sync_job.py` | Asks for the data. Does not know retries exist. |
| `retry.py` | Steps 1-7 of the dumb version, and nothing else. |
| `partner_api.py` | Classifies each failure as temporary or permanent. |
| `config.py` | The two numbers: how many attempts, how long to wait. |

The classification lives in `partner_api.py` on purpose: **only the thing that knows the partner knows which of their errors are worth retrying.** A generic retry helper cannot know that a 429 from this partner is temporary but their 422 never is.

## 7 · Consequences  → §1

**We gain:** a few seconds of partner downtime costs a few seconds, not an hour.

**We accept in exchange:** a slow failure now takes longer to surface. A call that used to fail in 2 seconds can now take 30 before it reports. If anything upstream has a timeout shorter than that, it will now hit it — and that is a new failure mode we did not have this morning.

**It enables:** step 4, alerting. Now that "failed" means "failed persistently" rather than "hiccuped once", an alert on it is worth waking someone for.

## 8 · Feedback point  → §14

```
📌 FEEDBACK POINT

We are retrying reads only, and deliberately not writes.

WHY          Asking for the same data twice is harmless.
             Sending the same booking twice is not — and we
             cannot tell "it failed" apart from "it worked
             but the reply got lost".

TRADE-OFF    Writes stay fragile. A blip during a write still
             fails the run, and that is on purpose.

ALTERNATIVE  Make writes idempotent (a key derived from the
             request, so a repeat returns the first result)
             and then they could be retried too. That is real
             work and a separate decision.

CONSEQUENCE  Today: reads are resilient, writes are honest.
             If we ever want resilient writes, idempotency
             comes first — not more retries.
```

That is enough to answer "go on", "wait, why can't we retry writes?", or "actually do the idempotency now, we need it".

---

## What makes this work

- **Exponential backoff** is named in step 6, not step 3. By the time the term arrives you have already read what it does three times, so it lands as a label rather than a new concept.
- **Step 7 says what we lose**, not just what we gain. The slower failure is a real cost, and without it there is nothing to evaluate.
- **Step 1 repeats a decision from step 2 of the previous phase** even though it was "already said". Without that reminder, "retry" looks arbitrary — it is safe *because* the sync is one-way, and that is the connection worth restating.
- **Step 8 names the alternative we did not take.** A decision with no visible alternative is an announcement, not a decision.
