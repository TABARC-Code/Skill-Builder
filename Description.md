# Project Description

## The short version

Skill Builder routes skill-work through eleven purpose-built sub-skills instead of one file trying to be everything. It's shaped the way it is because it used to be two systems, built independently by the same author across sessions that never referenced each other, and this version is what happened when both were finally read side by side. Its an upgrade for the Claude-Creator Skill
---
## The problem

Two failure modes recur in any skill-building system that grows past a handful of skills. First, a single skill file trying to hold ideation, engineering rigour, packaging mechanics, and relationship bookkeeping in one document either bloats past what's worth loading every session or degrades to platitudes to stay short. Second, and less obvious: a system that improves itself by rewriting its own instructions has no way to tell the difference between a fix that was made and a fix that was only described. That second problem isn't hypothetical here — it happened to this exact codebase. A prior session reported a rebuild of this skill as complete, and the described changes never reached disk. Nobody caught it until a later session read the files instead of trusting the account of them.

## The approach

Split by phase, not by feature. Each sub-skill owns one stage of a pipeline (classify the request, explore ideas, bridge to a brief, engineer the specification, build and package, audit an existing skill, verify the audit's own claims) and is read on demand, never all at once. Two sidecars (relationship contracts, classification) handle cross-cutting bookkeeping that would otherwise leak into every phase file. Two more (retirement/merge, fleet sweep) handle a skill's end of life and cross-repository defect patterns, which most systems like this don't address at all because they're designed around a skill being built, not around a skill repository being maintained for years.

The governing choice underneath all of it: a claim about the system's own state is not the same as the system's actual state, and the gap between them is where the worst bugs in this kind of self-modifying tooling live. Every phase that changes something ends with a step that re-reads what changed, rather than trusting the log entry that says it happened.

## Architecture

```text
Colony Request Audit (intake gate)
  -> Creative Exploration Engine (divergent ideation)
  -> Skill Handoff Bridge (controlled handshake)
  -> Structured Skill Engineering (specification, layers, eval model, failure taxonomy)
  -> Skill Creator Execution (bundled toolchain: validate, test, describe, package)

Kaizen Audit Loop (improvement entry point)
  -> Skill Validation & Repair (mandatory: re-reads files, doesn't trust the log)

Sidecars, loaded only when live:
  Relationship Contract Engine, Skill Classification Engine,
  Skill Retirement & Merge, Fleet Kaizen Sweep
```

`resources/skill-creator/` is a full copy of the upstream Skill Creator toolchain (scripts, grading agents, JSON schemas, its own eval suite), bundled directly rather than assumed to be separately installed. Four scripts in it (`package_skill.py`, `run_eval.py`, `run_loop.py`, `improve_description.py`) import sibling modules via `scripts.<name>`, so they need module-style invocation; two (`quick_validate.py`, `utils.py`) don't and run standalone.

## Design decisions

**Sub-skills carry no `name`/`description` frontmatter.** The rejected alternative was giving each sub-skill its own installable identity, which is how an earlier version of this system ended up with sub-skills that could trigger independently of the router — a real collision discovered, not a hypothetical one. The cost of the fix is that a sub-skill can never be installed or invoked on its own; that's accepted as correct, not as a limitation to work around.

**Skill Creator's toolchain is bundled, not delegated to an installed sibling skill.** The rejected alternative — "delegate to an installed skill-creator if present" — was what an earlier version of this system actually did, and it meant the packaging and validation machinery only worked when something else happened to be installed too. Bundling costs disk space and duplication risk (this copy can drift from the upstream source); it buys the system working standalone.

**Validation & Repair re-reads files instead of trusting the Kaizen log.** This is the direct, structural response to the rebuild-that-never-landed failure mentioned above. The rejected alternative was tightening the wording of what counts as a valid log entry; that doesn't fix anything if nobody checks the entry against reality. The chosen fix costs an extra verification pass on every Kaizen cycle; it buys a system that can't silently drift from what it claims about itself.

**Two Dewey-style code sources merged rather than one replacing the other.** One development line used broad four-anchor codes (005, 808, 794, 658); the other used fine-grained per-subskill codes within real Dewey neighbourhoods. Both are kept: broad anchors for cross-repository classification, fine codes for skill-builder's own eleven sub-skills.

## Trust and boundaries

Nothing in this skill runs user-supplied code or reaches a network. The bundled scripts read and write files within the skill package directory tree they're pointed at — `quick_validate.py` and the classification/relationship checks are read-only; `package_skill.py` writes a `.skill` zip archive; nothing else writes to disk. Where credentials or external services would matter, there aren't any in this package.

## Current state

Working: the full pipeline (phases 0 through 5a), all four sidecars, the bundled toolchain, and all six repository documentation files — each independently exercised and re-verified against actual file content during this session, not assumed from having written them.

Unknown: how the system performs under real multi-week usage, since it was built and validated in a single extended session rather than across the kind of drift and repeated-error accumulation the Kaizen and Fleet Sweep mechanisms are actually designed to catch. Those mechanisms are implemented and internally consistent; whether they catch what they're meant to catch in practice is not yet observed.

## Operational notes

Meant to be read by a Claude instance, one file at a time, per the routing table in `SKILL.md` — never load the whole package into context for a single task. The bundled Python scripts are meant to be run from a shell with file and bash-tool access, which on claude.ai means Cowork or a similar tool-enabled surface rather than a plain chat.

## Limitations and awkward bits

The `resources/skill-creator/` copy is a snapshot. If the upstream Skill Creator changes, this copy doesn't inherit the change automatically — that's the cost of bundling instead of delegating, paid in full. Four of the nine bundled scripts need module-style invocation and will fail with a confusing `ModuleNotFoundError` if run the obvious way, which is exactly the kind of thing a first-time user will hit and swear at; it's documented in the README's Troubleshooting section rather than fixed at the script level, because fixing it means patching upstream Skill Creator source that this package doesn't own.

There is also, somewhat awkwardly, a structural reason to distrust any future claim this system makes about its own completeness without checking: it's already happened once, to this exact codebase, and the fix for that is a process (Validation & Repair) rather than a guarantee. A process can be skipped. Worth remembering.

## Future direction

No formally scheduled work. Plausible next steps, none of them committed: exercising Fleet Kaizen Sweep against a real multi-skill repository to see whether the root-cause-grouping logic holds up outside the worked example; a script that automates the `scripts/` package wrapper `package_skill.py` currently needs, so the module-invocation quirk stops being a documented workaround and becomes a non-issue.
