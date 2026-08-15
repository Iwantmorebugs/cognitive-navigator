# Cognitive Navigator

An Agent Skill that keeps *you* able to steer while the agent works fast.

```bash
npx skills add Iwantmorebugs/cognitive-navigator
```

## The problem it addresses

Coding agents produce decisions faster than a human can absorb them. That gap is usually described as "the agent explains things too technically", which makes it sound like a writing problem. It is not.

What accumulates in the gap is not unread text — it is **decisions that other decisions are already resting on**. Ten minutes behind, and changing your mind costs a rewrite. Thirty minutes behind, and you stop asking.

So the end state is not "confused user". It is a user who understands the summary perfectly and no longer has any practical way to disagree with it.

## The position this skill takes

**The human is the bottleneck, and should stay the bottleneck.**

Not because slow is good — because the human is the station that decides what counts as finished. Work the human cannot yet evaluate is not progress; it is inventory that has not passed inspection, and it may be scrap.

So the skill does **not** slow the agent down. It does not add permission prompts. What it does is keep the distance between *what has been built* and *what you understand* small enough that you can still change direction.

## What it actually changes

| | |
|---|---|
| **A visual map, repeated** | Where we are, what came before, what comes next — in the same shape every time, so you recognise it in two seconds instead of reading for it |
| **Plain language before code** | What the thing *does*, in human steps, before any class or file is named |
| **Zoom out on confusion** | If you say "I don't understand", it moves *back* through the abstraction levels. Never forward into more detail |
| **Decisions show their alternative** | A choice presented without the option it rejected is an announcement, not a decision |
| **Need before pattern** | Never "we added a Repository because we use the Repository pattern". If the need cannot be named, the abstraction is probably unnecessary |

## When not to use it

Skip it for small, mechanical, or self-evident work — a rename, a typo, a one-line fix, a dependency bump. The skill has guards against ceremony (`§12`, `§13`), but the cheapest guard is not invoking it.

It earns its keep on multi-step work where decisions accumulate: architecture, refactors, integrations, debugging sessions that wander.

## How to tell whether it is working

This is a behavioural skill, so it cannot be unit tested. But it is not unfalsifiable either — here are four checks, cheapest first.

**1. The out-of-context test** *(no human needed)*
Take a single **substantive** response from the middle of a session — one that made a decision, changed behaviour, or introduced something. Show it to someone, or to another model, who has not seen the conversation. Ask: *where is this in the project, why is this step happening, and what comes next?*

If they can answer from that one response alone, it is working. This is the skill's own stated goal (`§17`) turned into something you can run in batch, with no memory of the session.

**Scope it, or you will get false failures.** A response that answers "what is the file path?" in four lines is *correct* and will fail this test. The skill deliberately scales structure to conceptual complexity (`§24`), so applying this to every response measures verbosity, not orientation. Judge it on the responses where a decision was made.

**2. The explain-it-back test** *(the ground truth)*
After a working session, close the laptop and describe the system out loud: what problem we had, what we chose, why, what it costs us, what is next.

Whatever you cannot say is what did not land. This is the only measurement that matters; the other three are proxies for it.

**3. The interruption test** *(counter-intuitive)*
Count how often you interrupt, and **when**.

Working looks like: you interrupt *more*, and *earlier* — before three things are built on top of the decision you dislike. Interruptions dropping to zero is not success, it is the failure mode this skill exists to prevent. A user who never objects has usually stopped being able to.

**4. The ceremony test** *(negative check)*
Give it something trivial. If a variable rename produces the full apparatus of maps and phase checkpoints, `§12` and `§13` are not doing their job and the skill has become theatre.

## Status

**v0.1 — low mileage.** The design is deliberate and argued, but it has not been run across many sessions or many people. If you use it, the four checks above are also how you would tell me it does not work.

## Layout

```
skills/cognitive-navigator/
├── SKILL.md          the skill itself — 17 sections
├── EXAMPLE.md        the core loop walked end to end (no framework, no DDD)
└── TRACEABILITY.md   business → code, for work with a real domain layer
```

`EXAMPLE.md` and `TRACEABILITY.md` are read on demand, so an ordinary session only pays for `SKILL.md`.

## Install

```bash
npx skills add Iwantmorebugs/cognitive-navigator
```

Or copy `skills/cognitive-navigator/` into `~/.claude/skills/` (personal) or `.claude/skills/` (per project), then invoke with `/cognitive-navigator`.

## License

MIT
