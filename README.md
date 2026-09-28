# skill-eval-loop

[![Agent Skills](https://img.shields.io/badge/Agent_Skills-open_format-059669)](https://agentskills.io)
[![skills.sh](https://img.shields.io/badge/skills.sh-npx_skills_add-059669)](https://www.skills.sh)
[![Claude Code](https://img.shields.io/badge/Claude_Code-compatible-059669)](https://docs.claude.com/en/docs/claude-code/skills)
[![Codex CLI](https://img.shields.io/badge/Codex_CLI-compatible-059669)](https://developers.openai.com/codex/skills)
[![Gemini CLI](https://img.shields.io/badge/Gemini_CLI-compatible-059669)](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/skills.md)

An [Agent Skill](https://agentskills.io) for skill maintainers. It hardens another skill from its automatic
agent evals: score every run against a fixed fidelity rubric read from the agent's own log, fix the skill at
the root cause of each deviation, cut a patch release, let the release re-run the evals, and repeat until
Claude Code, Codex and Gemini CLI all score full.

**An eval that passes its checks has not proved the agent used the skill as written.** In the session this
skill was extracted from, all 21 runs across six releases passed typecheck, build and tests, while agents
patched the skill's templates, converted its test suite to another runner, widened its security boundary and
invented a credential. Every one of those traced to a sentence in the skill. One agent's template edit turned
out to be an open redirect the skill had carried since its first release. Three scored rounds took the skill
from 8, 6 and 7 out of 8 to a unanimous 8 out of 8; [`references/provenance.md`](references/provenance.md) has
the record.

## Install

```bash
npx skills add timerise-ai/skill-eval-loop
```

Name the agents instead with `-a`, for example `npx skills add timerise-ai/skill-eval-loop -a claude-code`.

### Manual install

The skill is a plain [Agent Skills](https://agentskills.io) folder, `SKILL.md` plus markdown references, so
cloning it into an agent's skills directory is all an install is. For Claude Code:

```bash
git clone https://github.com/timerise-ai/skill-eval-loop.git ~/.claude/skills/skill-eval-loop
```

Update the skill with `git pull` in its directory. The current release is **0.1.0**. See
[`CHANGELOG.md`](CHANGELOG.md). The [skills index](https://github.com/timerise-ai/skills) lists the other
Timerise Skills.

## Activation

The skill activates when a maintainer asks to fix a skill from its evals, iterate it to a full score, or score
its eval runs. Invoke it explicitly with `/skill-eval-loop` in Claude Code, `$skill-eval-loop` in Codex CLI,
or from `/skills` in Gemini CLI, in the working directory of the skill to harden, or with its path:
`/skill-eval-loop ../site-pin-gate`. `/skill-eval-loop score` scores the latest release's runs and stops,
with no fix and no release.

It needs the `gh` CLI signed in with rights to push and publish releases on the target repository, and a
target that already runs its evals on every published release. Every round pushes, tags and publishes, which
runs billed agent sessions, so the skill confirms once before the first round and then runs unattended.

## What's inside

| File | Contents |
|---|---|
| `SKILL.md` | Entry point: the loop diagram, critical facts, hard rules, invocation, quick start and reference directory |
| `references/rubric.md` | The eight-item fidelity rubric, how to derive it for a target skill, what is not scored, how scores are recorded |
| `references/collecting.md` | Finding and waiting for a release's eval run, downloading the logs, and `read_logs.py` for all three agents |
| `references/fixing.md` | The root-cause table for deviations, reproducing an agent's template edit, and where each fix goes |
| `references/releasing.md` | The template check with `extract_blocks.py`, the patch release, and the dispatch round |
| `references/loop.md` | Rounds, the stop rule, autonomy, and the per-round and final reports |
| `references/provenance.md` | The session this was extracted from, its scores per round, what was kept deliberately and what is new |

## The six non-negotiables

1. **Result frontmatter is never edited and a run is never deleted.** Scores go in the result's body, in a
   `chore(evals)` commit.
2. **The eval is never changed to make a skill pass.** Prompt, unattended note, harness and workflow are
   fixed; every fix lands in the skill.
3. **An agent's template edit is reproduced before it is adopted.** A real defect gets a template fix and a
   test that fails on the old code; an improvisation gets a sentence that forbids it.
4. **No round ships without the template check.** Extracted code type-checks and the documented test count
   holds under every runner the skill names.
5. **Evals never bump a version.** A round with nothing to fix re-runs by dispatch.
6. **The stop rule is fixed before round one.** At least three rounds, unanimous through round five, two of
   three after.

## Not this

| Not this | Use instead |
|---|---|
| Writing a new skill, or extracting one from an app | `extract-skill` or `skill-creator` |
| Cutting a release without scoring evals | `bumpv` |
| Changing the eval harness, prompts or workflow | The skills index repository, by a maintainer |

## Contributing

Issues and pull requests are welcome. Pure markdown, but the two Python scripts in the references are meant to
run: `read_logs.py` against a downloaded eval run, and `extract_blocks.py` against a skill's references. Claims
about agent behaviour should name the run that showed it. Commits follow Conventional Commits and releases
follow [STANDARD.md](https://github.com/timerise-ai/skills/blob/main/STANDARD.md) in the index; `CLAUDE.md`
carries the editing conventions.

## Part of the Timerise Skills

This is one of the [Timerise Skills](https://github.com/timerise-ai/skills), and the one that keeps the others
honest: it is how their eval results turn into fixes.

## Author

Built and maintained by [Timerise](https://timerise.ai).

## License

MIT. See [LICENSE](LICENSE).
