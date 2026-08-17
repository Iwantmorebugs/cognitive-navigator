---
name: cognitive-checkpoint
description: "Take one snapshot of where the work stands right now and stop — the map, what came before, what is happening, what comes next, and which decisions are still open. The cheap mid-session companion to cognitive-navigator: deliberately small so it can be called repeatedly without re-paying for the full skill. Trigger phrases — checkpoint, where are we, catch me up, I lost the thread, what did we decide, recap, status, dónde estamos, ponme al día, resumen de dónde vamos, qué habíamos decidido."
---

# Cognitive Checkpoint

Emit **one snapshot and stop.** No new work, no implementation, no next step taken.

This is the mid-session companion to `cognitive-navigator`. That skill governs how
a whole session is run; this one only takes the photograph. It is short on purpose
— calling it five times in a session should cost almost nothing.

## What to emit

Translate every label into the user's language.

```text
🗺️ WHERE WE ARE

Objective: <one line — what all of this is for>

✓ 1. <done>
✓ 2. <done>
✗ -- <rejected>              ← and why, in four words
→ 3. <current>               ← 📍 HERE
○ 4. <pending>
○ 5. <pending>
```

```text
✓ BEFORE   <the decision or result the current step rests on>
📍 NOW     <what is happening>
→ NEXT     <what follows, and why it follows>
```

```text
🧠 OPEN DECISIONS

<what was decided>
   why: <reason>
   how you'd know it's wrong: <something checkable>

<what is still undecided>
   waiting on: <you / a measurement / an answer>
```

```text
⚠️ NEEDS YOU

<anything blocked on the user's judgement, one line each>
```

Drop any block that is genuinely empty. Do not pad it to look complete.

## Rules

**Read the session, do not re-derive.** The snapshot comes from what has actually
been said and done in this conversation. Do not go re-read the repository to
reconstruct it — that is a different, slower job and it will drift from what the
user remembers.

**Mark gaps as gaps.** If you cannot reconstruct why something was decided, write
`unknown — not recorded` instead of inferring a plausible reason. A confident
wrong map is worse than an honest hole: the user stops checking precisely because
it reads as authoritative.

**Every decision gets its falsifier.** The `how you'd know it's wrong` line is not
optional garnish. An explanation the user cannot check produces confidence without
judgement, which is the specific failure this whole approach exists to avoid.

**No work.** Not even an obvious one-line fix. If something urgent surfaces while
assembling the snapshot, put it under `NEEDS YOU` and stop. The user asked where
they are, not for progress.

**Then stop.** The response ends after the snapshot.
