# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

An [Agent Skill](https://agentskills.io) package: markdown only, with no `package.json`. It teaches a skill
maintainer's agent to harden another skill from its automatic agent evals: score each run against a fixed
rubric from the agent's log, fix the skill at the root cause, patch-release, and loop until every agent scores
full. The commands in `references/` (`gh`, `git`, `bun`, `npx vitest`) run against the **target** skill
repository, not this one.

`references/provenance.md` records the one session the loop was extracted from. Its figures (21 runs, six
releases, the per-round scores) are measured; keep them exact and do not add figures that were not measured.

## Structure

- `SKILL.md`: entry point, loaded whole on every activation; stays at or under 150 lines, the closing index
  line aside. Frontmatter `description` is the trigger surface.
- `README.md`: install, activation, file table, six non-negotiables, *Not this*, contributing.
- `references/rubric.md`, `collecting.md`, `fixing.md`, `releasing.md`, `loop.md`, `provenance.md`: one topic
  each, loaded on demand.

## Editing conventions

- **Keep the three tables in sync** with `references/`: the reference directory and the quick start in
  `SKILL.md`, and the file table in `README.md`.
- **The hard rules and the non-negotiables move together**: the six hard rules in `SKILL.md` and the six
  non-negotiables in `README.md` state the same six things.
- **The two scripts run.** `read_logs.py` in `collecting.md` and `extract_blocks.py` in `releasing.md` name
  their file on the first line. After editing either, extract it and run it: `read_logs.py` against a
  downloaded eval run (`gh run download <id> --repo timerise-ai/<skill> -D <dir>`), `extract_blocks.py`
  against a skill whose references carry `// file:` blocks.
- **The stop-rule numbers are one decision.** Three, five and two of three appear in `SKILL.md`, `loop.md`,
  the README and the provenance; change them everywhere or nowhere.
- **Mark additions as additions.** Anything the source session did not exercise belongs under *Added* in
  `provenance.md`.
- Commits follow Conventional Commits with no AI attribution trailers; releases follow the index's
  STANDARD.md.
