# Grill workspace

Control file for the grill skill. To run: invoke the grill skill and tell it to follow this workspace.

## File paths
- resume:           ./resume.md
- jobs:             ./jobs.md
- story bank:       ./story.md
- drained log:      ./drained-log.md
- hardened resume:  ./resume-hardened.md

## Config
- Practice-mode default archetype (if you don't want to be asked): <amazon | mckinsey | ask>

## How a session runs (summary — canonical procedure lives in the skill)
1. Skill reads all files above to load state.
2. Lists the JDs in jobs.md + a "none — practice mode" option. You pick.
3. JD picked → archetype resolved from the company, bullet selection weighted to the JD.
   Practice mode → you pick Amazon or McKinsey.
4. Skill selects the next bullet not yet drained for that archetype, confirms it, and drills one question at a time to a verdict (strong / weak / cut).
5. Writes story → story.md, status → drained-log.md, hardened line → resume-hardened.md.
