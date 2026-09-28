# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

An [Agent Skill](https://agentskills.io) package: markdown only. There is no `package.json` here and nothing
in this repository executes. It teaches a skill maintainer's agent to harden another skill from its automatic
agent evals: score each run against a fixed rubric from the agent's log, fix the skill at the root cause,
patch-release, and loop until every agent scores full.

Keep the two straight: the commands in `references/` (`gh`, `git`, `bun`, `npx vitest`) run against the
**target** skill repository, not this one. The two Python scripts are the only code, and they too run
against a target or its downloaded eval logs.

The skill was written by the maintainer who has run the loop on a published skill. `references/provenance.md`
is the rationale layer: the recorded session, its scores per round, what was kept deliberately and what was
added here. Its figures (21 runs, six releases, the per-round scores) are measured; keep them exact and do
not add figures that were not measured.

## Structure

- `SKILL.md`: entry point, loaded whole on every activation, so it stays between 130 and 160 lines, the
  closing index line aside. Frontmatter `description` is the trigger surface; the body carries the loop
  diagram and the seam paragraph that stands in for `adaptation.md`, five critical facts, six hard rules, the
  invocation table, the quick start, the reference directory and the closing index line.
- `README.md`: the human-facing front door, in the section order of the index's STANDARD.md: install,
  activation, file table, six non-negotiables, requirements, *Not this*, contributing, footer.
- `references/rubric.md`, `collecting.md`, `fixing.md`, `releasing.md`, `loop.md`, `provenance.md`: one topic
  each, loaded on demand.
- `evals/`: `prompts.md` holds what a maintainer types after installing, in their words, none naming a
  target skill, since the loop hardens any skill; the first prompt is the agent eval run before every release.
  The skill builds no code, so the checks only confirm the fixture app was left intact and the notes carry
  the score.
  Every other file there is one eval run: measured frontmatter that is never edited, then the notes of the
  person who ran it. Add a prompt rather than rewording one that has results. The procedure is section 10 of
  the index's STANDARD.md.
- `.github/workflows/agent-eval.yml`: the caller of the index's reusable eval workflow, run on every
  published release and on a maintainer's dispatch. It is the same in every skill and was set up by a
  maintainer; do not edit it, and never add a trigger on `push` or `pull_request`.

## Editing conventions

- **Code blocks name their destination on the first line.** `read_logs.py` in `collecting.md` and
  `extract_blocks.py` in `releasing.md` start with `# file: <name>`; the probe in `fixing.md` with
  `// file: probe.ts`.
- **The two scripts run.** After editing either, extract it and run it: `read_logs.py` against a downloaded
  eval run (`gh run download <id> --repo timerise-ai/<skill> -D <dir>`), `extract_blocks.py` against a skill
  whose references carry `// file:` blocks.
- **Identifiers are shared across files.** The rubric item numbers (1 to 8, with 4 to 6 derived from the
  target), the script names, the commit types `fix(skill)`, `chore(evals)` and `chore(release)`, and the
  agent names `claude-code`, `codex`, `gemini-cli` appear in several references. Change them in all or none.
- **Keep the three tables in sync** with `references/`: the reference directory and the quick start in
  `SKILL.md`, and the file table in `README.md`. Links are relative.
- **The hard rules and the non-negotiables move together**: the six hard rules in `SKILL.md` and the six
  non-negotiables in `README.md` state the same six things in the same order, and neither is ever presented
  as optional.
- **The stop-rule numbers are one decision.** Three, five and two of three appear in `SKILL.md`, `loop.md`,
  the README and the provenance; change them everywhere or nowhere.
- **The odd-looking parts stay.** The binary eight-item rubric, the failed item 2 for a correct template edit,
  unanimity through round five: each has an entry under *Kept deliberately* in `provenance.md`. Read it before
  simplifying.
- **Mark additions as additions.** Anything the recorded session did not exercise belongs under *Added* in
  `provenance.md`.
- **Nothing is renamed by the target.** The rubric's items 4 to 6 are written per target from its own
  non-negotiables; every other name here is the skill's contract.
- **Plain punctuation, wrapped at 110 columns.** No em-dashes, en-dashes, arrows, middle dots or smart quotes
  in prose or tables; ranges are written "1 to 4".
- Commits follow Conventional Commits with no AI attribution trailers; releases follow the index's
  STANDARD.md.
