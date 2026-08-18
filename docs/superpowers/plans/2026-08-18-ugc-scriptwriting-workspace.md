# UGC Scriptwriting Workspace Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a file-based workspace where Joey workshops UGC scripts across three niches, and which learns his voice from his edits.

**Architecture:** Markdown files organized on two orthogonal axes — `niches/` owns voice, `formats/` owns structure — with `library/` holding real scripts as the source of truth and `AGENTS.md` encoding the routing and learning rules. Git provides the audit trail for everything the workspace learns. No database, no dependencies, no build step.

**Tech Stack:** Markdown, git, bash (for verification only — nothing is committed as a script).

**Spec:** `docs/superpowers/specs/2026-08-18-ugc-scriptwriting-workspace-design.md`

## Global Constraints

- **Markdown and git only.** No database, API, network access, dependencies, build step, or application code.
- **No writing back to `ugc-researcher`.** Alignment with that project is convention only.
- **Format names match the `ugc-researcher` LLM taxonomy** — `10/10 list`, `S-tier list`, `Numbered list`, `How-to method` — so one vocabulary covers both projects.
- **Output contract is exact:** `INSPO VIDEO:` / `DEMO TO USE:` / `SONG(S) TO USE:` / `HOOK:` / blank line / body. Fields with nothing to say are omitted, never filled with placeholders.
- **Voice rules are derived from real edits, never invented.** Scaffolded files ship unseeded with headings and a note explaining what belongs there.
- **Formats are descriptive, not prescriptive.** Library examples in the target niche outrank the format skeleton; where they conflict, the examples win.
- **Merge, don't append.** A new rule that refines an existing one rewrites it in place.
- **Three niches, fixed slugs:** `christian`, `self-improvement`, `self-improvement-women`.
- All files use `---` frontmatter only in `library/` entries; no other file carries frontmatter.

---

### Task 1: Workspace skeleton

Creates the directory tree, git hygiene, and the library entry template that fixes the frontmatter shape all later scripts follow.

**Files:**
- Create: `.gitignore`
- Create: `library/TEMPLATE.md`
- Create: `niches/.gitkeep`, `formats/.gitkeep`, `hooks/.gitkeep`
- Create: `library/christian/.gitkeep`, `library/self-improvement/.gitkeep`, `library/self-improvement-women/.gitkeep`
- Create: `drafts/.gitkeep`

**Interfaces:**
- Consumes: nothing (first task)
- Produces: the directory paths `niches/`, `formats/`, `hooks/`, `library/<niche>/`, `drafts/` that every later task writes into; and the library frontmatter keys `niche`, `format`, `source`, `performance`, `date` that Task 5 relies on.

- [ ] **Step 1: Write the failing structure check**

Run this from `/Users/joey/Developer/ugc-scriptwriting`. It asserts every required path exists:

```bash
check() {
  local fail=0
  for d in niches formats hooks drafts \
           library/christian library/self-improvement library/self-improvement-women; do
    if [ -d "$d" ]; then echo "OK   dir  $d"; else echo "MISS dir  $d"; fail=1; fi
  done
  for f in .gitignore library/TEMPLATE.md; do
    if [ -f "$f" ]; then echo "OK   file $f"; else echo "MISS file $f"; fail=1; fi
  done
  return $fail
}
check; echo "exit=$?"
```

- [ ] **Step 2: Run it to verify it fails**

Expected: every line reads `MISS ...` and the last line is `exit=1`.

- [ ] **Step 3: Create the directories and keepfiles**

```bash
mkdir -p niches formats hooks drafts \
         library/christian library/self-improvement library/self-improvement-women
touch niches/.gitkeep formats/.gitkeep hooks/.gitkeep drafts/.gitkeep \
      library/christian/.gitkeep library/self-improvement/.gitkeep \
      library/self-improvement-women/.gitkeep
```

- [ ] **Step 4: Create `.gitignore`**

```
.DS_Store
*.swp
.vscode/
```

- [ ] **Step 5: Create `library/TEMPLATE.md`**

```markdown
---
niche: christian | self-improvement | self-improvement-women
format: a format name matching a file in formats/
source: mine | competitor
performance: view count, or "unknown"
date: YYYY-MM-DD
---

<!--
Copy this file to library/<niche>/<slug>.md and replace the frontmatter values.

Below the frontmatter goes the script exactly as written or transcribed.

These entries are training data and the source of truth. Never edit a script to
fit a rule. If a rule and a real script disagree, the script wins and the rule
gets updated.
-->
```

