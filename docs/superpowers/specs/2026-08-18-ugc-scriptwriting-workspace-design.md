# UGC Scriptwriting Workspace — Design

**Date:** 2026-08-18
**Status:** Approved

## Purpose

A file-based workspace where Joey workshops UGC scripts across three niches, and
which gets better at his voice every session. It replaces a Claude web project:
the same "paste a script, remix it" loop, but the accumulated knowledge lives in
version-controlled markdown instead of a chat history that can't be edited,
diffed, or reverted.

The workspace is for **writing**. Performance tracking, competitor scraping, and
format discovery already live in `ugc-researcher` and stay there.

## Context

`~/Developer/ugc-researcher` is a Next.js + Supabase app that scrapes outside
creators, computes view lift, transcribes winners, and runs an LLM pass that
discovers and names formats (`research_videos.format_category`). It also stores
Joey's own scripts in `research_scripts` (`title`, `hook`, `body`, `niche`,
`notes`, `inspo_url`, `demo`, `songs`) and links them to creators and resulting
videos for per-script performance.

**This workspace does not connect to that database.** A read-only bridge was
considered and rejected: it buys smarter cold-start seeding at the cost of
secrets, setup, and a dependency on another project's schema. The workspace stays
offline markdown so it keeps working regardless of what happens to the app.

Two deliberate points of alignment, both convention-only:

- **Format names** reuse the researcher's LLM taxonomy (`10/10 list`,
  `S-tier list`, `Numbered list`, `How-to method`) so one vocabulary covers both
  projects and nothing needs translating.
- **Output shape** mirrors `research_scripts`' doc fields so a finished script
  pastes into the webapp without reformatting.

## Core model: voice × structure

Niche and format are orthogonal.

- **Niche** carries voice: diction, vocabulary, tone, references, what's allowed
  and what's cringe.
- **Format** carries structure: the beat-by-beat skeleton.

A script is *niche voice × format structure × topic*. This is why "5 steps to X"
reads completely differently in Christian versus female self-improvement — same
skeleton, different voice.

Modeling these as separate axes means 3 niches × N formats costs `3 + N` files
instead of `3 × N` templates, and a new format becomes one file that immediately
works across all three niches.

## Directory structure

```
ugc-scriptwriting/
├── AGENTS.md                      # routing, rules, output contract
├── niches/
│   ├── christian.md
│   ├── self-improvement.md
│   └── self-improvement-women.md
├── formats/
│   ├── 10-10-list.md
│   ├── s-tier-ranking.md
│   ├── things-not-to-do.md
│   └── numbered-steps.md
├── hooks/
│   ├── christian.md
│   ├── self-improvement.md
│   └── self-improvement-women.md
├── library/
│   ├── christian/
│   ├── self-improvement/
│   └── self-improvement-women/
└── drafts/
```

Niches cover: Christian; general self-improvement; female self-improvement. The
latter two share subject matter (fitness/gym/dieting, finance, college/students)
and differ in audience and voice — which is precisely why voice lives in its own
file per niche.

## Formats are descriptive, not prescriptive

Joey's formats vary between niches and mutate over time. A format file is a
**record of an observed pattern, not a template to fill in.** Three binding
rules:

1. **Examples outrank the skeleton.** When writing, the primary reference is real
   library examples in that exact niche + format. The format file is a map, not
   the territory. Where they conflict, the examples win.
2. **Never force a script to fit.** If the material wants a different shape, the
   script is right and the format file is out of date. Update the file.
3. **Split when reality demands it.** When one niche's variations section
   outgrows the shared skeleton, that variant graduates to its own file
   (`formats/10-10-list-christian.md`). Formats start unified and earn their way
   apart; they are never split preemptively.

## File anatomy

### `AGENTS.md`

The entry point every session reads. It is the only file that describes
behavior rather than content:

- **Niche and format index** — what exists and when each applies
- **Context loading rule** — which files to read before writing, and the
  precedence rule that library examples outrank format skeletons
- **Output contract** — the block format below
- **Learning rules** — what gets extracted to which file, and the merge-don't-append
  discipline
- **Commit discipline** — commit after each learning update

### `niches/<niche>.md`

