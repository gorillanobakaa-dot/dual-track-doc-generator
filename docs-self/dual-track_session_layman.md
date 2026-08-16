# The documentation tool got fixed, tested on itself, and put on GitHub — Plain Language

> Session record generated 2026-08-16

---

## What happened

There is a tool on this machine whose whole job is writing honest documentation. It takes a piece of work and produces two explanations of it: one for engineers, one for everyone else. Earlier this year it produced some embarrassing output — a report that bragged about itself as a great victory of debugging, and a plain-language section that was the engineer section with a thesaurus run over it. Its quality checks never noticed, because they were only looking for advertising language and code words, not for bragging or for two sections telling the same story.

Today the tool was repaired. Eight separate weaknesses were closed. It now refuses bragging, notices when the two explanations collapse into one, checks quality every time by default instead of only when asked, and gained a whole new ability: describing what was done in a work session — including what failed and what is unproven — instead of only describing finished code.

After the repair, the tool's home folder was published to GitHub so it has a public copy, and then the tool was pointed at itself: the page you are reading was produced by the repaired tool describing its own repair. That self-test is the point — if you are reading this, the new mode produced output that passed its own quality gate.

## Honest state of play

Done: the repaired tool exists, is 3283 lines, and sits in three places on this machine plus GitHub. Done: an automatic rule scan found no serious defects, only three 'finish this later' notes left in the code. Done as of this page: the new session mode has been run start to finish once — on itself. Half-done: an earlier attempt at self-description produced two nearly empty forms that were never finished or checked; they are still in the folder as evidence. Checked and found wrong: the three local copies are NOT identical — the safety backup in the vault still holds this morning's earlier draft, not the final repaired version.

## Worst case if something is wrong

The specific danger this tool guards against is a false 'all done' report. If one of today's repairs is itself broken — say the bragging filter has a typo in a pattern — then a future report could carry self-praise or an unmarked guess straight past the gate, and you would trust a document that reads as verified but is not. Concretely: a report might say 'the kernel patch is applied and tested' when the model only inferred it, and you would ship on that. The mitigation that exists today is the claim-sources table at the bottom of every report, which marks each conclusion as either quoted from the input or guessed by the model.

## What changed for you

**Bragging in reports**
- Before: The tool would let through sentences of self-praise — the writer calling its own work a great victory, or narrating how fast and decisive its own fix was.
- After:  Eighteen patterns of self-praise and drama now fail the quality check.
- Affects: everyone who reads its reports

**Quality checking**
- Before: Checking was optional, and the tool's own suggested commands skipped it.
- After:  Every report is checked unless someone deliberately turns checking off.
- Affects: everyone

**Describing work sessions**
- Before: The tool could only describe finished code or a list of changes, never 'here is what was actually done today, including the failures'.
- After:  A new session mode reads the change history plus the worker's own notes and writes the story of the work, with required sections for unproven claims and abandoned attempts.
- Affects: anyone who asks an assistant to work unattended and wants an honest record

**Made-up explanations**
- Before: When asked why a design was chosen, the tool's forms invited a plausible guess.
- After:  The forms now demand either a quoted source or the exact sentence 'Not available in the source material.'
- Affects: everyone

## What you can do now

- Ask any assistant working in a folder to run the session mode afterwards, and get an honest record of the session — including what was tried and dropped, and what is claimed but unproven.
- Trust that a report which passed the gate contains no self-praise, no advertising language, and a table telling you which statements are quotes and which are guesses.
- Fetch the tool from GitHub at github.com/gorillanobakaa-dot/dual-track-doc-generator on any machine.
- Run the rule-based scan on any folder with no internet and no AI involved at all.

## What is still missing

- **A finished self-description of the tool's own code (the earlier attempt produced near-empty forms).** — Someone who wants a full reference for the tool's internals must read the code or the README; the polished internal reference does not exist yet.
- **An up-to-date vault backup. The comparison ran and the vault copy is this morning's earlier draft.** — If you ever restore from the vault, you get a version missing the last round of today's repairs — it needs a re-sync, which is the owner's call.
- **Proof of the repairs on a second, real job (this self-test is only one).** — Confidence in the new checks rests on a single run until the tool documents another session.

## How to check that what we say is done actually works

**Step 1:**
```bash
Open github.com/gorillanobakaa-dot/dual-track-doc-generator in a browser.
```
  - **Pass:** The repository exists and contains dual_track.py, README.md, and a docs-self folder including this document.

**Step 2:**
```bash
Ask an assistant to run the tool's checker on this very document (session render with checking on).
```
  - **Pass:** It reports a score of at least 85 out of 100 and no banned-language findings.

**Step 3:**
```bash
Ask an assistant to put an obvious sentence of self-praise (the writer calling its own debugging a great victory) into a test report and run the checker.
```
  - **Pass:** The checker fails the report and names the self-praise it found. If it passes, today's repair did not work.


## Should you be concerned?

Mildly, in two places. First, the repairs have been proven on exactly one run — this one — and a single passing test is thin evidence; treat the next few reports from this tool with normal suspicion. Second, the folder on GitHub contains the two abandoned near-empty forms, which could confuse a stranger into thinking they are the tool's real self-documentation; they are labelled here as abandoned precisely so that does not happen. Nothing in today's work touches your data, your network, or any running system — the tool reads files and writes Markdown, and that is all.

## Glossary

**Validation gate** — An automatic quality check that fails a document for bragging, advertising language, missing sections, or a low score.

**Session mode** — The new ability to describe a day's work in a folder — what changed, why, what failed, what is unproven.

**Claim sources** — The table at the bottom of every report saying, for each conclusion, whether it was quoted from the input or guessed.

**Precheck** — A rule-only scan that finds defects without any AI involved.

## Claim Sources

| Claim | Basis | Evidence |
|-------|-------|----------|
| The self-test passing is meaningful evidence the repairs work | 🤖 model inference | *(none — model judgment)* |
| The earlier self-description attempt was abandoned in an unusable state | 📄 stated in input | thin stubs: most arrays empty ... never brought up to standard or rendered |
| The tool never touches the network | 📄 stated in input | The script never makes a network call |
| One passing run is thin evidence and the next reports deserve normal suspicion | 🤖 model inference | *(none — model judgment)* |


---
**How to verify this document:**
`📄 stated in input` — the model's phrasing of something your source text said.
Find the matching line in the original to verify.
`🤖 model inference` — the model's own judgment or synthesis. Treat as opinion,
not measurement. Re-run on the same input and check whether specific numbers
stay consistent between runs.

*Session record. Plain-language track. Its developer twin covers the same session in technical detail.*