# skill-eval-loop

[![Agent Skills](https://img.shields.io/badge/Agent_Skills-open_format-059669)](https://agentskills.io)
[![skills.sh](https://img.shields.io/badge/skills.sh-npx_skills_add-059669)](https://www.skills.sh)
[![Claude Code](https://img.shields.io/badge/Claude_Code-compatible-059669)](https://docs.claude.com/en/docs/claude-code/skills)
[![Codex CLI](https://img.shields.io/badge/Codex_CLI-compatible-059669)](https://developers.openai.com/codex/skills)
[![Gemini CLI](https://img.shields.io/badge/Gemini_CLI-compatible-059669)](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/skills.md)

An [Agent Skill](https://agentskills.io) for skill maintainers. It hardens another skill, one that builds a
module for **Next.js App Router** apps, from its automatic agent evals: score every run against a fixed
fidelity rubric read from the agent's own log, fix the skill at the root cause of each deviation, cut a patch
release, let the release re-run the evals, and repeat until Claude Code, Codex and Gemini CLI all score full.

**An eval that passes its checks has not proved the agent used the skill as written.** The deviations live in
the agent's log, not in the result's checks, and each one traces back to a sentence in the skill that allowed
it. Score the log, fix the sentence, and let the next release's eval prove the fix.

This skill was written by the maintainer who has run the loop on a published skill, from its first scored
release to a unanimous full score. What it carries holds by construction: a rubric fixed before round one and
the same for every agent, so scores compare across rounds; every agent's template edit reproduced against the
template before it is adopted; a template check that type-checks the templates and matches the
documented test count before any round ships; and a stop rule that is set in advance and never moved.
[`references/provenance.md`](references/provenance.md) has the record.

## Install

One command, via the [skills.sh](https://www.skills.sh) CLI, which installs the skill into every
skills-compatible agent it detects, including Claude Code, Codex CLI and Gemini CLI:

```bash
npx skills add timerise-ai/skill-eval-loop
```

Name the agents instead with `-a`, for example `npx skills add timerise-ai/skill-eval-loop -a claude-code -a
codex`.

### Manual install

Nothing here is Claude-specific: the skill is a plain [Agent Skills](https://agentskills.io) folder,
`SKILL.md` plus markdown references with no file that calls a model, so cloning it into an agent's skills
directory is all an install is. For Claude Code:

```bash
git clone https://github.com/timerise-ai/skill-eval-loop.git ~/.claude/skills/skill-eval-loop
```

To scope it to a single project instead, clone it into that project's `.claude/skills/` directory. For another
agent, clone into that agent's skills directory, or symlink the Claude Code copy so one `git pull` updates
every agent:

```bash
mkdir -p ~/.agents/skills
ln -s ~/.claude/skills/skill-eval-loop ~/.agents/skills/skill-eval-loop
```

Update the skill with `git pull` in its directory. The current release is **0.1.0**. See
[CHANGELOG.md](CHANGELOG.md). The [skills index](https://github.com/timerise-ai/skills) lists the other
Timerise Skills and how to install them all at once.

## Activation

The skill activates automatically when a maintainer asks to fix a skill from its evals, iterate it to a full
score, or score its eval runs. Invoke it explicitly with `/skill-eval-loop` in Claude Code, `$skill-eval-loop`
in Codex CLI, or from `/skills` in Gemini CLI, in the working directory of the skill to harden, or with its
path: `/skill-eval-loop ../site-pin-gate`. `/skill-eval-loop score` scores the latest release's runs and
stops, with no fix and no release.

Each host matches a task against the description its own way, so invoke the skill explicitly on a first run
rather than assuming it fired. Only `SKILL.md` is read up front; the `references/` files load on demand.

## What's inside

| File | Contents |
|---|---|
| `SKILL.md` | Entry point: the loop diagram, the seam, critical facts, hard rules, invocation, quick start and reference directory |
| `references/rubric.md` | The eight-item fidelity rubric, how to derive it for a target skill, what is not scored, how scores are recorded |
| `references/collecting.md` | Finding and waiting for a release's eval run, downloading the logs, and `read_logs.py` for all three agents |
| `references/fixing.md` | The root-cause table for deviations, reproducing an agent's template edit, and where each fix goes |
| `references/releasing.md` | The template check with `extract_blocks.py`, the patch release, and the dispatch round |
| `references/loop.md` | Rounds, the stop rule, autonomy, and the per-round and final reports |
| `references/provenance.md` | The engineering ledger: the recorded session and its scores per round, what was kept deliberately, and what was added here and never exercised |
| `README.md` | This file |
| `CHANGELOG.md` | One section per release, newest first |
| `CLAUDE.md` | The editing conventions, for an agent editing this repository |
| `LICENSE` | MIT |

The skill builds no code, so it carries no `evals/` folder of its own. The seam with the target skill is its
own rules and recipes, read and never rewritten: rubric items 4 to 6 come from the target's non-negotiables,
and the template check and commit convention from the target's `CLAUDE.md`. The prompt, the unattended note,
the harness and the workflow belong to the index and stay outside it.

## The six non-negotiables

These are never optional. Each is stated as a hard rule in `SKILL.md`, in the same order:

1. **Result frontmatter is never edited and a run is never deleted.** The frontmatter is what was measured, so
   scores go in the result's body, in a `chore(evals)` commit; `git diff` on the frontmatter stays empty.
2. **The eval is never changed to make a skill pass.** Prompt, unattended note, harness and workflow are the
   same for every skill, so a fix there proves nothing about this one; every fix lands in the skill.
3. **An agent's template edit is reproduced before it is adopted.** An edit is a claim, and a probe against
   the shipped template settles it: a real defect gets a template fix and a test that fails on the old code;
   an improvisation gets a sentence that forbids it.
4. **No round ships without the template check.** The templates type-check and the documented test count
   holds under every runner the skill names, because a round's fix can break the templates it did not touch.
5. **Evals never bump a version.** A version marks a change to the skill; a round with nothing to fix re-runs
   by dispatch, and score commits never ride a release commit.
6. **The stop rule is fixed before round one and followed to the end.** At least three rounds, unanimous
   through round five, two of three after; moving it mid-loop would let a round's scores choose its own
   finish line.

Everything else is the target skill's: its rubric items 4 to 6, its template-check recipe, its commit
convention.

## Requirements

The `gh` CLI signed in with rights to push and publish releases on the target repository, and a target that
already runs its evals on every published release. Every round pushes, tags and publishes, which runs billed
agent sessions, so the skill confirms once before the first round and then runs unattended.

## Not this

| Not this | Use instead |
|---|---|
| Writing a new skill, or turning an app's module into one | `extract-skill` or `skill-creator` |
| Cutting a release without scoring evals | `bumpv` |
| Checking an app's code against its docs | `code-audit` |
| Changing the eval harness, prompts or workflow | The skills index repository, by a maintainer |

## Contributing

Issues and pull requests are welcome here. Pure markdown, with no build step, but the two Python scripts in
the references are checked: `read_logs.py` is run against a downloaded eval run, and `extract_blocks.py`
against a skill whose references carry `// file:` blocks. Claims in this skill are meant to be verifiable: if
you change a factual claim about agent behaviour or a CLI, say how you verified it, whether against the run
whose log showed it, the `gh` or agent CLI's own help, or a reproduction.

Adding, removing or renaming a file in `references/` means updating the quick start and the reference
directory table in `SKILL.md`, the file table above, and any relative cross-links. `references/provenance.md`
is the ledger that must stay truthful: its figures are measured, and anything the recorded session did not
exercise is marked as added; add an entry for anything you change. Commits follow Conventional Commits and
releases follow [STANDARD.md](https://github.com/timerise-ai/skills/blob/main/STANDARD.md) in the index;
`CLAUDE.md` carries the full editing conventions.

## Part of the Timerise Skills

This is one of the [Timerise Skills](https://github.com/timerise-ai/skills): modules for **Next.js App
Router** apps written by our own senior engineers from the modules they have shipped, not synthetic, each
published as its own repository and indexed there. They share one layout, so an agent that has read one knows
how to read the next: a `SKILL.md` entry point, `references/` loaded on demand, and a seam contract carrying
the module's non-negotiables. This one is for their maintainers: it turns the others' eval results into
fixes.

## Author

Built and maintained by [Timerise](https://timerise.ai).

## License

MIT. See [LICENSE](LICENSE).
