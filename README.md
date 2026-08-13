# Skill Builder

A meta-skill for building other Claude skills. Yes, it's recursive. No, that's not a problem — it's the point.

Skill Builder turns a rough idea into a reusable, tested, classified, linked skill, and then keeps it honest after it ships. It's a router: read the root, pick the phase or sidecar the job needs, read that one file, do the work. Never the whole package in context at once.

Author:TABARC-Code
---

## What it does

Routes skill-building work through eleven sub-skills instead of one swollen file. A new idea goes through intake classification, creative exploration, a controlled handoff into an engineering brief, structured engineering against a real specification (hierarchy layer, abstraction level, failure taxonomy, eval scoring), then a Skill Creator execution phase that drafts, tests, and packages the result using a genuinely bundled toolchain — real scripts, not a hoped-for install.

An existing skill that needs fixing skips straight to a Kaizen audit loop, and every Kaizen pass has to close through a validation step that re-reads the actual files before the fix is logged as done. That last part exists because it was needed: an earlier version of this exact system was once reported fixed without the fix ever reaching disk.

Skills that have run their course get a formal retirement or merge, with redirected links and a stub left behind rather than a silent deletion. A defect found in one skill can be swept across the whole installed repository to check whether it's local or structural.

## Status

Implemented and validated against the bundled Skill Creator toolchain: the full 0 to 5a pipeline, all four sidecars (relationship contracts, classification, retirement/merge, fleet sweep), the six repository documentation files, and the bundled `resources/skill-creator/` scripts.

Not implemented: no CI configuration, no automated test runner beyond the bundled eval scripts, no packaged release history before this version — the version numbering starts where it starts.

## Why it exists

Two separate development lines of this system existed and never reconnected: one built process discipline around self-auditing and disk-verified fixes, the other built a genuinely deep engineering methodology and a real bundled toolchain. Neither knew about the other. This package is the reunion, and it exists so the next skill built with it doesn't have to choose between those two strengths.

## Features

Implemented:
- intake classification before any work starts (Scout, Follower, Memory, Outlier posture)
- creative exploration with a fixed Creative Draft output format
- a handoff bridge with an explicit readiness rule, not a vibe check
- structured engineering with hierarchy layers, abstraction levels, a weighted eval scoring model, and a ten-category failure taxonomy
- a bundled Skill Creator toolchain: validation, eval running, description tuning, packaging
- a Kaizen audit loop that closes through a mandatory file-content verification step
- a relationship contract system with a five-question Link Validity Test, not backlink vibes
- Dewey-style classification with a quiet description tag on every sub-skill
- formal retirement and merge, with verified redirects
- a repository-wide sweep for defects that turn out not to be local

Planned: nothing formally scheduled. This is a working system, not a roadmap with a working demo attached.

## How it works

```text
Colony Request Audit
  → Creative Exploration → Handoff Bridge → Structured Engineering → Skill Creator
  ↔ Relationship Contract Engine (when links change)
  ↔ Skill Classification Engine (when identity/listing changes)

Improving something that already exists:
Colony Request Audit → Kaizen Audit Loop → (Structured Engineering or Skill Creator, as needed) → Validation & Repair
```

Sub-skill files carry no `name` or `description` frontmatter, which is deliberate: it's what stops them triggering standalone, independent of this router. `ROUTER_GATING_EVALS.md` defines the tests for this; `evals/router-gating-results.md` records which of them have actually been run, and which haven't.

## Requirements

- A Claude surface that supports skills with file-creation and bash tools available (Claude Code, or Claude.ai/Cowork with those tools enabled). Routing and drafting work without them; the bundled scripts, packaging, and validation don't.
- Python 3 for the bundled scripts under `resources/skill-creator/scripts/`.
- PyYAML (`pip install pyyaml`), for frontmatter parsing in `quick_validate.py` and related scripts. No other third-party Python dependency observed in the bundled scripts.

## Installation

Copy or clone the `skill-builder/` directory into wherever your Claude environment reads installed skills from. The exact path depends on your setup (Claude Code, Claude.ai, Cowork all differ, and Claude Code's own default has moved between versions) — check your environment's current skills documentation rather than guessing the path from this README. No build step, no package manager install, for the skill itself once it's in the right place.

## Setup

Nothing to configure before first use. The bundled toolchain expects to be run from inside the `skill-builder/` directory, or with paths given relative to it — see Usage below.

## First successful run

```text
$ python3 resources/skill-creator/scripts/quick_validate.py .
Skill is valid!
```

Run from inside the `skill-builder/` directory. That line is what a clean install looks like.

## Usage

Ask Claude to build, improve, retire, or audit a skill, and mention `skill-builder` — or just describe the work ("turn this workflow into a skill", "this skill keeps getting the date wrong", "retire this old skill into the new one") and let the router pick it up from the description match.

To run the bundled tools directly:

```text
python3 resources/skill-creator/scripts/quick_validate.py <skill-path>
```

Packaging and a few other scripts (`package_skill.py`, `run_eval.py`, `run_loop.py`, `improve_description.py`) import sibling modules as `scripts.<name>`, which means they need to be run as a module from a directory that treats `scripts/` as a package, not invoked directly by file path. See Troubleshooting.

Full worked example, idea to shipped skill to a later Kaizen pass to eventual retirement: `EXAMPLE_WORKFLOW.md`. Fast path versus proper path versus improvement path: `USAGE.md`.

## Configuration

No environment variables or config files. Behaviour is controlled entirely by which phase or sidecar file gets read, per `SKILL.md`'s routing rules.

## Security

Nothing in this skill executes arbitrary user-supplied code — the bundled scripts operate on skill package files (Markdown, JSON, YAML frontmatter) on disk. `package_skill.py` writes a `.skill` zip archive; it doesn't fetch anything over a network or accept credentials. No secrets, tokens, or external service calls anywhere in the bundled toolchain.

## Troubleshooting

**`ModuleNotFoundError: No module named 'scripts'` when running `package_skill.py` directly.**
Cause: the script imports sibling modules as `scripts.quick_validate` and similar, which only resolves when Python treats the scripts directory as a package. Check: did you run it as `python3 resources/skill-creator/scripts/package_skill.py <path>` directly? Fix: run it as a module instead, from a working directory one level above a `scripts/` folder that contains an `__init__.py` — `python3 -m scripts.package_skill <path>`. The same applies to `improve_description.py`, `run_eval.py`, and `run_loop.py`. `quick_validate.py` and `utils.py` don't have this problem and run fine invoked directly.

**A sub-skill seems to trigger on its own, without going through the router.**
Cause: it's picked up a `name`/`description` frontmatter block it shouldn't have. Check: `grep -l "^name:" subskills/*.md` should return nothing. Fix: strip the frontmatter — sub-skills are plain reference files, never independently installable.

**`run_loop.py` doesn't do anything useful.**
Cause: it needs the `claude` CLI, which means Claude Code specifically. On claude.ai or Cowork, run the eval loop by hand: follow the skill's own instructions against each test prompt and compare the output yourself.

## Known limitations

No automated CI. Eval running outside Claude Code is manual, not scripted. No release history exists before the version this package ships as — that's not an omission, there genuinely isn't one.

## Development notes

The eleven sub-skills are flat `.md` files under `subskills/`, not folders with their own `SKILL.md` — this is what keeps the Skills API from treating any of them as a second installable skill. The bundled Skill Creator toolchain under `resources/skill-creator/` is a full copy of the upstream scripts, agents, and schemas, kept separate from the main skill body so reading `SKILL.md` doesn't mean loading all of it.

## Licence

Not specified in this package.