- [ ] **Step 6: Re-run the structure check to verify it passes**

Run the `check` function from Step 1 again.
Expected: every line reads `OK ...` and the last line is `exit=0`.

- [ ] **Step 7: Commit**

```bash
git add -A
git commit -m "Scaffold workspace directory structure

Directory tree, git hygiene, and the library entry template that fixes
the frontmatter shape every captured script follows."
```

---

### Task 2: Niche voice files and hook banks

Three niche voice specs and three matching hook banks. Both are per-niche and reviewed on the same judgment, so they ship together. Files are deliberately **unseeded** — headings and guidance only, because rules invented before seeing real scripts would be guesses.

**Files:**
- Create: `niches/christian.md`, `niches/self-improvement.md`, `niches/self-improvement-women.md`
- Create: `hooks/christian.md`, `hooks/self-improvement.md`, `hooks/self-improvement-women.md`
- Delete: `niches/.gitkeep`, `hooks/.gitkeep`

**Interfaces:**
- Consumes: the `niches/` and `hooks/` directories from Task 1.
- Produces: the six file paths above, and the seven `##` section headings in every niche file (`Audience`, `Diction & vocabulary`, `Tone & posture`, `Hook conventions`, `CTA conventions`, `Banned phrases`, `Example lines`) that Task 4's `AGENTS.md` learning table routes edits into.

- [ ] **Step 1: Write the failing content check**

```bash
check_niches() {
  local fail=0
  for n in christian self-improvement self-improvement-women; do
    for f in "niches/$n.md" "hooks/$n.md"; do
      if [ ! -f "$f" ]; then echo "MISS $f"; fail=1; continue; fi
      echo "OK   $f ($(grep -c '^## ' "$f") h2 sections)"
    done
  done
  for n in christian self-improvement self-improvement-women; do
    local c
    c=$(grep -c '^## ' "niches/$n.md" 2>/dev/null || echo 0)
    if [ "$c" -ne 7 ]; then echo "BAD  niches/$n.md expected 7 h2, got $c"; fail=1; fi
  done
  return $fail
}
check_niches; echo "exit=$?"
```

- [ ] **Step 2: Run it to verify it fails**

Expected: six `MISS` lines, three `BAD` lines, last line `exit=1`.

- [ ] **Step 3: Generate the three niche files**

Each niche differs only in its title and audience-scope line, so generate them from one template to keep the three genuinely identical in structure:

```bash
write_niche() {
  local slug="$1" title="$2" covers="$3"
  cat > "niches/$slug.md" <<EOF
# $title

> **Voice spec.** Structure lives in \`formats/\`; this file owns *how it sounds*.
> Covers: $covers
>
> Rules here are derived from real edits during sessions — never invented.
> **Status: unseeded.** Populated during the first seeding session.

## Audience

<!-- Who is watching, what they want, what they are skeptical of. -->

## Diction & vocabulary

**We say:**

**We never say:**

## Tone & posture

<!-- The relationship to the viewer: peer, mentor, confessional, challenger. -->

## Hook conventions

<!-- How this audience specifically gets stopped in the first two seconds. -->

## CTA conventions

<!-- How this audience gets asked, and how hard. -->

## Banned phrases

<!-- AI-tells and cringe, quoted exactly as they should never appear. -->

## Example lines

<!-- Real lines from library/ that nail the voice, each with a pointer to its source file. -->
EOF
}

write_niche christian "Christian" "faith content"
write_niche self-improvement "Self-improvement" "fitness / gym / dieting, finance, college / students"
write_niche self-improvement-women "Self-improvement (women)" "same subjects as self-improvement, written for a female audience in a distinct voice"
```

- [ ] **Step 4: Generate the three hook banks**

```bash
write_hooks() {
  local slug="$1" title="$2"
  cat > "hooks/$slug.md" <<EOF
# $title — hook bank

> Hooks that landed for this niche, grouped by pattern. Added automatically when
> a hook works during a session.
> **Status: unseeded.**

<!--
Entry shape:

## <pattern name>

- "<the hook, verbatim>" — format: <format name> — source: mine | competitor — YYYY-MM-DD
-->
EOF
}

write_hooks christian "Christian"
write_hooks self-improvement "Self-improvement"
write_hooks self-improvement-women "Self-improvement (women)"
```

