# grill + strengthen

A two-skill behavioral-interview toolkit for Claude Code.

- **`grill`** is the adversarial pass. It stress-tests resume bullets one question at a time until you and the model converge on the defensible version of the work you actually did.
- **`strengthen`** is the collaborative pass. After a `weak` or `cut` verdict from grill, it works *with* you to find a defensible reframe of the same real work — without inflating metrics, inventing people, or putting words in resistance you didn't actually face.

Three archetypes ship with grill:

- **Amazon Leadership Principles** — STAR with bar-raiser "Dive Deep" chains.
- **McKinsey PEI** — Personal Impact, Entrepreneurial Drive, Leadership; one story, ~10 minutes deep.
- **Technical PM** — Technical Credibility, Data Literacy & Metrics, Managing Trade-offs, Cross-functional Collaboration; no-code technical behavioral for Meta / Stripe / Google PM-T / FAANG technical-PM loops.

Both skills are anchored on the same principle: truth over invention. A `weak` or `cut` verdict from grill is the skill working, not failing. A partial strengthening from strengthen is honest progress, not a manufactured victory.

## Repo layout

```
.
├── README.md
├── docs/
│   └── build-instructions.md      # the original build spec
├── grill/                         # skill 1 — adversarial drilling
│   ├── SKILL.md                   # engine: loop, modes, terminal states, schemas
│   ├── references/
│   │   ├── amazon.md              # Leadership Principles rubric
│   │   ├── mckinsey-pei.md        # PEI rubric
│   │   └── technical-pm.md        # Technical PM rubric
│   └── templates/                 # copy these into your vault
│       ├── orchestrator.md        # vault control file
│       ├── resume.md
│       ├── jobs.md
│       ├── story.md
│       ├── drained-log.md
│       └── resume-hardened.md
└── strengthen/                    # skill 2 — collaborative reframing
    ├── SKILL.md
    └── templates/
        └── strengthened-stories.md
```

Each `SKILL.md` is a reusable engine. `orchestrator.md` is the shared vault-side manifest — it points to your workspace file paths and tells Claude to follow either skill against them. Keep the logic in the skills, the wiring in the vault.

## Install

### User-level (available from any project — recommended)

```bash
mkdir -p ~/.claude/skills/grill ~/.claude/skills/strengthen
cp -r grill/SKILL.md grill/references grill/templates ~/.claude/skills/grill/
cp -r strengthen/SKILL.md strengthen/templates ~/.claude/skills/strengthen/
```

### Project-level (skills only available inside one project)

```bash
mkdir -p /path/to/project/.claude/skills/grill /path/to/project/.claude/skills/strengthen
cp -r grill/SKILL.md grill/references grill/templates /path/to/project/.claude/skills/grill/
cp -r strengthen/SKILL.md strengthen/templates /path/to/project/.claude/skills/strengthen/
```

### Symlinks if you'd rather updates flow through

```bash
ln -s "$(pwd)/grill" ~/.claude/skills/grill
ln -s "$(pwd)/strengthen" ~/.claude/skills/strengthen
```

## Set up your workspace

The skills run against a shared workspace (typically an Obsidian vault). Drop the templates into your vault:

```bash
cp grill/templates/*.md /path/to/your/vault/
cp strengthen/templates/*.md /path/to/your/vault/
```

Then:

1. Open `orchestrator.md` — confirm the file paths are right (the defaults are relative, so they work if the vault root *is* where the files live).
2. Paste your resume bullets into `resume.md`, one bullet per line under each `## Role — Company, dates` header.
3. Paste target JDs into `jobs.md`, one per `## Company — Role` header.

## Run — grill

From inside your vault:

```
/grill
```

Or just ask: *"grill my resume"*, *"drill me for the Stripe PM JD"*, *"prep me for behavioral."*

The skill will:

1. Read your workspace state.
2. Show your JDs + a `none — practice mode` option.
3. Resolve the archetype from the company (Amazon → LP, McKinsey/MBB → PEI, Meta/Stripe/Google PM-T → Technical PM).
4. Pick a bullet, restate it, then drill one question at a time until it reaches a verdict: `strong`, `weak`, or `cut`.
5. Append a STAR story to `story.md`, update `drained-log.md`, and write a hardened page-version to `resume-hardened.md`.
6. On `weak` or `cut`, offer to hand off to `/strengthen` for a reframing pass.

"Drained" is per-archetype — a bullet that survives Amazon's individual-contribution grilling is not drained for McKinsey PEI or Technical PM.

## Run — strengthen

After a `weak` or `cut` verdict, or any time you want to rework a story:

```
/strengthen
```

Or just say: *"strengthen this story"*, *"reframe this bullet"*, *"make this defensible"*, *"salvage this."*

The skill will:

1. List all `weak`/`cut` stories from `story.md` and `resume-hardened.md` that haven't been strengthened yet.
2. Diagnose what specifically went weak against the archetype rubric — naming the dimension, the weak-answer tell, and the unmet pass condition.
3. Brainstorm collaboratively (one question at a time) through up to four modes:
   - **Honest extraction** — drill a dimension the original pass missed
   - **Reframe through a different rubric dimension** within the same archetype
   - **Reframe across archetypes** — sometimes a Personal Impact story is being drilled as Amazon Ownership and just needs to switch to PEI
   - **Suggest a different bullet** — if the underlying work doesn't have what the archetype needs, propose other bullets that do
4. Refuse inflation explicitly. If you propose rounding 24% up to 30%, or naming a CTO who wasn't involved, the skill names it as inflation, refuses, and offers a truthful alternative.
5. Write the reframed story to `strengthened-stories.md` with a `WHY NOW DEFENSIBLE` explanation and an optional `STILL NEEDS` flag for residual risk in the room.
6. Update `drained-log.md` to mark the cell as `strengthened`.

If the user explicitly asks for external grounding (e.g. *"what is McKinsey actually probing in PEI right now?"*), the strengthen skill can spawn a one-shot research subagent and incorporate the findings into the next brainstorm turn. Use sparingly.

## Privacy

The workspace files (`resume.md`, `jobs.md`, `story.md`, `drained-log.md`, `resume-hardened.md`, `strengthened-stories.md`, `orchestrator.md`) at the **repo root** are gitignored by default. If you ever instantiate the templates into the repo dir itself instead of a separate vault, your real resume content stays out of git.

The templates inside `grill/templates/` and `strengthen/templates/` contain no personal data and are safe to commit.

## Adding an archetype

To add a new behavioral rubric (e.g. a Bain-specific one), drop a new file in `grill/references/` and extend the company → archetype mapping in `grill/SKILL.md`. The drill loop, schemas, terminal states, and the strengthen skill all stay unchanged — they're archetype-agnostic.

## License

No license file is set yet — pick one before pushing publicly if you want others to reuse it.

## Credits

Built from the spec in [`docs/build-instructions.md`](docs/build-instructions.md), extended with Technical PM archetype and the strengthen companion skill.
