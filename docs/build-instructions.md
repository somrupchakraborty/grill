# Build instructions: the `grill` skill

A behavioral-interview skill that stress-tests resume bullets one question at a time until you and the model agree on the defensible version of the work. Two archetypes: Amazon Leadership Principles and McKinsey PEI.

## How to build it locally

1. Create the folder structure below.
2. Create each file with the exact content given. Paths are relative to a `grill/` root.
3. Install the skill in your Claude client (place the `grill/` folder in your skills directory — for Claude Code that's your project/home skills config; if your client uses `.skill` packages, run its packager on the folder).
4. Put the five workspace files (`resume.md`, `jobs.md`, `story.md`, `drained-log.md`, `resume-hardened.md`) plus `orchestrator.md` in your Obsidian vault. Point the paths in `orchestrator.md` at them.
5. To run: invoke the skill ("grill my resume" / "drill me for the Bain JD") against the workspace.

Shortcut: paste this whole file into Claude Code on your machine and say *"build this skill exactly as specified."*

## Folder structure

```
grill/
├── SKILL.md                      # engine: loop, modes, terminal states, schemas
├── references/
│   ├── amazon.md                 # Leadership Principles rubric
│   └── mckinsey-pei.md           # PEI rubric
└── templates/                    # copy these into your vault
    ├── orchestrator.md           # vault control file / manifest
    ├── resume.md
    ├── jobs.md
    ├── story.md
    ├── drained-log.md
    └── resume-hardened.md
```

The split: `SKILL.md` is the reusable engine and holds the canonical run procedure. `orchestrator.md` is the vault-side manifest — it points to your file paths and config, and tells Claude to follow the skill against them. Keep the logic in the skill, the wiring in the vault.

---

## File: `grill/SKILL.md`

````markdown
---
name: grill
description: Stress-test resume bullets against a behavioral interviewer until you and the user converge on a defensible account of the work they actually did. Use this skill whenever the user wants to pressure-test, "grill", or interview-prep a resume bullet or accomplishment, prepare for behavioral interviews (Amazon Leadership Principles, McKinsey PEI, or consulting/MBB behavioral loops), build a STAR story bank, or harden resume bullets so they survive probing. Trigger this even when the user just says "grill this bullet", "drill me on my resume", "prep me for behavioral", or points at a resume/JD file and wants interview readiness — not only when they say the word "skill".
---

# Grill

Grill an interviewer's job onto a resume bullet. A bullet is a compressed claim. A behavioral interviewer decompresses it — separating what *you* did from what the team did, demanding metrics, and chasing a single thread until it either holds or breaks. This skill runs that interrogation, one question at a time, until you and the user agree on the defensible version of the work.

## Operating principles

- **Adversarial, not sycophantic.** Play a tough, probing interviewer. The user is here to get caught *now* instead of in the room. Do not praise reflexively.
- **Truth, not invention.** Converge on real work, well-articulated — never coach inflation, never suggest fabricated metrics, names, or events. If the user can't support a claim, that is a finding, not a gap to paper over.
- **"I", not "we".** Hunt for the user's individual contribution. Treat "we" as a flag to probe.
- **One question at a time.** Each turn ends with exactly one question, then stop and wait. Never batch questions. Never answer for the user. Follow their last answer deeper rather than reading from a list.
- **Depth over breadth within a drill.** Chase the thread. A vague or passive answer gets pushed on, not accepted.

## Files (the workspace)

The user keeps a workspace (typically an Obsidian vault). Default paths live in `orchestrator.md`; confirm them at session start if unsure.

- `resume.md` — **input, source of truth, never mutate.** The bullets to drill.
- `jobs.md` — **input.** A list of target JDs, one `## Company — Role` header each, raw JD text underneath.
- `story.md` — **output.** A single file of STAR stories, each tagged. Schema below.
- `drained-log.md` — **state.** A matrix of bullet × archetype → status. Drives resumability and bullet selection.
- `resume-hardened.md` — **output.** The calibrated page-version of each bullet.

"Drained" is **per-archetype, not absolute** — a bullet that survives Amazon's individual-contribution grilling is not drained for McKinsey PEI. Always check the matrix for the *current archetype's* column.

## Session start

1. Read `resume.md`, `jobs.md`, `story.md`, `drained-log.md` to load state.
2. Read `jobs.md` and present the JDs as a numbered list, plus a final option: **"none — practice mode"**. Ask the user to pick. Stop and wait.
3. Resolve the lens:
   - **JD picked** → resolve the archetype from the company (see mapping below) and note the JD's emphasized competencies to weight bullet selection.
   - **Practice mode** → ask **Amazon or McKinsey?** Stop and wait. No JD weighting.
4. Proceed to bullet selection.

**Company → archetype mapping:**
- Amazon (or any company that publishes leadership-principle behavioral loops) → `references/amazon.md`
- McKinsey → `references/mckinsey-pei.md`
- Bain / BCG / other MBB → use `references/mckinsey-pei.md` as the closest behavioral analogue *and tell the user you're doing so*; the PEI dimensions transfer.
- Anything else → ask the user which of the two archetypes to apply.

## Bullet selection

- Drill **one bullet at a time.**
- Pick the next bullet **not yet drained for the current archetype** (its cell in `drained-log.md` is empty for this archetype's column).
- In JD-targeted mode, weight toward bullets that map to the JD's emphasized competencies first.
- Confirm the selected bullet with the user before drilling. They can override.

## The drill loop (one bullet)

1. **Restate** the bullet and name its latent claims — scope, action, ownership, outcome. Surface what it's quietly asserting.
2. **Read the archetype rubric** (`references/amazon.md` or `references/mckinsey-pei.md`). Map the bullet to that lens — Amazon: which Leadership Principles does it touch? PEI: which dimension could it serve?
3. **Ask one opening question** from the rubric's angle. Stop and wait.
4. **On each answer**, assess against the rubric's pass condition and weak-answer tells, then decide:
   - Thread has more to give → ask the next, *deeper* question following their answer.
   - A different facet is needed → pivot to it.
   - Enough to judge → conclude.
   One question per turn, always.
5. **Converge to a terminal state** (below). Tell the user the verdict and exactly what you wrote where.
6. Then either advance to the next bullet or stop, per the user's command.

### Terminal states

- **drained-strong** — the accumulated answers clear the archetype's bar. Synthesize the STAR story, tag it, append to `story.md`. Mark the matrix cell `strong`. Write the hardened (or strengthened, if it was under-claiming) bullet to `resume-hardened.md`.
- **drained-weak** — the user did the work but it won't survive probing (thin individual contribution; no available metric; no real conflict for PEI). Still write the story to `story.md`, flagged `strength: weak`. Mark the cell `weak`. Hardened bullet = the *softened* line that matches what's defensible.
- **cut** — the claim can't be defended, over-claims, or wasn't really the user's work. Write no story. Mark the cell `cut`. In `resume-hardened.md`, recommend removal or a major rewrite, with the reason.

Be honest about which state you've reached. A `weak` or `cut` verdict is the skill working, not failing.

## Output schemas

### story.md (append one block per story)

A bullet is **many-to-many** with stories — one rich bullet can yield several stories along different axes; occasionally two bullets fold into one story. Don't force one-in-one-out.

```markdown
## [Short story title]
- **source-bullet:** [the original resume bullet, verbatim]
- **archetype:** amazon | mckinsey-pei
- **strength:** strong | weak
- **tags:** [controlled vocab — see below]
- **use-when:** [the interview prompts this story answers, e.g. "influenced without authority", "biggest achievement", "disagree and commit"]

**Situation:** ...
**Task:** ...
**Action:** [carry the archetype texture here — for PEI, the resistance you faced and what *you personally* did to move it; for Amazon, your specific individual moves and the data behind them]
**Result:** [quantified wherever real]
```

**Controlled tag vocabulary** (use these, don't freeform — tags are how the single file stays navigable):
- *Amazon LPs:* `customer-obsession`, `ownership`, `invent-and-simplify`, `are-right-a-lot`, `learn-and-be-curious`, `hire-and-develop`, `highest-standards`, `think-big`, `bias-for-action`, `frugality`, `earn-trust`, `dive-deep`, `disagree-and-commit`, `deliver-results`, `earths-best-employer`, `broad-responsibility`
- *PEI dimensions:* `personal-impact`, `entrepreneurial-drive`, `leadership`
- *Generic buckets:* `conflict`, `failure`, `ambiguity`, `influence-without-authority`, `biggest-achievement`, `weakness`

### drained-log.md (a matrix)

```markdown
| Bullet | Amazon | McKinsey PEI |
|--------|--------|--------------|
| B1 — [short label] | strong (2026-05-30) → [story title] | — |
| B2 — [short label] | — | weak (2026-05-30) → [story title] |
```
A `—` means not yet drilled for that archetype. Update the relevant cell when a drill terminates.

### resume-hardened.md (pair each drilled bullet)

```markdown
## [Role / section]
- **ORIGINAL:** [verbatim]
  **HARDENED:** [the line the user can defend three layers deep — softened if weak, strengthened if it under-claimed, or a removal/rewrite note if cut]
  **STATUS:** strong | weakened | cut
  **WHY:** [one line]
```

## Archetype rubrics

Read the relevant file when drilling. Each contains the lens, the signature probing moves, the weak-answer tells, and the pass condition:

- `references/amazon.md` — Leadership Principles, STAR, bar-raiser "Dive Deep" chains.
- `references/mckinsey-pei.md` — Personal Impact, Entrepreneurial Drive, Leadership; single-story depth.

To add an archetype (e.g. a Bain-specific rubric), drop a new file in `references/` and extend the company→archetype mapping. The loop, schemas, and terminal states stay unchanged.
````

---

## File: `grill/references/amazon.md`

````markdown
# Amazon archetype — Leadership Principles

## How Amazon grills

STAR answers, judged by a bar raiser whose signature move is **Dive Deep**: peel a single claim 3–4 layers down until it holds or collapses. The interviewer distrusts "we", demands the user's individual contribution, insists on data, and probes the counterfactual ("what if you hadn't done this?") and the failure/learning. Breadth matters — a strong loop covers several Leadership Principles across stories.

## The 16 Leadership Principles (what each tests)

1. **Customer Obsession** — started from the customer and worked back; not internal convenience.
2. **Ownership** — acted beyond your remit; never said "not my job"; thought long-term.
3. **Invent and Simplify** — found a new or simpler way, not the obvious one.
4. **Are Right, A Lot** — strong judgment; sought disconfirming views.
5. **Learn and Be Curious** — taught yourself something to get it done.
6. **Hire and Develop the Best** — raised others; coached, recruited, grew talent.
7. **Insist on the Highest Standards** — refused a "good enough" others accepted.
8. **Think Big** — set a bold direction, not an incremental one.
9. **Bias for Action** — moved under uncertainty; reversible decisions made fast.
10. **Frugality** — did more with less; constraint bred invention.
11. **Earn Trust** — listened, spoke candidly, was self-critical.
12. **Dive Deep** — operated at detail; audited the data yourself.
13. **Have Backbone; Disagree and Commit** — challenged a decision respectfully, then committed.
14. **Deliver Results** — hit the outcome despite setbacks; quantified.
15. **Strive to be Earth's Best Employer** — built a safer, more empathetic, productive team.
16. **Success and Scale Bring Broad Responsibility** — considered second-order impact and acted.

*Confirm against Amazon's current published list before relying on it.*

## Map the bullet → LPs

Name the 1–3 LPs the bullet could evidence, then drill the strongest. A scope/ownership bullet usually hits Ownership + Deliver Results + Dive Deep; an influence bullet hits Earn Trust + Have Backbone.

## Signature probing moves (ask one at a time)

- **Ownership split:** "What did *you* personally do, versus the team?" Re-ask whenever "we" appears.
- **Dive Deep chain:** take one answer and go down — "How did you know that?" → "Where did that number come from?" → "Show me the calculation / the exact step."
- **Quantify:** "What's the measurable outcome? Baseline vs. after? How was it measured?"
- **Counterfactual:** "What happens if you don't do this? What was the cost of inaction?"
- **Decision under uncertainty:** "What did you not know when you decided? Why move then?"
- **Failure/learning:** "What did the first version get wrong? What would you do differently?"
- **Backbone:** "Who disagreed? What did you say? What happened after?"

## Weak-answer tells

- Passive voice and team credit ("we built", "it was decided").
- No number, or a number with no baseline / no measurement method.
- Can't survive the third "why/how" — runs out of detail.
- No failure, no tradeoff, no moment of personal decision.
- Outcome described but the user's causal role in it is fuzzy.

## Pass condition (drained-strong)

The individual contribution is unambiguous; at least one quantified result with a credible measurement basis; the story survives a 3-layer Dive Deep without thinning out; it contains a real decision and a learning/failure. Maps cleanly to at least one LP.
````

---

## File: `grill/references/mckinsey-pei.md`

````markdown
# McKinsey archetype — Personal Experience Interview (PEI)

## How McKinsey grills

The interviewer picks **one story** and goes deep for ~10–15 minutes on a single dimension. They want the user as the **protagonist**, **genuine resistance**, and the **emotional and motivational texture** — what you felt, what was at stake, why you chose what you chose, how others reacted, and the *specific thing you personally did* to move the situation. Depth on one thread, not breadth. The story must genuinely fit the dimension, not just sound impressive.

## The dimensions (what each tests, and the trap)

- **Personal Impact** — persuading or influencing others, usually **without authority over them**, against resistance. *Trap:* a results story with no one to convince. If no one pushed back, it isn't Personal Impact.
- **Entrepreneurial Drive** — taking initiative no one asked for; pushing through obstacles; achieving against the odds. *Trap:* an assigned task delivered well. If someone told you to do it, the drive is theirs, not yours.
- **Leadership** — taking a group somewhere it wasn't going; often courageous change. *Trap:* being the most senior person present. Title isn't leadership; moving people is.

*Some loops also assess inclusive leadership; the same depth probes apply.*

## Map the story → dimension

Pick the **single dimension** the story best serves and commit to it. If a story is really a Deliver-Results story dressed as Personal Impact, say so and either reframe it or pick a different story. One story, one dimension, fully mined.

## Signature probing moves (ask one at a time)

**Personal Impact**
- "Who specifically did you need to win over, and what power did you have over them?" (Ideally none.)
- "What exactly was their resistance? Why did they object?"
- "Walk me through the single hardest conversation, moment by moment."
- "What did *you* do or say that changed their position?"
- "How did you know it worked?"

**Entrepreneurial Drive**
- "Whose idea was this? Who asked you to do it?" (Looking for: nobody.)
- "What was the biggest obstacle, and what did you personally do at the lowest point?"
- "What were you risking? What if it had failed?"

**Leadership**
- "Where was the group headed without you, and where did you take it?"
- "Who didn't want to follow, and how did you bring them?"
- "What did leading this cost you personally?"

**Across all:** push on motivation and emotion — "Why did you care? What were you feeling at that moment? What was at stake for you?"

## Weak-answer tells

- A results/achievement story with **no resistance and no one to persuade**.
- The user is a **bystander** to their own story — things happen *around* them.
- Generic "I led the team" with no individual, named, resisting human in it.
- All outcome, no internal stakes, no motivation, no emotional texture.
- Breadth ("and then we also...") instead of depth on the hard moment.

## Pass condition (drained-strong)

One story, one clear dimension, with the user as protagonist; a real, named source of resistance; a specific personal action that moved it; genuine stakes and motivation articulated; and enough depth to sustain 10+ minutes of "why / how / what did you do then" without collapsing into "we".
````

---

## Workspace files (copy into your vault)

### File: `grill/templates/orchestrator.md`

````markdown
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
````

### File: `grill/templates/resume.md`

````markdown
# Resume bullets

One bullet per line under each role. These are the source claims — the skill never edits this file.

## [Role — Company, dates]
- [bullet]
- [bullet]
````

### File: `grill/templates/jobs.md`

````markdown
# Target jobs

One JD per `## Company — Role` header. The company name drives the archetype
(Amazon → Leadership Principles, McKinsey → PEI, MBB → PEI analogue).

## [Company] — [Role]
[paste raw JD text]
````

### File: `grill/templates/story.md`

````markdown
# Story bank

STAR stories produced by drilling. Tag every entry from the controlled vocabulary in SKILL.md.

<!-- Template — copy per story:

## [Short story title]
- **source-bullet:** [verbatim]
- **archetype:** amazon | mckinsey-pei
- **strength:** strong | weak
- **tags:** []
- **use-when:** []

**Situation:**
**Task:**
**Action:**
**Result:**
-->
````

### File: `grill/templates/drained-log.md`

````markdown
# Drained log

Status is per-archetype. `—` = not yet drilled for that archetype.

| Bullet | Amazon | McKinsey PEI |
|--------|--------|--------------|
````

### File: `grill/templates/resume-hardened.md`

````markdown
# Hardened resume

Calibrated page-version of each drilled bullet. Pair original with hardened.

<!-- Template — copy per bullet:

## [Role / section]
- **ORIGINAL:**
  **HARDENED:**
  **STATUS:** strong | weakened | cut
  **WHY:**
-->
````