- [ ] **Step 5: Remove the now-redundant keepfiles**

```bash
rm -f niches/.gitkeep hooks/.gitkeep
```

- [ ] **Step 6: Re-run the content check to verify it passes**

Run `check_niches` from Step 1 again.
Expected: six `OK` lines — the three `niches/*.md` reporting `7 h2 sections` and
the three `hooks/*.md` reporting `1 h2 sections` (the one inside the template
comment). No `BAD` lines, last line `exit=0`.

- [ ] **Step 7: Commit**

```bash
git add -A
git commit -m "Add niche voice specs and hook banks

Three niches, each with a voice spec and a hook bank. Shipped unseeded:
headings and guidance only, because voice rules invented before reading
real scripts would be guesses. Seeding fills them from real material."
```

---

### Task 3: Format files

Four format skeletons matching the formats Joey named, using the `ugc-researcher` taxonomy names. Each carries the descriptive-not-prescriptive warning and a per-niche variations section.

**Files:**
- Create: `formats/10-10-list.md`, `formats/s-tier-ranking.md`, `formats/things-not-to-do.md`, `formats/numbered-steps.md`
- Delete: `formats/.gitkeep`

**Interfaces:**
- Consumes: the `formats/` directory from Task 1.
- Produces: the four file paths above, referenced by name in Task 4's `AGENTS.md` format table. Each file contains six `##` sections (`What it is`, `Beat-by-beat skeleton`, `Length / timing`, `Hook pattern variants`, `Niche variations`, `Examples`) and three `###` niche subsections under `Niche variations`.

- [ ] **Step 1: Write the failing content check**

```bash
check_formats() {
  local fail=0
  for f in 10-10-list s-tier-ranking things-not-to-do numbered-steps; do
    local p="formats/$f.md"
    if [ ! -f "$p" ]; then echo "MISS $p"; fail=1; continue; fi
    local h2 h3
    h2=$(grep -c '^## ' "$p"); h3=$(grep -c '^### ' "$p")
    if [ "$h2" -eq 6 ] && [ "$h3" -eq 3 ]; then
      echo "OK   $p ($h2 h2 / $h3 h3)"
    else
      echo "BAD  $p expected 6 h2 and 3 h3, got $h2 / $h3"; fail=1
    fi
  done
  return $fail
}
check_formats; echo "exit=$?"
```

- [ ] **Step 2: Run it to verify it fails**

Expected: four `MISS` lines, last line `exit=1`.

- [ ] **Step 3: Generate the four format files**

`$name` is the taxonomy name, `$joey` is what Joey calls it in conversation — recording both means either phrasing routes to the same file:

```bash
write_format() {
  local slug="$1" name="$2" joey="$3"
  cat > "formats/$slug.md" <<EOF
# $name

> **Descriptive, not prescriptive.** This file records an observed pattern; it is
> not a template to fill in.
>
> **Library examples in the target niche outrank this skeleton.** Where they
> conflict, the examples are right and this file is stale — update it.
>
> Joey calls this: $joey
> **Status: unseeded.** The skeleton is derived during seeding from real scripts.

## What it is

<!-- One paragraph, written after reading real examples. -->

## Beat-by-beat skeleton

<!-- The observed structure, beat by beat. Derived from library examples, never invented. -->

## Length / timing

<!-- Target runtime and how the beats divide it. -->

## Hook pattern variants

<!-- The opening lines this format tends to use. Cross-reference hooks/<niche>.md. -->

## Niche variations

<!--
How each niche bends this format. Go as deep as the differences demand — the
same skeleton can read very differently across niches.

When one niche's section outgrows the shared skeleton above, graduate it to its
own file: formats/$slug-<niche>.md. Split when reality demands it, never
preemptively.
-->

### Christian

### Self-improvement

### Self-improvement (women)

## Examples

<!-- Links to library/ entries written in this format. -->
EOF
}

write_format 10-10-list "10/10 list" '"the 10/10 format"'
write_format s-tier-ranking "S-tier list" '"bad / better / best ranking"'
write_format things-not-to-do "Things not to do" '"4 things you should NOT be doing if you'"'"'re a ___"'
write_format numbered-steps "Numbered steps" '"5 steps to ___"'
```