- **Audience** — who is watching, what they want, what they're skeptical of
- **Diction & vocabulary** — words and phrases we use; words we never use
- **Tone & posture** — the relationship to the viewer
- **Hook conventions** — how this audience gets stopped
- **CTA conventions** — how this audience gets asked
- **Banned phrases** — AI-tells and cringe, quoted exactly
- **Example lines** — real lines that nail the voice

### `formats/<format>.md`

- **What it is** — one paragraph
- **Beat-by-beat skeleton** — the observed structure
- **Length / timing**
- **Hook pattern variants**
- **Niche variations** — how each niche bends it, with as much depth as needed
- **Examples** — links to `library/` entries

### `hooks/<niche>.md`

A running bank of proven hooks for that niche, grouped by pattern, each tagged
with the format it opened and its source.

### `library/<niche>/<slug>.md`

```yaml
---
niche: christian
format: 10/10 list
source: mine | competitor
performance: <views / "unknown">
date: YYYY-MM-DD
---
```

Body is the script exactly as written or transcribed. These are the training
data; they are never edited to fit a rule.

## Working loop

1. Joey pastes a script and names the niche (or it is inferred and confirmed).
2. Identify the format.
3. Load `niches/<niche>.md`, `formats/<format>.md`, and the 2–3 closest
   `library/<niche>/` examples in that format.
4. Produce the improvement or variations.
5. Joey edits. **His edits are the training signal.**

Sessions may also run brief-first (topic in, script out) or hook-first (generate
variations, pick one, build it out). Paste-and-remix is the dominant case and the
one the workspace is optimized for.

## Output contract

Mirrors the webapp's doc fields so output pastes straight in:

```
INSPO VIDEO: <url>
DEMO TO USE: <what to show on screen>
SONG(S) TO USE: <track>
HOOK: <the hook>

<body>
```

Fields with nothing to say are omitted rather than filled with placeholders.

## Learning mechanism

Learning is **automatic but never invisible.** When Joey edits a draft, the edit
is not simply accepted — the *rule behind it* is extracted and written to the
file that owns it:

| Edit type | Destination |
|---|---|
| Word, phrasing, or tone change | `niches/<niche>.md` |
| Structural change | `formats/<format>.md` |
| A hook worth keeping | `hooks/<niche>.md` |
| The finished script | `library/<niche>/` |

Each session ends with a one-line report of what was learned and which files
changed.

**Rules are merged, not appended.** A new rule that refines an existing one
rewrites it in place. This is what keeps niche files from bloating into
contradictory sludge. When a section grows unwieldy, it gets consolidated.

**Git is the audit trail.** The workspace is a git repo, committed after each
learning update. `git log` is the learning history, `git diff` shows exactly what
rule changed, and a bad lesson is one `git revert` away. This is why no separate
changelog file exists.

## Seeding

The workspace starts nearly empty on purpose — rules invented up front would be
guesses. First real session is a seeding pass: Joey pastes his existing scripts
(fewer than ten, currently in Google Docs), each becomes a `library/` entry, and
v1 of the niche voice files is derived from that real material.

Scaffolded niche, format, and hook files ship with their section headings and a
short note explaining what belongs there, so the structure is legible before it's
populated. Empty directories (`library/*`, `drafts/`) carry a `.gitkeep` so the
shape survives the first commit.

## Non-goals

- No database, API, or network access
- No writing back to `ugc-researcher`
- No build step, dependencies, or application code — markdown and git only
- No performance tracking; that is the researcher's job
- No exhaustive format taxonomy up front; formats are added when written

### Deferred, not rejected

Semantic retrieval over the library — Supabase with pgvector, or any embedding
index — is a plausible later addition. It is deferred rather than ruled out
because it only starts paying for itself once the library is large enough that
finding the right examples beats simply reading the folder. At fewer than ten
scripts, reading the folder wins outright.

Revisit when picking the 2–3 closest examples by hand stops being obvious. The
markdown library stays the source of truth in that world; an index would be a
cache built from it, never a replacement for it.

## Success criteria

1. Joey pastes a script and gets a remix that sounds like his niche, without
   restating voice rules in the prompt.
2. Voice files contain rules derived from real edits, not invented ones.
3. A correction given once is not needed twice.
4. `git log` shows a legible history of what the workspace has learned.
5. Output pastes into the webapp with no reformatting.
