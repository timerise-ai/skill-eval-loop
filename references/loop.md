# The loop: rounds, the stop rule, the report

## Rounds

| Round | Starts from | Ends with |
|---|---|---|
| Baseline (0) | the latest release's runs, already committed | scores in their result bodies; the fixes for round 1 |
| 1 to n | a release cut from the previous round's fixes | three new runs, scored |

The baseline is scored, never released. Round 1 is the first release this loop
cuts.

## The stop rule

Evaluate after scoring each round, in this order:

1. **Fewer than three rounds done:** continue, whatever the scores. If the
   round has nothing to fix, re-run it by dispatch
   ([releasing.md](releasing.md)); it still counts.
2. **Rounds three to five:** stop when all three agents score full. Otherwise
   fix and continue.
3. **After round five:** stop at the first round in which two of the three
   agents score full. Name the third agent's failed items in the final report
   as open work.

| Scores in a round | Rounds 1 to 2 | Rounds 3 to 5 | Round 6 on |
|---|---|---|---|
| 3 of 3 full | continue (dispatch) | **stop** | **stop** |
| 2 of 3 full | continue | continue | **stop** |
| fewer | continue | continue | continue |

The defaults are the recorded session's: minimum three, unanimity through five,
two of three after. Ask for different bounds before the first round if the
user gives any; never change them mid-loop.

**Stop early** only when a run fails outright (an agent did not work, the
workflow failed, a result is missing) or when the only fix available would
weaken one of the skill's non-negotiables. Report which, and wait for the user.

## Autonomy

Each round pushes to the default branch, creates a tag and publishes a
release, which runs billed agent sessions. Confirm once, before round 1, that
this may happen unattended. After that, report per round and keep going; do
not ask again for each push.

## Per-round report

Short, the same shape every round:

```markdown
**Round 2 (v0.3.6)**

| Agent | Score | Notes |
|---|---|---|
| claude-code | 8/8 | none |
| codex | 8/8 | first clean run: no template edits; model changed since round 1 |
| gemini-cli | 7/8 | `.env.example` left out a variable; the skill said "list both variables" |

**Round 3: v0.3.7 released** (<release URL>)
- <each fix, one line, and the file that carries it>

Watching <run id>.
```

## Final report

- The table of every round's scores, baseline first.
- The stop reason, in the rule's own terms.
- Each release's fixes in one line, with a real defect called out and the
  action it needs outside the skill (apps built from earlier versions).
- How each agent was scored (diff, summary, local rerun), so the reader knows
  what the scores rest on.
- Each agent's model per round, with any change marked at the round it
  happened.
- What changed for the catalog: the current version, a changed test count.

## Bookkeeping

Results from the same agent, prompt and day are suffixed `-2`, `-3` and so on
by the harness, so a day's rounds sort in order. Keep a scratch note that maps
each round to its version, its run id and its result files; it is what the
final report is built from, and the result files alone do not say which round they were.

## Checklist

- [ ] Stop rule bounds stated before round 1
- [ ] Autonomy confirmed once
- [ ] Each round: scored, notes committed, stop rule applied, report sent
- [ ] Final report with scores, stop reason, fixes and follow-ups
