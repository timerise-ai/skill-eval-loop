---
prompts:
  - prompt: "Before we start iterating on our timerise-ai/site-pin-gate skill, write down the rubric you will score its agent eval runs against."
  - prompt: "Every agent eval of timerise-ai/site-pin-gate passes, but I doubt the agents follow the skill. Score the runs of its latest release; do not fix or release anything."
  - prompt: "In the last evals of timerise-ai/site-pin-gate, Codex rewrote the shipped tests to node:assert and still passed. Find what in the skill let it, and fix the skill without releasing."
---

# Prompts

What a skill maintainer types after installing this skill, in their own words. An agent eval installs the
skill into an empty Next.js app, gives the agent one of these prompts and no further help, then type-checks,
builds and tests the result; the first prompt runs before every release. This skill works on another skill's
repository rather than on the app, so the checks only confirm the agent left the app intact: what a run
shows is in its notes, scored on whether the agent followed the rubric, the hard rules and the invocation.
The results are the other files in this folder. Section 10 of
[STANDARD.md](https://github.com/timerise-ai/skills/blob/main/STANDARD.md) says how a run is made.
