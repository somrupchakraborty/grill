# grill

An adversarial behavioral-interview skill for Claude Code. Stress-tests resume bullets one question at a time until you and the model converge on the defensible version of the work you actually did.

Three archetypes ship with it:

- **Amazon Leadership Principles** — STAR with bar-raiser "Dive Deep" chains.
- **McKinsey PEI** — Personal Impact, Entrepreneurial Drive, Leadership; one story, ~10 minutes deep.
- **Technical PM** — Technical Credibility, Data Literacy & Metrics, Managing Trade-offs, Cross-functional Collaboration; no-code technical behavioral for Meta / Stripe / Google PM-T / FAANG technical-PM loops.

The skill never invents metrics, never coaches inflation, and treats `we` as a flag to probe. A `weak` or `cut` verdict is the skill working, not failing.

## Repo layout

```
.
├── SKILL.md              # the engine: loop, modes, terminal states, schemas
├── references/
│   ├── amazon.md         # Leadership Principles rubric
│   └── mckinsey-pei.md   # PEI rubric
├── templates/            # copy these into your vault
│   ├── orchestrator.md   # vault control file
│   ├── resume.md
│   ├── jobs.md
│   ├── story.md
│   ├── drained-log.md
│   └── resume-hardened.md
└── docs/
    └── build-instructions.md   # the original build spec
```

`SKILL.md` is the reusable engine. `orchestrator.md` is the vault-side manifest — it points to your file paths and tells Claude to follow the skill against them. Keep the logic in the skill, the wiring in the vault.

## Install

### Project-level (skill only available inside one project)

```bash
mkdir -p /path/to/your/project/.claude/skills/grill
cp -r SKILL.md references templates /path/to/your/project/.claude/skills/grill/
```

### User-level (available from any project)

```bash
mkdir -p ~/.claude/skills/grill
cp -r SKILL.md references templates ~/.claude/skills/grill/
```

If you'd rather symlink so updates flow through:

```bash
ln -s "$(pwd)" ~/.claude/skills/grill
```

## Set up your workspace

The skill runs against a workspace (typically an Obsidian vault). Drop the six templates into your vault:

```bash
cp templates/*.md /path/to/your/vault/
```

Then:

1. Open `orchestrator.md` — confirm the file paths are right (the defaults are relative, so they work if the vault root *is* where the files live).
2. Paste your resume bullets into `resume.md`, one bullet per line under each `## Role — Company, dates` header.
3. Paste target JDs into `jobs.md`, one per `## Company — Role` header.

## Run

From inside your vault (or wherever the workspace files live):

```
/grill
```

Or just ask Claude: *"grill my resume"*, *"drill me for the Bain JD"*, *"prep me for behavioral."*

The skill will:

1. Read your workspace state.
2. Show your JDs + a `none — practice mode` option.
3. Resolve the archetype from the company (Amazon → LPs, McKinsey/MBB → PEI).
4. Pick a bullet, restate it, then drill one question at a time until it reaches a verdict: `strong`, `weak`, or `cut`.
5. Append a STAR story to `story.md`, update `drained-log.md`, and write a hardened page-version to `resume-hardened.md`.

"Drained" is per-archetype — a bullet that survives Amazon's individual-contribution grilling is not drained for McKinsey PEI.

## Privacy

The workspace files (`resume.md`, `jobs.md`, `story.md`, `drained-log.md`, `resume-hardened.md`, `orchestrator.md`) at the **repo root** are gitignored by default. If you ever instantiate the templates into the repo dir itself (instead of a separate vault), your real resume content stays out of git.

The templates inside `templates/` contain no personal data and are safe to commit.

## Adding an archetype

To add a new behavioral rubric (e.g. a Bain-specific one), drop a new file in `references/` and extend the company→archetype mapping in `SKILL.md`. The loop, schemas, and terminal states stay unchanged.

## License

No license file is set yet — pick one before pushing publicly if you want others to reuse it.

## Credits

Built from the spec in [`docs/build-instructions.md`](docs/build-instructions.md).
