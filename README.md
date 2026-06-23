# Reviewer skills — source repository

This repository is the **authoring home** for a family of self-contained, AI-Assisted-Coding reviewer skills. If you are an agent or a person about to edit anything here, read this first — the way these skills are deployed changes how you must edit them.

## Use these reviewers with `scsh`

These reviewers are built to run under **`scsh`** (Scoped Skills Helper) — it runs each skill in parallel, in an ephemeral container, on a clean clone of your repo. Install the whole family into any repository with one command:

```sh
scsh installskills https://github.com/dimacurrentai/code-review-skills
```

`scsh` reads this repo's `.scsh.yml`, copies the five reviewers into your `.skills/`, and merges each reviewer's YAML block **verbatim** (including the `invocations:` matrix) into your own `.scsh.yml`. At run time, `scsh run --profile code-review` expands those five skills into fifteen invocations named `{reviewer}-{route}`. It also wires the per-harness discovery symlinks (`.claude/skills`, `.codex/skills`, `.cursor/skills`, ...) and gitignores `tmp/` in that target repo — so this source repo ships no symlinks of its own. It does keep a local `tmp/` `.gitignore` for in-repo sanity, but that is authoring infra and is not installed into consumers. You can review a branch right away:

```sh
scsh run --profile code-review     # up to fifteen reviewers (five × three model routes), over origin/main..HEAD
```

When all three model routes are available on the host — GPT via opencode (`openai/gpt-5.5`), Opus via claude (`claude-opus-4-8`), and GLM-5.2 via opencode (`nebius-glm/zai-org/GLM-5.2`) — the fleet runs fifteen invocations in parallel. scsh skips routes whose harness or model is unavailable.

The authoring-only **`internal-self-check-reviewers`** is **not** installed (its `internal-` name marks it internal to this repo). Everything below is for working **on** the skills here.

## How these skills are deployed

Each reviewer is a standalone skill in its own directory under `.skills/` (`.skills/conventions-reviewer/`, `.skills/justification-reviewer/`, `.skills/reviewability-reviewer/`, `.skills/sanity-reviewer/`, `.skills/testing-reviewer/`), defined by a `SKILL.md`. A skill is shipped by **copying its directory, by itself, into a target repository**. Nothing else in this repo travels with it — in particular, `PRINCIPLES.md` does **not** go along. (`scsh installskills` automates exactly this copying — plus the `.scsh.yml` merge — see *Use these reviewers with `scsh`* above.)

## What this means when you edit a skill

- **Skills operate by themselves.** Each `SKILL.md` must be complete on its own: every rule it relies on has to be written into the skill, because nothing outside the copied directory exists at runtime.

- **Duplication is essential, not a smell.** The same rules — the special author, the output contract, tone, human-in-the-loop, the correctness baseline, and so on — appear both in `PRINCIPLES.md` and inside each skill on purpose. Do **not** try to "DRY it up" by pointing skills at a shared file or factoring the repetition out.

- **A skill must never reference `PRINCIPLES.md`.** That file does not exist in the target repo. A skill that mentions it, links to it, or assumes it is present is broken.

## What `PRINCIPLES.md` is for

It is the **canonical specification, kept here for the author's sanity** — not a runtime artifact, and never shipped. It outlines the **principles** every reviewer skill must follow. Its only job: when you edit a skill, check it against those principles to confirm it still honors every one and does not contradict the other skills. Treat the principles as the source of truth, and treat each skill as a self-contained restatement of the parts it needs.

## Running and self-checking the family

`.scsh.yml` at the repo root is a skills-only manifest listing five reviewers (each with an `invocations:` matrix for three model routes) plus `internal-self-check-reviewers`. Consumers receive the same matrix after `scsh installskills`; run time expands it to fifteen invocations. Every skill is in a profile, so a bare `scsh run` (the default profile) is intentionally a **no-op** — scsh just lists the available profiles. The family auditor runs under **`scsh run --profile internal-self-check`** — it reads `PRINCIPLES.md`, the manifest, and each reviewer's `SKILL.md` and verifies the family still conforms: the manifest is complete, and every reviewer is self-contained and faithful to the principles in `PRINCIPLES.md`. Run it after committing. `.skills/internal-self-check-reviewers/` is authoring-only and is never copied to a target repo (its `internal-` name keeps `scsh installskills` from shipping it). See `PRINCIPLES.md` section 8.

## Conventions in this repo

- All Markdown files, including this one, use **one long line per paragraph** — no hard-wrapping within a paragraph. (See `PRINCIPLES.md`, line 3.)

- `tmp/` holds reviewer and self-check output (`tmp/code-review-<skill-name>.json`, `tmp/internal-self-check-reviewers.json`); `scsh installskills` gitignores `tmp/` in the consumer so a skill writing its result there never dirties the working tree.
