# Grill + Strengthen workspace

Control file for the grill and strengthen skills. To run: invoke either skill and tell it to follow this workspace.

## File paths
- resume:                 ./resume.md
- jobs:                   ./jobs.md
- story bank:             ./story.md
- drained log:            ./drained-log.md
- hardened resume:        ./resume-hardened.md
- strengthened stories:   ./strengthened-stories.md
- grill rubrics:          [leave blank to use default ../grill/references/ — override only if you've installed the skills in non-standard locations]

## Config
- Practice-mode default archetype (if you don't want to be asked): <amazon | mckinsey | technical-pm | ask>

## How a grill session runs (summary — canonical procedure lives in grill/SKILL.md)
1. Skill reads workspace files to load state.
2. Lists the JDs in jobs.md + a "none — practice mode" option. You pick.
3. JD picked → archetype resolved from the company, bullet selection weighted to the JD.
   Practice mode → you pick Amazon / McKinsey PEI / Technical PM.
4. Skill drills one bullet, one question at a time, to a verdict (strong / weak / cut).
5. Writes story → story.md, status → drained-log.md, hardened line → resume-hardened.md.
6. On weak/cut, offers to hand off to `/strengthen` for a reframing pass.

## How a strengthen session runs (summary — canonical procedure lives in strengthen/SKILL.md)
1. Skill reads workspace + the relevant grill rubric.
2. Lists all `weak` or `cut` stories not yet strengthened. You pick.
3. Diagnoses what went weak against the archetype rubric.
4. Brainstorms collaboratively (one question at a time) through up to four modes:
   honest extraction, dimension reframe, cross-archetype reframe, or different-bullet swap.
5. Writes the strengthened story → strengthened-stories.md, updates drained-log.md cell from `weak` to `strengthened`.
6. Refuses inflation — never invents metrics, named people, resistance, or events the user didn't actually experience.
