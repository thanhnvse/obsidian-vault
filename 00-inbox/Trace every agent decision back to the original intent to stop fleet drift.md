---
tags: [ai-agents, multi-agent, orchestration, context-engineering]
status: draft
author: claude
source: "https://www.youtube.com/watch?v=AL-PQuB2wy0"
created: 2026-09-30
score: 0.644
review: "parked"
score_reasons: ["atomic: 0.37 (fail)", "links_reasoned: 0.57 (borderline)"]
judged_by: "jev-1.13.0"
judge_rounds: 2
---
# Trace every agent decision back to the original intent to stop fleet drift

## Core idea
When a fleet of coding agents drifts because each step is locally reasonable, the fix is a feedback loop that checks decisions against the human's original intent. Adding rules to individual agents does not fix it. OpenRig (open source) closes that loop in two halves:
- **Refocus**: trace each decision back to the original intent, so drift shows up as a decision with no path back.
- **Productivity monitoring**: measure whether the activity is actually moving the project forward, because a drifting fleet looks busy.

Source caveat: summarised from the video's description and chapter list only; the transcript was not read.

## Why choose / why not
- Choose when: agents run unattended for long enough that nobody reviews each step.
- Don't choose when: a single supervised session. The intent check is the human, and the extra machinery is overhead. → see [[A swarm of coding agents drifts from intent without any agent going rogue]]

## Related
- [[A swarm of coding agents drifts from intent without any agent going rogue]]: the failure mode these countermeasures target.
- OpenRig: https://openrig.dev/ · https://github.com/mvschwarz/openrig
