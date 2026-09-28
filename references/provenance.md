# Provenance

This skill was extracted from one maintainer session on a published Timerise
skill: a Next.js module that puts a shared-PIN gate in front of a whole site.
That skill already had an eval workflow running Claude Code, Codex and Gemini
CLI on prompt 1 of its eval prompts at every published release. The session
took it from 0.3.2 to 0.3.7. The first two releases were fixed ad hoc from
reading logs; the last three ran as the loop this skill describes, with a
fixed rubric and stop rule.

## The record

| Release | Runs | Checks passed | Rubric scores (claude-code / codex / gemini-cli) | What the round fixed |
|---|---|---|---|---|
| 0.3.2 | 6 | 6 | not scored | |
| 0.3.3 | 3 | 3 | not scored | attempt store unbounded under a key flood; `.env.example` ignored by `.env*`; runner guidance; an *Indexing* section |
| 0.3.4 | 3 | 3 | baseline: 8 / 6 / 7 | matcher hard rule in `SKILL.md`; `no-store` on the unlock redirects |
| 0.3.5 | 3 | 3 | round 1: 8 / 6 / 8 | install the runner, copy the matcher verbatim, hand over the secret |
| 0.3.6 | 3 | 3 | round 2: 8 / 8 / 7 | an open redirect through dot segments, found by an agent; wiring kept small |
| 0.3.7 | 3 | 3 | round 3: 8 / 8 / 8 | all three env variables named; no PIN invented when none is given |

Every one of the 21 runs passed typecheck, build and tests. That is the fact
the skill is built on: `result: pass` measured nothing the loop needed.

## What the session established

### 1. Passing checks hide template edits

In the ad hoc releases, Codex patched a template's memory bound, rewrote the
test imports behind a shim runner, widened the matcher and added headers to
the handler, and every run still passed. Only reading the log showed it.

**Shipped:** the rubric's items 2 to 6, scored from the log. See
[rubric.md](rubric.md).

### 2. One agent's deviation was a security defect in the skill

Against 0.3.5, Codex edited the return-path sanitiser to resolve its result a
second time. A probe showed why: `/a/..//evil.example` passed every existing
check and normalised to `//evil.example`, which the unlock redirect resolved to
another host. The skill had carried the hole since its first release, under a
hard rule written to prevent exactly that class of bug.

**Shipped:** "reproduce before you adopt" as a hard rule and a procedure. See
[fixing.md](fixing.md).

### 3. Deviations trace to sentences

Each failed item in the scored rounds traced to a sentence: "list both
variables" (a missing variable), "choose the matcher" (a new matcher shape),
the eval note's "no external services" (a converted suite), silence on the
no-PIN case (an invented PIN). Each fix was a sentence, and the next round
confirmed it.

**Shipped:** the root-cause table. See [fixing.md](fixing.md).

### 4. Placement matters more than wording

An *Indexing* section in a reference did not stop Codex widening the matcher
in 0.3.3; a hard rule in `SKILL.md` in 0.3.4 did.

**Shipped:** critical fact 5 and the "where a fix goes" table.

## Kept deliberately

- **A binary, eight-item rubric.** Partial credit makes rounds incomparable,
  and a fixed count keeps scores comparable across skills.
- **Extras the skill allows are not scored.** Penalising a harmless header
  only moves the next improvisation somewhere else.
- **The template edit that found the defect still failed item 2.** A correct
  edit is still an agent working around the skill; the credit goes to the fix.
- **Unanimity through round five.** Three agents disagreeing is information
  about the skill; the two-of-three fallback exists only to end a loop that
  keeps finding one agent's taste.
- **Score notes in the result body.** The standard gives the body to the person
  who ran the eval, and the frontmatter stays what was measured.

## Added

Designed in this skill and never exercised in the source session:

- **The local rerun with the index harness.** Planned for summaries that hint
  at a deviation; no summary in the session needed one.
- **The two-of-three fallback after round five.** The session stopped at round
  three with a unanimous score.
- **The dispatch round** for a round with nothing to fix before the minimum.
- **The read_logs.py and extract_blocks.py scripts** as files. The session ran
  the same logic inline; the scripts here were written from it and checked
  against that session's logs.
