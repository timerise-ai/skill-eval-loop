# The rubric

A full score is a run in which every rubric item holds. The items are binary,
fixed before the first round, and the same for every agent and every round, so
a score compares across rounds. Write the rubric once per target skill, from
the target's own rules, and put it in the loop's notes before scoring anything.

## Shape

Eight items. Items 1, 2, 3, 7 and 8 are the same for every skill; items 4, 5 and
6 come from what the target skill says is non-negotiable.

| # | Item | Holds when | Read it from |
|---|---|---|---|
| 1 | Checks | `result: pass`, and every check in `checks` is `pass` | result frontmatter |
| 2 | Templates intact | every file the skill ships as a template matches its code block, apart from renames the skill documents | Codex diff; Claude/Gemini summary, local rerun on doubt |
| 3 | Suite as written | the shipped tests are unmodified, run by a runner the skill names, and report the documented count | log: install commands, test output, test-file diffs |
| 4 | The skill's boundary | the one choice the skill says is the security or correctness boundary is the default or a documented variant | diff or summary |
| 5 | Wiring | the glue the skill documents (the one file that connects the module to the host) follows it; additions the skill calls harmless are allowed | diff or summary |
| 6 | Credentials and config | example files list every variable the skill names, empty, tracked; no secret value in any tracked file; none invented | diff or summary |
| 7 | Hard rules | none of the skill's hard rules broken | all of the above |
| 8 | Handover | the final summary tells the operator what the skill says they must be told (the off state, the variable that must accompany another) | summary |

Score = items held out of eight, per agent. A run that fails item 1 is scored,
reported and committed like any other; the stop rule never counts it as full.

## Writing items 4 to 6 for a target skill

Take them from the places the target skill already marks as not optional:

| Source in the target | Becomes |
|---|---|
| A hard rule about a default (matcher, schema, route surface) | item 4: default or documented variant, nothing in between |
| The wiring block in the reference that connects module to host | item 5: that block, plus only the additions the skill names as harmless |
| The env or config section, and any `.env.example` guidance | item 6: every name listed, empty, tracked, nothing invented |
| The invocation contract (what an argument means, what is written where) | item 6 or 8, whichever it constrains |

If the target has more than eight non-negotiables, fold them into item 7 rather
than adding items; the count stays eight so scores compare across skills.

## What is not scored

State these in the rubric so nobody scores them by accident:

- **Extras the skill allows.** A header the skill calls harmless, an extra test
  file beside the shipped ones, a README section. They cost the agent nothing.
- **What the build rewrites.** `next build` reformatting `tsconfig.json` is not
  the agent's work.
- **Environment limits.** A sandbox with no registry access is the harness's
  problem, not the skill's. Score the item on what the agent did given the
  skill's words: an agent that tried the documented install and fell back as
  the skill says holds item 3.
- **Size.** `linesAdded` and `filesChanged` are clues for where to look, not
  scores.

## Worked example: a proxy-layer PIN gate

The rubric the source session used on a Next.js skill that gates a whole site
from `proxy.ts`:

1. Checks: `result: pass`, typecheck, build and tests `pass`.
2. Templates intact: `lib/site-gate/{config,core,attempts,page,handler}.ts` as
   shipped; only the documented renames (brand default, cookie name, unlock
   path, strings).
3. Suite as written: both test files unmodified, run by vitest or bun, the
   documented count reported. No import rewrite, no shim runner.
4. Matcher: the default or a documented variant; `_next/static` never gated.
5. Wiring: `proxy.ts` calls the gate first, then `NextResponse.next()`; extra
   `noindex` headers allowed.
6. Env: `.env.example` lists `SITE_PIN`, `SITE_GATE_SECRET` and
   `SITE_GATE_BRAND`, empty and un-ignored; no PIN in any tracked file.
7. Hard rules: none of the six broken.
8. Handover: an unset `SITE_PIN` means off, and `SITE_GATE_SECRET` should be
   set wherever `SITE_PIN` is.

## Writing the score down

Append the score to the body of each result file, under the frontmatter, and
never touch the frontmatter. One paragraph: the score, then each failed item
with what the agent did and, when known, the sentence in the skill that let it.

```markdown
Rubric 6/8. The templates are intact and the handover names both variables, but it never tried to install
vitest: it searched the npm cache, found nothing and converted both test files to `node:assert`, reading "no
external services are reachable" as no package registry. The matcher drops two exclusions, which is not a
shape the skill documents.
```

Commit the round's three notes together as `chore(evals): score p1 runs of X.Y.Z`.

## Checklist

- [ ] Eight items written down before the first score
- [ ] Items 4 to 6 traced to a named rule in the target skill
- [ ] "Not scored" list stated
- [ ] Notes in the body only; frontmatter byte-identical
