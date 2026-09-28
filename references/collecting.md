# Collecting and reading the runs

A published release starts the eval workflow. It runs the three agents one
after another on prompt 1, commits one result file per agent from a bot, and
uploads each agent's full log as an artifact. Collecting a round is: find the
run, wait for it, pull the bot's commits, download the logs, read each one.

## What each agent leaves behind

| Agent | Result file | Log artifact | What the log holds |
|---|---|---|---|
| claude-code | `evals/<date>-claude-code-p1[-k].md` | `agent-log-claude-code-p1` | `exit N`, then one JSON object; the summary is `.result`, turns are `.num_turns`, the models are the keys of `.modelUsage` |
| codex | `evals/<date>-codex-p1[-k].md` | `agent-log-codex-p1` | the plain-text final message, then `--- stderr ---` with a header naming `model:` and `reasoning effort:`, and the whole transcript: every command, its output, and the working diff after each turn |
| gemini-cli | `evals/<date>-gemini-cli-p1[-k].md` | `agent-log-gemini-cli-p1` | `exit N`, then one pretty-printed JSON object; the summary is `.response`, tool calls are under `.stats.tools`, the models under `.stats.models` |

Only the Codex log shows code. For Claude Code and Gemini the summary is what
you score, and a local rerun is what you do when the summary is not enough.

## Find the run and wait for it

```bash
REPO=timerise-ai/<skill>
gh run list --repo "$REPO" --limit 1          # the release's run, event "release"
RUN=<id>
# In the background: the harness wakes you when it exits. Recent rounds took 10-15 minutes.
gh run watch "$RUN" --repo "$REPO" --exit-status --interval 30 >/dev/null 2>&1
gh run view "$RUN" --repo "$REPO" --json conclusion,jobs \
  -q '.conclusion, (.jobs[]|"\(.name): \(.conclusion)")'
```

A `manual: skipped` job is normal on a release run. A failed job means an agent
did not work (an API or authentication error); the harness writes no result
file for it. Stop the loop and report; do not score a round with a missing
agent as if it had two.

## Pull the results

```bash
git pull --ff-only
for c in $(git log --format=%h -3); do
  f=$(git show "$c" --name-only --format= | head -1)
  echo "== $f"
  grep -E '^(agent|model|reasoningEffort|skillVersion|durationMinutes|filesChanged|linesAdded|result|  (typecheck|build|tests)):' "$f" | tr '\n' ' '
  echo
done
```

Check `skillVersion` equals the release you cut. A mismatch means the harness
installed an older tag; the round does not count.

Note each agent's `model`, and `reasoningEffort` where it is recorded, next to
its score. The index pins one model per agent and moves to a new one by a
decision in its changelog, not by anything the skill did. A change mid-loop does
not stop it, but a score that moves across the change may be the model's rather
than the fix's, so the round report and the final report name it.

## Download and read the logs

```bash
LOGS=<scratchpad>/logs/<version>
rm -rf "$LOGS" && gh run download "$RUN" --repo "$REPO" -D "$LOGS"
python3 read_logs.py "$LOGS"
```

```python
# file: read_logs.py
"""Print each agent's final summary and, for Codex, the files its final diff touches.

Usage: python3 read_logs.py <dir holding agent-log-*/agent.log>
       python3 read_logs.py <dir> <path>   also print Codex's final diff of <path>
"""
import json
import pathlib
import sys

root = pathlib.Path(sys.argv[1])
show = sys.argv[2] if len(sys.argv) > 2 else None


def split(text: str) -> tuple[str, str]:
    stdout, _, stderr = text.partition("\n--- stderr ---")
    return stdout, stderr


def first_json(stdout: str) -> dict:
    # Claude Code prints one line of JSON, Gemini a pretty-printed object; both follow "exit N".
    start = stdout.index("{")
    obj, _ = json.JSONDecoder().raw_decode(stdout[start:])
    return obj


def codex_final(stderr: str) -> list[str]:
    # Codex repeats the working diff after every turn; the last block after a bare "codex" line is final.
    lines = stderr.splitlines()
    marks = [i for i, line in enumerate(lines) if line == "codex"]
    return lines[marks[-1]:] if marks else lines


def diff_of(block: list[str], path: str) -> list[str]:
    out, inside = [], False
    for line in block:
        if line.startswith("diff --git "):
            if inside:
                break
            inside = line.startswith(f"diff --git a/{path} ")
        if inside:
            out.append(line)
    return out


for log in sorted(root.glob("agent-log-*/agent.log")):
    agent = log.parent.name.removeprefix("agent-log-").rsplit("-p", 1)[0]
    stdout, stderr = split(log.read_text())
    print(f"\n===== {agent}")
    if agent == "codex":
        print(stdout.strip())
        final = codex_final(stderr)
        files = sorted({l.split()[2][2:] for l in final if l.startswith("diff --git ")})
        print("final diff touches:", ", ".join(files) or "nothing")
        installs = [l for l in stderr.splitlines() if l.startswith("/bin/bash -lc") and "install" in l]
        print("install commands:", *installs[:5], sep="\n  ")
        if show:
            print("\n".join(diff_of(final, show)))
    else:
        obj = first_json(stdout)
        print(obj.get("result") or obj.get("response") or "(no summary field)")
```

## What to look for, per agent

| Look for | In Codex's log | In a summary |
|---|---|---|
| Template edits (item 2) | a template path in "final diff touches" | "copied unchanged" versus "adjusted", "hardened", "fixed" |
| Suite rewrites (item 3) | a test-file diff; `node --test`, `run-tests`, `test-runtime` in commands | the runner named; the count reported |
| Install behaviour (item 3) | `npm install ... --offline`, `npm cache ls`, `find ... vitest` | "installed vitest as a dev dependency" |
| Boundary (item 4) | the `config` export's diff | the matcher or equivalent quoted |
| Wiring (item 5) | the wiring file's diff | "wired", "wrapped", "lazy", "503" |
| Config (item 6) | `.env.example` and `.gitignore` diffs | which variables it lists; "populated `.env.local`" |
| Handover (item 8) | the plain-text final message | the assumptions list |

A test count different from the documented one is the fastest signal: it means
the agent added, dropped or converted tests. More tests than documented is
usually an extra file of its own beside the shipped ones; check before scoring.

## Rerunning one agent locally

When a summary hints at a deviation it does not show, rerun that agent with
the index's harness to read the actual diff. This needs the agent's CLI and its
credentials on the machine, so it is the exception, not the routine.

```bash
git clone https://github.com/timerise-ai/skills.git <scratchpad>/skills
node <scratchpad>/skills/eval/run.mjs --skill-dir <target skill repo> \
  --agent gemini-cli --prompt 1 --model <the result's model> --log <scratchpad>/gemini.log
# for codex, also --reasoning <the result's reasoningEffort>
# the app folder is printed as "Working in ..."; read it with git diff there
```

A local rerun is for reading only. It installs the skill from the published
repository, so it tests the released version, and it runs on the model the
release's result names, since a CLI's own default can differ from it; its
result file is not committed, because a hand-started run is not the release's
run.

## Checklist

- [ ] The run's three eval jobs succeeded
- [ ] Three result files pulled, `skillVersion` equals the release, each agent's `model` noted
- [ ] Every log read: Codex's final diff, both summaries
- [ ] Any doubt settled by a local rerun or recorded as doubt in the notes
