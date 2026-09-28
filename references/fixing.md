# Turning a deviation into a fix

Every failed rubric item has a cause in the skill: a sentence that allowed it,
a sentence that is missing, a rule stated where it is not read, or a template
that is wrong. Find which, and fix it there. The agent is never the fix.

## Root causes

| Cause | What it looks like | Seen in the recorded session | Fix |
|---|---|---|---|
| **Literal wording** | the agent did exactly what a sentence says, and the sentence is wrong | "list both variables" produced an `.env.example` without the third variable | correct the sentence everywhere it appears, including checklists |
| **Ambiguous environment note** | the agent over-reads the eval's unattended note | "no external services are reachable" read as "no package registry", so vitest was never installed and the suite was converted | say in the skill what the note does not forbid ("the package registry is not an external service") |
| **Rule in the wrong place** | the rule exists in a reference, and the agent never loaded it or weighed it low | an *Indexing* section in a reference did not stop the matcher being widened; a hard rule in `SKILL.md` did | move or repeat the rule in `SKILL.md`: a hard rule, the quick-start step, or the invocation table |
| **Unstated default** | the skill says "choose" where it means "copy" | "choose the matcher" produced a new matcher shape each run | name the default and say it is copied verbatim unless the task names a reason |
| **Real template defect** | the agent edits a template to close a hole that exists | a sanitiser accepted `/a/..//evil.example`, which normalises to `//evil.example` and redirected off-site | reproduce, fix the template, add a test that fails on the old code, record it in the provenance |
| **Improvisation beyond the contract** | the agent wraps the module in logic the skill never asked for | lazy initialisation, a `503` when a secret is missing, a token-format pre-check | one paragraph next to the wiring: what not to add, and why each is unnecessary |
| **Missing contract clause** | the skill is silent on a case, and agents fill it differently | with no PIN given, one agent invented a development PIN and wrote it to `.env.local` | add the clause to every place the contract lives |
| **Environment limit** | the agent did the documented thing and the sandbox refused | `npm install` failing in a sandbox without registry access | not a skill defect; do not score against the agent; note it |

## Reproduce before you adopt

When an agent edits a template, treat the edit as a claim. Write the smallest
probe that feeds the agent's input to the shipped template and prints what
happens:

```ts
// file: probe.ts
import { safeReturnPath } from './lib/site-gate/core';

const ORIGIN = 'https://site.example';
for (const raw of ['/a/..//evil.example', '/.//evil.example', '/a/%2e%2e//evil.example']) {
  const path = safeReturnPath(raw, ORIGIN, '/__unlock');
  console.log(JSON.stringify(raw), '->', path, '->', new URL(path, ORIGIN).href);
}
```

Run it in the scratch project where the templates are written out
([releasing.md](releasing.md)), with `bun probe.ts`.

| Probe result | Verdict | Action |
|---|---|---|
| The template misbehaves on the agent's input | real defect | fix the template the way that keeps every existing test green; add the agent's input as a test case; re-run the probe; score the agent's edit as item 2 failed anyway, since a template edit is a deviation even when it is right |
| The template already handles it | improvisation | answer it with a sentence where the agent would have read it; explain why the extra code is unnecessary |

A real defect found this way is the most valuable result the loop produces.
Label the release that fixes it as a security or fix release in the changelog,
and say which apps built from earlier versions need the fix copied in.

## Where a fix goes

| The fix is | Put it in | Keep in sync |
|---|---|---|
| A rule that must hold on every run | `SKILL.md`: a hard rule, or the quick-start step that does the work | the README's matching non-negotiable |
| A step the agent performs | the quick-start step, pointing at the reference | the reference's own section |
| Detail, reasons, variants | the reference | the reference directory's keywords |
| A template change | the code block, its test block, the behaviour contract if it has one | the documented test count in every file that states it |
| An invocation clause | every place the skill's `CLAUDE.md` says the contract lives | all of them in one commit |
| A new design, not proven before | the provenance's *Added* section, marked as found by the eval | |

`SKILL.md` usually has a line budget. When a fix needs a line, find one to
give back in the same file (merge two short quick-start steps, tighten a rule
in place) rather than going over. A router that grows past its budget stops
being read whole.

## What not to fix

- **Harmless extras.** A header the skill already calls harmless is not worth a
  rule; adding one only moves the next improvisation somewhere else.
- **One agent's taste.** A wording change aimed at one agent must not break
  what the other two already do right; re-read the two passing summaries
  against the new sentence before committing.
- **The eval.** The prompt, the unattended note, the harness and the workflow
  are fixed for every skill. A deviation the note causes is fixed by what the
  skill says about the note.

## Commit

One `fix(skill): ...` commit per round, subject naming the behaviour, body
naming the eval that found each part:

```
fix(skill): install vitest, copy the matcher verbatim, hand over the secret

From the 0.3.4 evals. Codex read "no external services are reachable"
as no package registry and converted the suite to node:assert; the
quick start and testing.md now say to run npm i -D vitest and never to
rewrite the suite.
```

Score notes go in a separate `chore(evals)` commit, before the fix.

## Checklist

- [ ] Every failed item traced to one cause in the table
- [ ] Every template edit probed; real defects fixed with a failing-first test
- [ ] Each fix placed where the agent reads it, and mirrored where the skill requires
- [ ] `SKILL.md` within its line budget
- [ ] The two passing agents' behaviour re-read against the new wording
