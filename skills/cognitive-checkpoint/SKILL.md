---
name: cognitive-checkpoint
description: "Use whenever the user asks where things stand: where are we, checkpoint, catch me up, recap, status, remind me, what did we decide, what were we doing, dónde estamos, ponme al día, recuérdame, qué habíamos decidido, resumen de dónde vamos. Emits one snapshot and stops — the map of done/rejected/here/pending, before-now-next, the open decisions with what would prove each one wrong, and anything waiting on the user. Does no work of its own. Deliberately tiny, so prefer it over re-invoking cognitive-navigator for any request that only wants the current picture: that one is the full working discipline, costs roughly twelve times more, and once loaded in a session it never needs loading again."
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
