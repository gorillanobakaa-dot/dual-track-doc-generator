# dual_track.py — IBM/DITA Dual-Track Documentation Generator

**Location**: `SECOND.BRAIN/Scripts/documentation/dual-track/dual_track.py`  
**Canonical copy**: `~/Documents/ai-tooling/opencode/Gorilla.Opencode/Documentation.Scripts/Documentation.Writing.Scripts/dual_track.py`  
**Vault backup**: `~/Documents/Scripts.For.Work/Scripts.Vault.Do.Not.Delete/doc-tooling/dual_track.py`  
**Symlink on PATH**: `~/.local/bin/dual-track` → canonical copy  
**Current version**: 3283 lines · fixed 2026-08-16 by Bob (IBM) · (header previously said 3225 — that was the 08:06 draft's count)  
**Language**: Python 3 · zero dependencies (stdlib only for core operation)

---

## What This Script Does

`dual_track.py` turns source code, changelogs, diffs, or session work records
into two parallel, complete documents:

- **Developer track** — DITA-structured internal documentation (concept + reference + task
  topics). Written for an engineer who will maintain, audit, or fork the work.
- **Layman track** — IBM plain-language explanation for a non-technical reader who needs to
  understand what was done, what risk they are accepting, and what they can now do.

Neither track is a summary of the other. They cover the same subject in two completely
different languages, structured for two completely different questions.

The script **never makes a network call**. It has two jobs:

1. **`prep`** — reads your source and writes a `.prep.json` containing a system prompt,
   user prompt, and JSON Schema. The model fills this.
2. **`render`** — reads back the filled `.filled.json` and renders polished Markdown
   using deterministic Python code. The model has no control over headings, table format,
   section order, or the claim-sources footer.

Whatever fills the JSON — Claude, GPT, a human, a shell pipeline — is the middle step.
This script does not care how it happens.

---

## Why It Exists

The problem it solves is a specific one: **most documentation tools ask a model to write
Markdown directly**. When you do that, the model controls the structure, and structure
is where quality collapses. Headings appear inconsistently, the layman and developer
sections tell the same story in slightly different words, important gaps are filled with
plausible-sounding inference, and there is no way to tell which claims are facts and which
are guesses.

`dual_track.py` removes that variable. The model fills a typed, schema-validated JSON
object. Python renders the document. The model's prose goes into named fields with specific
purposes. The rendered output is always structured identically because the renderer is
deterministic Python, not an LLM.

---

## The Four Modes

### `code` — document a source file or directory
```bash
dual-track code prep internal/tui/tui.go --output-dir ./docs
# fill the .prep.json files
dual-track code render internal/tui/tui.go --output-dir ./docs
```
Produces: developer track, layman track, and optionally an audit track.
`prep` also runs a rule-based pre-check (deterministic, no model) and embeds
the findings into the audit track before the model sees anything.

### `session` — document what was done in a directory *(added 2026-08-16)*
```bash
dual-track session prep /path/to/folder --since v1.2.3 --narrative notes.txt
# fill the .prep.json files
dual-track session render /path/to/folder
```
This is the mode for "an agent worked in this folder — document what was done."
It assembles four input layers:
- `git diff --stat` (scope — what files changed)
- `git diff` (evidence — the exact changes, capped at 600 lines)
- `git log --stat` (timeline — commit sequence)
- `--narrative FILE` (why — the reasoning that was never committed)

The schema asks for `before_state`, `after_state`, `claimed_but_not_verified`,
`tried_and_abandoned`, and `open_items`. These fields force the model to write
about **work done**, not **current code state**. Those are different things.

### `release` — generate release notes from a changelog or diff
```bash
dual-track release prep --input CHANGELOG.md --output-dir ./release-notes
dual-track release render --output-dir ./release-notes
```
Or auto-generate from git since the last tag:
```bash
dual-track release prep --output-dir ./release-notes
```

### `precheck` — rule-based scan, no model, no network
```bash
dual-track precheck /path/to/folder --output-dir ./docs
```
Exits 1 if any P0 or P1 defect is found. Writes `PRECHECK.md` and `PRECHECK.json`.
Safe to use as a CI gate. Every finding is dual-track: `track_a` plain English with
a physical analogy, `track_b` file/line/mechanism.

---

## The Claim-Sourcing System

Every rendered document ends with a Claim Sources table:

| Claim | Basis | Evidence |
|-------|-------|----------|
| Implementation is ~80% complete | 🤖 model inference | "flags present in argparse but consumers absent" |
| Glob bug was fixed | 📄 stated in input | "args.globs now consumed at every call site" |

The `basis` field uses a strict enum: `stated_in_input` or `model_inference`. The
`score_document()` function reads `basis == "stated_in_input"` literally — a model
spelling it differently scores zero for evidence while looking perfectly sourced to
a human reader. The schema is the enforcement mechanism, not the prompt.

**What claim sources are for**: flagging conclusions and assessments — statements that
required reasoning rather than copying. They are NOT for raw facts (line counts, file
names, quoted strings). A claim_sources list containing only `stated_in_input` entries
is a red flag — every assessment involves inference.

---

## The Validation Gate

`render` validates by default. Pass `--no-validate` to skip.

Validation checks:
- Required DITA topics are present and non-empty
- No banned marketing phrases survived (e.g. "seamlessly", "under the hood")
- No self-congratulatory language survived (e.g. "triumph", "I proved", "neutralizing")
- Concrete numbers are present (digit count > 8)
- Quality score ≥ 85/100 across 6 weighted categories

Validation exits with code 2 if checks fail. The Markdown is written anyway so you
can read what needs fixing. **Do not publish without passing validation.**

The `then_run` hint in every `.prep.json` file includes `--validate`:
```
dual-track code render myfile.py --output-dir ./docs --validate
```

---

## What Was Wrong Before the 2026-08-16 Fix

This section is the honest record of what was broken and why. Preserved here so the
next agent does not have to rediscover it.

### Gap 1 — No "session" mode (structural)

**What was wrong**: The script had `code` (document the end state of a file) and
`release` (document a changelog). Neither answered the question: "an agent worked in
this folder for four hours — write the documentation of what was done."

`code` mode reads the current file. It documents the *state*. It cannot document the
*work* — why an approach was chosen, what was tried and failed, what was claimed done
but never tested, what remains. That information lives in git history and session notes,
not in the source file.

**What was added**: The `session` mode. It assembles `git diff`, `git log --stat`, and
an optional narrative file into a single prompt. The schema is structured around work
facts (`before_state`, `after_state`, `tried_and_abandoned`, `claimed_but_not_verified`,
`open_items`) rather than code facts (`flags`, `api_surface`, `kill_switches`).

---

### Gap 2 — `build_context_block()` gathered metadata, not causality (structural)

**What was wrong**: The auto-context function collected git tags, branch name, file
language, line count, and imports. All of this is metadata about the *current state*.
None of it answered: what did prior agents claim? What contradictions exist? What
needs to be confirmed or corrected?

When prior agents had written `STATUS_REPORT.md`, `IMPLEMENTATION_PROGRESS.md`, and
`REVISED_GAP_ANALYSIS.md` — all contradicting each other — none of that context
reached the model. The model would read the source file and produce a document that
silently repeated or contradicted the prior claims, with no awareness that prior claims
existed.

**What was added**: `get_prior_doc_context()` scans the target directory for
`STATUS*.md`, `PROGRESS*.md`, `GAP*.md`, `AUDIT*.md`, `IMPLEMENTATION*.md`, etc.
and injects the first 40 lines of the three most recent ones into every prompt with
explicit instructions: *"Your document must confirm, correct, or supersede these
claims. Do NOT repeat a prior claim without verifying it against the source material."*

---

### Gap 3 — The separation score only checked syntax, not ideas (structural)

**What was wrong**: The quality score's "separation" check counted occurrences of
`func()`, `::`, `struct`, and `malloc` in the layman track. If the count exceeded 12,
it deducted 8 points. This only caught surface-level jargon. It did not catch the real
failure mode: a layman track that tells exactly the same story as the developer track
in slightly plainer words.

The ERR-UI-004 document is the proof: Section A had no `func()` calls, scored fine,
and was still just the developer narrative rephrased. The two tracks had collapsed at
the *idea* level, not the *syntax* level.

**What was added**: A second check that scans the layman `big_picture` field for
developer vocabulary density — function, method, class, parameter, callback, async,
thread, compile, runtime, exception, etc. If more than 6 such terms appear in the
plain-language description, the separation score is penalised. Both checks now cap at
6 points each (replacing the single 8-point jargon penalty), so neither alone can
wipe out the full category score.

---

### Gap 4 — The banned-phrase list only caught marketing language (structural)

**What was wrong**: `BANNED_PHRASES` contained press-release language: "seamlessly
integrates", "robust solution", "best-in-class". These catch a model acting as a
marketing copywriter. They do not catch a model acting as a self-congratulatory narrator.

The ERR-UI-004 layman track contained: *"This is a triumph of methodical debugging!"*,
*"I immediately deployed the pristine version from our Kernel Safety Vault, neutralizing
the rogue code"*, *"proves the power of relentless root-cause analysis"*. None of these
triggered the banned-phrase check. All of them are a failure of documentation discipline
— the document is about the work, not the agent who did it.

**What was added**: `BANNED_SELF_CONGRAT` — a second set of 18 patterns covering
superlatives applied to the agent's own work (`masterful`, `perfectly`, `flawlessly`),
dramatic framing (`triumph`, `neutralizing`, `rogue code`, `heroic`, `landmark`), and
first-person pride (`I proved`, `I immediately`, `I demonstrated`). Validation now runs
two separate checks and reports them separately so you know which failure you have.

---

### Gap 5 — `--validate` was opt-in when it should be the default (structural)

**What was wrong**: `code render` and `release render` had `--validate` as an optional
flag. The natural workflow — `dual-track code render file.py --output-dir ./docs` —
skipped validation entirely. The `then_run` hint written into every `.prep.json` also
omitted `--validate`, so even if you copy-pasted the exact next command the tool gave
you, you skipped the gate.

The August 15 documentation session is the proof: the pfind docs were never rendered
at all, but even in sessions where they were rendered, nobody noticed that the quality
gate was optional and being skipped.

**What was changed**: `code render`, `release render`, and `session render` now
validate by default. Pass `--no-validate` to skip. The `then_run` hint in every prep
file now always includes `--validate`.

---

### Gap 6 — `worst_case` prompt invited hedging (prompt)

**What was wrong**: The layman schema asked for: *"The realistic worst outcome —
financial, privacy, or otherwise — stated plainly. Not catastrophized, not minimized."*

This is too abstract. A model left to fill this writes "the script could return
incorrect results" — technically true of everything and tells the reader nothing.

**What was changed**: *"The most harmful outcome that is plausible given what this code
actually does. State it in terms of what the user would experience, with a concrete
example. Do not write 'incorrect results' as a catch-all — name the specific result
that would be incorrect and what it would cost the user."*

---

### Gap 7 — `design_rationale` invited invention (prompt)

**What was wrong**: The developer schema asked: *"Why this design was chosen over
alternatives."* A model cannot answer this from source code alone — the reasons a
design was chosen are in commit messages, comments, and session notes, not in the
implementation. With no source for the answer, models produce plausible-sounding
rationale that is pure inference, presented without any marker indicating it is inference.

**What was changed**: The field was renamed `known_alternatives` and the prompt now
says: *"Quote source comments explaining why. If none are documented, write 'Not
available in the source material.' Do not infer rationale from the code structure."*
This makes the absence of information visible instead of papering it over.

---

### Gap 8 — `_CLAIM_SOURCING` was backwards (prompt)

**What was wrong**: The instruction said *"add an entry for every concrete fact"*.
This caused models to fill the claim_sources table with things like:
- `{"claim": "file is 1969 lines", "basis": "stated_in_input", "evidence": "1969 lines"}`
- `{"claim": "tool uses rrf_fuse()", "basis": "stated_in_input", "evidence": "rrf_fuse()"}`

These are not the claims that need flagging. Quoting a line count is not an inference.
The claims that need flagging are conclusions: *"this implementation is 80% complete"*,
*"the glob bug was the cause of silent failures"*. Those require reasoning. A
claim_sources list full of quoted line counts gave a false impression of rigour while
leaving the actual inferences completely unmarked.

**What was changed**: *"Add an entry for every CONCLUSION or ASSESSMENT — any statement
that required reasoning rather than copying. Do NOT add entries for raw facts you quoted
directly. A claim_sources list containing only 'stated_in_input' entries is a red flag —
every assessment involves inference."*

---

## Operational Copies on This Machine

All three copies are identical as of 2026-08-16 (re-verified by `sha256sum` after the
vault copy was found holding the 08:06 draft and re-synced the same evening):

| Copy | Path | Role |
|------|------|------|
| Primary / symlink target | `~/Documents/ai-tooling/opencode/Gorilla.Opencode/Documentation.Scripts/Documentation.Writing.Scripts/dual_track.py` | Working copy; `~/.local/bin/dual-track` points here |
| Vault backup | `~/Documents/Scripts.For.Work/Scripts.Vault.Do.Not.Delete/doc-tooling/dual_track.py` | Recovery copy |
| Second Brain | `~/Documents/SECOND.BRAIN/Scripts/documentation/dual-track/dual_track.py` | This folder |

When updating the script, update all three. The symlink does not need to change —
it already points to the canonical copy.

---

## Quick Reference

```bash
# Document a source file
dual-track code prep myfile.py --output-dir ./docs --format both
# fill myfile_developer.prep.json and myfile_layman.prep.json
dual-track code render myfile.py --output-dir ./docs

# Document a directory of related files as one unit
dual-track code prep ./patches/01_MEDIA/ --output-dir ./docs

# Document what an agent did in a folder (session mode)
dual-track session prep /path/to/folder --since v1.2.3
dual-track session prep /path/to/folder --since HEAD~5 --narrative session_notes.txt
dual-track session render /path/to/folder

# Generate release notes from git log
dual-track release prep --output-dir ./release-notes
dual-track release render --output-dir ./release-notes

# Rule-based scan only (no model, no network, safe as CI gate)
dual-track precheck /path/to/folder

# Generate paste-ready prompts for use in any browser chat (zero-quota path)
dual-track code prep myfile.py --output-dir ./docs --payload

# Skip validation (rarely right)
dual-track code render myfile.py --output-dir ./docs --no-validate
```

---

## Project-Specific Rules

Place a `.dual-track-rules.py` file in any project root. It must define:
```python
def rules(infos, collect):
    for i in infos:
        # i has: name, ref, path, language, lines, code_lines, complexity, sha256, content
        if some_condition(i['content']):
            collect.add('P1',
                'Plain English explanation with physical analogy.',
                f"{i['ref']}: technical detail, file and line.",
                'What to do to fix it.')
```
The file is auto-discovered by walking up from the target until a `.git` directory is
found. It is loaded only if it exists; most projects need only the builtin rules.

---

*Last updated: 2026-08-16 · Fixed by Bob (IBM) · See gap analysis above for full rationale*
