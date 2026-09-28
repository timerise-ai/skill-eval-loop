---
prompts:
  - prompt: "Before we start iterating on one of our skills, write down the rubric you will score its agent eval runs against, and say which items you still need the skill itself to fill in."
  - prompt: "Every agent eval of a skill I maintain passes, but I doubt the agents follow the skill. Tell me how you would score the runs of its latest release and what you need from me to do it; do not fix or release anything."
  - prompt: "In the last evals of a skill I maintain, Codex rewrote the shipped tests to node:assert and still passed. Explain what in a skill usually lets that happen, and where the fix belongs, without changing or releasing anything."
---

# Prompts

What a skill maintainer types after installing this skill, in their own words. An agent eval installs the
skill into an empty Next.js app, gives the agent one of these prompts and no further help, then type-checks,
builds and tests the result; the first prompt runs before every release. The prompts name no target skill:
this skill hardens any skill, so a run shows how the agent applies the loop's contract (the fixed rubric
items, the hard rules, the invocation) when the target is not at hand, and how it asks for what it lacks
rather than inventing a target. This skill works on another skill's repository rather than on the app, so the
checks only confirm the agent left the app intact: what a run shows is in its notes, scored on whether the
agent followed the rubric, the hard rules and the invocation. The results are the other files in this folder.
Section 10 of [STANDARD.md](https://github.com/timerise-ai/skills/blob/main/STANDARD.md) says how a run is
made.
