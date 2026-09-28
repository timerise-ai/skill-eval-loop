# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0] - 2026-09-28

First release.

### Added

- `SKILL.md`: the loop from a release's automatic agent evals to a full score,
  with five critical facts, six hard rules, the invocation table and the quick
  start.
- `references/rubric.md`: the eight-item fidelity rubric, how to derive its
  skill-specific items, what is not scored, and how scores are written into
  result bodies.
- `references/collecting.md`: finding, waiting for and downloading a release's
  eval run, each agent's model noted per round, and `read_logs.py`, which
  prints each agent's summary and Codex's final diff.
- `references/fixing.md`: the root-cause table for deviations, the
  reproduce-before-adopt procedure, and where each kind of fix goes.
- `references/releasing.md`: the template check with `extract_blocks.py`, the
  patch release, and the dispatch round.
- `references/loop.md`: rounds, the stop rule, autonomy and the reports.
- `references/provenance.md`: the session the loop was worked out in, with
  its scores per round, what was kept deliberately and what was added.
