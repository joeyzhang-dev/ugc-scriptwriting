# UGC Scriptwriting Workspace — agent guide

Joey's writing workspace. He pastes a script, we remix it in the right voice, and
the workspace learns from his edits.

**Markdown and git only.** No database, no API calls, no dependencies, no build
step, no application code. Performance tracking, competitor scraping, and format
discovery live in `~/Developer/ugc-researcher` and stay there. Do not add
integration between the two.

Design spec:
`docs/superpowers/specs/2026-08-18-ugc-scriptwriting-workspace-design.md`

## The model

A script is **niche voice × format structure × topic**.

- `niches/` owns voice — diction, tone, what's allowed, what's cringe
- `formats/` owns structure — the beat-by-beat skeleton
- `library/` holds real scripts — the training data and the source of truth
- `hooks/` is a per-niche bank of hooks that landed
- `drafts/` is scratch space for work in progress

## Niches

| Niche | File | Covers |
|---|---|---|
| Christian | `niches/christian.md` | faith content |
| Self-improvement | `niches/self-improvement.md` | fitness / gym / dieting, finance, college / students |
| Self-improvement (women) | `niches/self-improvement-women.md` | same subjects, female audience and a distinct voice |

The last two share subject matter and differ in audience and voice. **Never treat
them as interchangeable** — that difference is the whole reason voice lives in
its own file per niche.

## Formats

Names match the LLM taxonomy in `ugc-researcher` so one vocabulary covers both
projects. Joey's own phrasing is recorded in each file so either name routes to
the same place.

| Format | File | Joey calls it |
|---|---|---|
| 10/10 list | `formats/10-10-list.md` | "the 10/10 format" |
| S-tier list | `formats/s-tier-ranking.md` | "bad / better / best ranking" |
| Things not to do | `formats/things-not-to-do.md` | "4 things you should NOT be doing if you're a ___" |
| Numbered steps | `formats/numbered-steps.md` | "5 steps to ___" |

When Joey writes in a format with no file, create one — named after the
`ugc-researcher` taxonomy where a matching bucket exists, otherwise a concise
Title Case name that will be reusable.

## Before writing: load context

1. Identify the niche. Ask if genuinely ambiguous — do not guess.
2. Identify the format.
3. Read `niches/<niche>.md`.
4. Read `formats/<format>.md`.
5. Read the 2–3 closest `library/<niche>/` entries in that format.

**Precedence: library examples outrank the format file.** The format file is a
map, not the territory. If a real example contradicts the skeleton, the example
is right and the skeleton is stale — update it.

**Never force a script into a skeleton.** If the material wants a different
shape, the material wins and the format file gets corrected.

## Session shapes

Paste-and-remix is the dominant case and what this workspace is optimized for,
but two others come up:

- **Brief-first** — a topic goes in ("5 steps to fix your sleep, self-improvement"), a script comes out
- **Hook-first** — generate hook variations, Joey picks one, then build the script out from it

All three load context the same way. The difference is only where the session
starts.

## Output contract

Mirrors the doc fields in Joey's webapp so output pastes in without reformatting.
**Omit any field with nothing to say — never emit a placeholder.**

```
INSPO VIDEO: <url>
DEMO TO USE: <what to show on screen>
SONG(S) TO USE: <track>
HOOK: <the hook>

<body>
```

When producing more than one option, label each `VARIATION 1`, `VARIATION 2`, and
so on, on its own line above the block. Leave the block underneath unchanged so
every variation still pastes into the webapp cleanly on its own.

## Learning — automatic, never invisible

Joey's edits are the training signal. When he edits a draft, do not simply accept
it. Work out the *rule* behind the change and write it to the file that owns it:

| What changed | Goes to |
|---|---|
| Word, phrasing, or tone | `niches/<niche>.md` |
| Structure or beat order | `formats/<format>.md` |
| A hook worth keeping | `hooks/<niche>.md` |
| The finished script | `library/<niche>/<slug>.md` |

Rules:

- **Merge, don't append.** A new rule refining an existing one rewrites it in
  place. Appending blindly turns these files into contradictory sludge.
- **Consolidate when a section sprawls.**
- **Report in one line** what was learned and which files changed. Writes are
  automatic, but Joey always sees them.
- **Derive rules from real edits only.** Never invent a voice rule because a file
  looks empty.
- **Never edit a `library/` script to fit a rule.** Scripts are evidence; rules
  are the summary. When they disagree, the rule is wrong.

## Git

Commit after each learning update. `git log` is the learning history and
`git revert` is how a bad lesson gets removed. There is deliberately no changelog
file — git is the changelog.

## Current state

Niche, format, and hook files are scaffolded but **unseeded**. The first working
session is a seeding pass: Joey pastes his existing scripts, each becomes a
`library/` entry, and v1 of the voice rules is derived from that real material.

Until seeding happens, rely on what Joey says in-session — and write it down
afterward.
