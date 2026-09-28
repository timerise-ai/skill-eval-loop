# Shipping a round

A round ends in a patch release, and the release is what starts the next
round's evals. Verify the templates, commit, write the changelog, tag, push,
publish, and hand the wait to a background watch.

## Verify the templates

A skill whose references carry code blocks names their destination on the
first line (`// file: lib/x/core.ts`). Extract them into a scratch project and
run the checks the target's `CLAUDE.md` names. The files and the tsconfig
flags come from that recipe; the extractor below is generic.

```python
# file: extract_blocks.py
"""Write every fenced ts/tsx block whose first line is `// file: <path>` to <out>/<path>.

Usage: python3 extract_blocks.py <out-dir> <reference.md> [<reference.md> ...]
"""
import os
import re
import sys

out, sources = sys.argv[1], sys.argv[2:]
# `{3} rather than three literal backticks, which would close this block when it is itself in markdown.
pattern = re.compile(r"`{3}(?:ts|tsx)\n(// file: (\S+)\n.*?)`{3}", re.S)
for source in sources:
    with open(source) as handle:
        for match in pattern.finditer(handle.read()):
            path = os.path.join(out, match.group(2))
            os.makedirs(os.path.dirname(path), exist_ok=True)
            with open(path, "w") as target:
                target.write(match.group(1))
            print(match.group(2))
```

```bash
S=<scratchpad>/proj
mkdir -p "$S" && python3 extract_blocks.py "$S" references/module.md references/handler.md references/testing.md
cd "$S"
# first time only: npm init -y && npm i -D typescript next vitest @types/node @types/react, plus the tsconfig the recipe names
npx tsc --noEmit && echo TSC OK
bun test lib/<module> 2>&1 | tail -3
npx vitest run lib/<module> 2>&1 | grep Tests
```

Keep the scratch project between rounds and re-extract over it. Both runners
must report the same count, and it must equal the count every doc states. A
new test changes that number in the README, the testing reference and
`CLAUDE.md` in the same commit.

A round that changed prose only still re-runs the check: a shared identifier
can drift in prose that a later reader copies into code.

## Changelog and version

Follow the target's release convention; for the Timerise skills that is a
patch release per round:

```bash
git status --porcelain                 # must be empty
git describe --tags --abbrev=0         # vX.Y.Z of the last round
git log --oneline "$(git describe --tags --abbrev=0)"..HEAD
```

The bump is a patch unless a fix breaks existing usage. `chore(evals)` commits
never count toward it. Add a section to the top of `CHANGELOG.md`:

```markdown
## [0.3.6] - 2026-09-27

Security fix release, from scoring the prompt-1 agent eval runs against 0.3.5.

### Security

- <what was exploitable, how, and what apps built from earlier versions must copy in>

### Changed

- <each rule or wording change, and the file that carries it>
```

Update the "current release" line in the README, then commit only those two
files as `chore(release): X.Y.Z`.

## Tag, push, publish

```bash
V=0.3.6
git tag -a "v$V" -m "v$V"               # annotated: push --follow-tags skips lightweight tags
git cat-file -t "v$V"                   # prints "tag"
git push --follow-tags
git ls-remote --tags origin "v$V"       # the tag reached the remote
gh release create "v$V" --repo "$REPO" --title "v$V" \
  --notes "$(awk -v v="$V" '$0 ~ "^## \\[" v "\\]" {f=1; next} /^## \[/ {f=0} f' CHANGELOG.md)"
sleep 15 && gh run list --repo "$REPO" --limit 1   # the eval run, event "release"
```

Pushing a tag does not start the evals; publishing the release does. Then
start the background watch from [collecting.md](collecting.md).

## A round with nothing to fix

When every agent already scores full but the minimum number of rounds is not
reached, do not release: evals never bump a version. Re-run the same version
by dispatching each agent:

```bash
for a in claude-code codex gemini-cli; do
  gh workflow run agent-eval.yml --repo "$REPO" -f agent="$a" -f prompt=1
done
```

The workflow's concurrency group queues them one after another. Each run
commits its own result file; collect them as a round.

## Commit hygiene

- Stage named files only; never `git add -A` in a skill repository.
- Follow the maintainer's commit convention for trailers. The source session's
  maintainer forbids AI attribution trailers in commit messages; check
  `git log -1 --format=%B` after each commit.
- Never push anything from the scratch project or the logs directory.

## Checklist

- [ ] Templates extracted, type-checked, and both runners report the documented count
- [ ] `fix(skill)` commit, then `chore(release): X.Y.Z` with only the changelog and README
- [ ] Annotated tag, pushed, confirmed on the remote
- [ ] Release published with its changelog section as notes
- [ ] The eval run started, and a background watch on it
