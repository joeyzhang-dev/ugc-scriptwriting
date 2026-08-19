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

**How Joey iterates.** He works label-first and in rounds: he supplies a finished
script, a draft, or structural direction; you identify CTA placement, write the
bridge copy, and reassemble the *full* script; he selects an option or redirects;
you refine. List points get a short label first, then the beat-level build-out
underneath. Expect several rounds on individual points before one lands — that is
the process working, not a failure.

## Writing discipline

- **Match the reference script's length.** When Joey supplies a reference or a
  draft, the output tracks its length. Running long is the single most common
  complaint — he pushes back on it consistently. More words is not more value.
- **Flag risks proactively.** Call out platform or content risk, logical gaps,
  factual problems, and typos without being asked. Joey catches these himself and
  expects them caught first.
- **Reassemble the whole script.** When changing part of a script, return the
  complete thing, not just the changed fragment.

## Output contract

Mirrors the doc fields in Joey's webapp so output pastes in without reformatting.
**Omit any field with nothing to say — never emit a placeholder.**

```
INSPO VIDEO: <url>
DEMO TO USE: <what to show on screen>
SONG(S) TO USE: <track>
TEXT HOOK: <the on-screen text>
HOOK: <the spoken hook>

<body>
```

`TEXT HOOK` and `HOOK` are different lines and are written separately. The
on-screen text can carry things speech can't — the seeded Christian script uses
scare quotes around "Christian" to signal a challenge before a word is spoken.
Never duplicate one into the other.

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

Seeding is in progress.

| Niche | State |
|---|---|
| Christian | **v1 seeded** from 2 real scripts, both `Things not to do`. Voice spec, hook bank, and the format's Christian section are populated. |
| Self-improvement | unseeded |
| Self-improvement (women) | unseeded |

Formats other than `Things not to do` are still unseeded skeletons.

For unseeded niches and formats, rely on what Joey says in-session — and write it
down afterward. Do not fill an empty section by generalizing from the Christian
material; the niches differ in voice by design.

Sections marked `UNCONFIRMED` or `HYPOTHESIS` in any file are inferences awaiting
Joey's confirmation. Do not promote them to rules until he confirms them.