- [ ] **Step 4: Remove the now-redundant keepfile**

```bash
rm -f formats/.gitkeep
```

- [ ] **Step 5: Re-run the content check to verify it passes**

Run `check_formats` from Step 1 again.
Expected: four `OK` lines each reporting `6 h2 / 3 h3`, last line `exit=0`.

- [ ] **Step 6: Verify Joey's phrasing survived the quoting**

The `things-not-to-do` heredoc contains a nested apostrophe, which is the one line most likely to have been mangled:

```bash
grep -n 'Joey calls this' formats/*.md
```

Expected, exactly (verified by dry-run):

```
formats/10-10-list.md:9:> Joey calls this: "the 10/10 format"
formats/numbered-steps.md:9:> Joey calls this: "5 steps to ___"
formats/s-tier-ranking.md:9:> Joey calls this: "bad / better / best ranking"
formats/things-not-to-do.md:9:> Joey calls this: "4 things you should NOT be doing if you're a ___"
```

- [ ] **Step 7: Commit**

```bash
git add -A
git commit -m "Add format skeletons

Four formats named after the ugc-researcher taxonomy, each recording
Joey's own phrasing so either name routes to the same file. Marked
descriptive-not-prescriptive with an explicit rule that library examples
outrank the skeleton, plus a documented split path when one niche's
variations outgrow the shared structure."
```

---

### Task 4: AGENTS.md

The brain. Every session reads this file. It is the only file describing behavior rather than content.

**Files:**
- Create: `AGENTS.md`
- Create: `CLAUDE.md` (symlink to `AGENTS.md`)

**Interfaces:**
- Consumes: every path created in Tasks 1–3 — it indexes them by name.
- Produces: the routing, precedence, output-contract, and learning rules that Task 5 rehearses.

- [ ] **Step 1: Write the failing check**

```bash
check_agents() {
  local fail=0
  [ -f AGENTS.md ] || { echo "MISS AGENTS.md"; fail=1; }
  [ -L CLAUDE.md ] || { echo "MISS CLAUDE.md symlink"; fail=1; }
  for needle in "INSPO VIDEO:" "DEMO TO USE:" "SONG(S) TO USE:" "HOOK:" \
                "niches/christian.md" "niches/self-improvement.md" \
                "niches/self-improvement-women.md" \
                "formats/10-10-list.md" "formats/s-tier-ranking.md" \
                "formats/things-not-to-do.md" "formats/numbered-steps.md" \
                "Merge, don't append" "library examples"; do
    if grep -qF "$needle" AGENTS.md 2>/dev/null; then
      echo "OK   contains: $needle"
    else
      echo "MISS contains: $needle"; fail=1
    fi
  done
  return $fail
}
check_agents; echo "exit=$?"
```

- [ ] **Step 2: Run it to verify it fails**

Expected: all lines `MISS`, last line `exit=1`.

- [ ] **Step 3: Create `AGENTS.md`**

````markdown
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
````

- [ ] **Step 4: Create the `CLAUDE.md` symlink**

Joey uses both Copilot CLI (reads `AGENTS.md`) and Claude Code (reads `CLAUDE.md`). A symlink means one file, no drift:

```bash
ln -sf AGENTS.md CLAUDE.md
```

- [ ] **Step 5: Re-run the check to verify it passes**

Run `check_agents` from Step 1 again.
Expected: every line `OK`, last line `exit=0`.

- [ ] **Step 6: Verify the symlink resolves**

```bash
ls -l CLAUDE.md && head -1 CLAUDE.md
```

Expected: `CLAUDE.md -> AGENTS.md` and the first line reads
`# UGC Scriptwriting Workspace — agent guide`.

- [ ] **Step 7: Commit**

```bash
git add -A
git commit -m "Add AGENTS.md workspace brain

Routing rules, the library-outranks-skeleton precedence rule, the output
contract matching the webapp's doc fields, and the learning table that
sends each kind of edit to the file that owns it. CLAUDE.md symlinks to
it so Copilot CLI and Claude Code read one file with no drift."
```

---

### Task 5: End-to-end rehearsal

Nothing so far has been exercised as a *workflow*. This task runs one real pass through the loop against a throwaway script, confirms routing and output, then removes the test artifacts. It exists because a structure that reads correctly can still fail the first time it is used.

**Files:**
- Create then delete: `library/self-improvement/rehearsal.md`
- Modify (only if the rehearsal exposes a gap): `AGENTS.md`, `formats/numbered-steps.md`

**Interfaces:**
- Consumes: `AGENTS.md` routing rules and the output contract from Task 4; the library frontmatter keys from Task 1.
- Produces: no lasting artifacts — a verified workflow and any corrections the rehearsal surfaced.

- [ ] **Step 1: Create a throwaway library entry**

This stands in for a real seeded script so the "read the 2–3 closest library entries" step has something to find:

```bash
cat > library/self-improvement/rehearsal.md <<'EOF'
---
niche: self-improvement
format: Numbered steps
source: mine
performance: unknown
date: 2026-08-18
---

HOOK: 5 steps to fix your sleep before finals week.

Step one, same wake time every day, even Saturday.
Step two, no screens the last hour.
Step three, room cold and pitch black.
Step four, caffeine cutoff is 2pm, not 6pm.
Step five, if you can't sleep in 20 minutes, get up.
EOF
```

- [ ] **Step 2: Verify the entry is discoverable by niche and format**

This is exactly the lookup `AGENTS.md` Step 5 asks for:

```bash
grep -rl "format: Numbered steps" library/self-improvement/
```

Expected: `library/self-improvement/rehearsal.md`

- [ ] **Step 3: Run the rehearsal**

In a fresh session in this directory, give the agent this prompt verbatim:

> Self-improvement niche. Make me two variations of this: "5 steps to fix your sleep before finals week."

Observe and record whether the agent:

1. Read `AGENTS.md`
2. Read `niches/self-improvement.md`
3. Read `formats/numbered-steps.md`
4. Found and read `library/self-improvement/rehearsal.md`
5. Emitted the output contract with `HOOK:` present and **no placeholder** lines for `INSPO VIDEO` / `DEMO TO USE` / `SONG(S) TO USE`
6. Did **not** invent voice rules and write them into `niches/self-improvement.md`

- [ ] **Step 4: Confirm no unintended writes**

```bash
git status --short
```

Expected: only `?? library/self-improvement/rehearsal.md`. Any modification to a
`niches/` or `formats/` file means the agent invented rules from an unseeded
file, which Task 4's "derive rules from real edits only" line is supposed to
prevent.

- [ ] **Step 5: Fix any gap the rehearsal exposed**

If any of the six observations in Step 3 failed, or Step 4 showed unintended
writes, edit `AGENTS.md` to close the specific gap — the routing step that was
skipped, or the rule that was not followed — then repeat Steps 3 and 4 until both
pass. Make no other changes.

- [ ] **Step 6: Remove the rehearsal artifact**

```bash
rm library/self-improvement/rehearsal.md
git status --short
```

Expected: clean, or showing only `AGENTS.md` if Step 5 required a fix.

- [ ] **Step 7: Commit**

If Step 5 changed nothing, there is nothing to commit — the rehearsal passed and
left no trace. Otherwise:

```bash
git add -A
git commit -m "Close routing gaps found in end-to-end rehearsal

First real pass through the loop against a throwaway script surfaced
gaps between what AGENTS.md said and what it actually caused. Fixed."
```

---

## Seeding (first real session, not an implementation task)

The workspace ships unseeded on purpose. The first working session with Joey is
the seeding pass, and it is his session rather than an implementation step:

1. Joey pastes his existing scripts (fewer than ten, currently in Google Docs).
2. Each becomes a `library/<niche>/<slug>.md` entry using `library/TEMPLATE.md`.
3. v1 of each `niches/*.md` voice spec is derived from that real material.
4. Format skeletons are filled in from the structures those scripts actually use.
5. `Status: unseeded` markers are removed from every file that has been populated.

## Success criteria

From the spec, verifiable after seeding:

1. Joey pastes a script and gets a remix that sounds like his niche, without restating voice rules in the prompt.
2. Voice files contain rules derived from real edits, not invented ones.
3. A correction given once is not needed twice.
4. `git log` shows a legible history of what the workspace has learned.
5. Output pastes into the webapp with no reformatting.
