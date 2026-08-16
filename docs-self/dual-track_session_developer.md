# dual_track.py 2026-08-16: eight-gap overhaul, session mode, and first publication

> Session record generated 2026-08-16

---

## Problem Being Solved

The generator's output on prior jobs exposed eight failure modes. The ERR-UI-004 document passed the old checks while containing self-congratulatory narration and a layman track that repeated the developer narrative in plainer words. The 2026-08-15 pfind session skipped rendering and validation entirely because `--validate` was opt-in. The tool also had no way to document work done in a directory — `code` mode reads current state, and work history lives in git and session notes, not in the source file.

## Approach Taken

Close the gaps inside `dual_track.py` itself instead of relying on prompt discipline. Three structural additions (a `session` mode, `get_prior_doc_context()` injection, an idea-level separation check), one new ban list (`BANNED_SELF_CONGRAT`), one default flip (validation on unless `--no-validate`), and three prompt rewrites (`worst_case`, `known_alternatives`, `_CLAIM_SOURCING`). Then publish the folder as a git repository and run the tool on itself as its first end-to-end test.

## Before

Before this session the script had two documentation modes (`code`, `release`), a marketing-only banned-phrase list, a syntax-only separation check, opt-in validation, and prompts that invited invented rationale and over-broad worst cases. The folder was not a git repository and existed only as one of three local copies.

## After

The script is 3283 lines with four modes (`code`, `session`, `release`, `precheck`) and validation on by default. The folder is a git repository with one commit (`b0b880d`) pushed to `github.com/gorillanobakaa-dot/dual-track-doc-generator`. The self-documentation in `docs-self/` is in a mixed state: the offline precheck ran (one P2 finding), the `code`-mode prep files exist, but their `.filled.json` files are thin stubs that no one rendered or validated. This session-mode run is the first fill that reaches the render-and-validate stage.

## Known Alternatives

Not available in the source material. The commit message (`Add dual-track documentation generator source and self-docs`) records no rejected approaches, and the session notes name none.

## Files Changed

| File | Change | What Changed | Why |
|------|--------|--------------|-----|
| `dual_track.py` | added | The full 3283-line generator enters version control, carrying all eight fixes made earlier today. | The overhaul described in `README.md` happened before the repository existed, so the initial commit is also the record of the fixed script. |
| `README.md` | added | 363 lines: usage for all four modes, the claim-sourcing system, the validation gate, and a gap-by-gap record of what was wrong before the fix. | The gap analysis is preserved so the next agent does not rediscover the eight failure modes. |
| `docs-self/PRECHECK.md` | added | Offline rule-based scan result: zero P0, zero P1, one P2 (three TODO/FIXME markers in the source). | The precheck is the only part of the self-documentation that is not a model's opinion. |
| `docs-self/PRECHECK.json` | added | The same precheck finding in machine-readable form. | Same run as `PRECHECK.md`. |
| `docs-self/dual_track_developer.prep.json` | added | `code`-mode prep envelope for the developer track of the script's own documentation. | Self-documentation attempt started at 10:44, before the repository was created. |
| `docs-self/dual_track_layman.prep.json` | added | `code`-mode prep envelope for the layman track. | Same self-documentation attempt. |
| `docs-self/PAYLOAD.dual_track.developer.txt` | added | Paste-ready prompt (3441 lines) for filling the developer track in any chat window. | The `--payload` path exists for zero-quota situations. |
| `docs-self/PAYLOAD.dual_track.layman.txt` | added | Paste-ready prompt (3470 lines) for the layman track. | Same `--payload` run. |

## Decisions Made

- 📄 **Enforce quality in code, not in prompts: ban lists, score checks, and schema enums live in `dual_track.py` where a model cannot talk its way around them.** — The README states the schema is 'the enforcement mechanism, not the prompt'.
- 📄 **Flip validation from opt-in to opt-out on every render command.** — The 2026-08-15 session proved that even the tool's own `then_run` hint let users skip the gate.
- 📄 **Rename `design_rationale` to `known_alternatives` and require 'Not available in the source material.' when rationale is undocumented.** — A model cannot recover design intent from an implementation, so the field invited invention.
- 🤖 **Publish the folder to GitHub as its own repository instead of leaving it as a local copy.** — The session notes record the publication but not the motive; a public home for the tool is the plausible reason.
- 📄 **Leave the thin `code`-mode filled stubs in place instead of deleting them.** — The session notes cite the house rule that nothing is deleted; the stubs stay as evidence of the attempt.

## Tried and Abandoned

- **Filling the `code`-mode prep files (the 16:42 `.filled.json` stubs).** — The fill was minimal — most reference arrays empty, one claim source each — and no one ran render or validation on it. The session notes judge that it would not pass the script's own gate. Abandoned instead of completed; the session-mode run supersedes it as the self-documentation path.

## ⚠ Claimed But Not Verified

*Prior documents claimed these are done. No test evidence found in this diff:*

- The README states all three local copies (canonical, vault, Second Brain) are identical as of 2026-08-16. A `sha256sum` check during documentation DISPROVED this: the canonical and Second Brain copies match (3283 lines), but the vault copy is the 08:06 snapshot (3225 lines) and never received the final 08:57 version. The README's line-count header (3225) is also stale for the same reason.
- The eight fixes are described in the README but have no end-to-end test evidence in the diff; this session-mode run is the first such test.
- The `~/.local/bin/dual-track` symlink is stated to point at the canonical copy; the session records do not re-verify it.

## Open Items

| Item | Priority | Blocks |
|------|----------|--------|
| Complete or supersede the `code`-mode self-documentation: the two `.filled.json` stubs were never brought to standard or rendered. | medium | nothing currently — the session-mode documentation covers the publication |
| Resolve or track the three TODO/FIXME markers that the precheck flags as its single P2 finding. | low | nothing currently |
| Re-sync the vault copy of `dual_track.py`: the `sha256sum` check shows it holds the 08:06 pre-final snapshot (3225 lines), not the 08:57 final (3283 lines). The owner decides whether to overwrite the vault snapshot or keep both. | medium | recovery from the vault would restore a version missing the final 58 lines of today's fixes |

## How to verify this work is correct

**Step 1:**
```bash
python3 dual_track.py precheck . --output-dir ./docs-self
```
  - **Pass:** Exit code 0; `PRECHECK.md` reports zero P0 and zero P1 findings.
  - **Fail:** Exit code 1, or a P0/P1 row in the findings table.

**Step 2:**
```bash
python3 dual_track.py session render /home/gorilla/Documents/SECOND.BRAIN/Scripts/documentation/dual-track --output-dir ./docs-self --validate
```
  - **Pass:** Two Markdown files appear (`dual-track_session_developer.md`, `dual-track_session_layman.md`) and both tracks score at least 85/100.
  - **Fail:** Exit code 2 with named failing checks, or a score below the 85 threshold.

**Step 3:**
```bash
git -C /home/gorilla/Documents/SECOND.BRAIN/Scripts/documentation/dual-track log --oneline origin/master
```
  - **Pass:** The pushed history contains commit `b0b880d` and the follow-up commit with the session documentation.
  - **Fail:** `origin/master` missing the documentation commit, meaning the push never happened.

**Step 4:**
```bash
sha256sum ~/Documents/ai-tooling/opencode/Gorilla.Opencode/Documentation.Scripts/Documentation.Writing.Scripts/dual_track.py ~/Documents/Scripts.For.Work/Scripts.Vault.Do.Not.Delete/doc-tooling/dual_track.py ~/Documents/SECOND.BRAIN/Scripts/documentation/dual-track/dual_track.py
```
  - **Pass:** Three identical hashes, confirming the README's identical-copies claim.
  - **Fail:** Any hash differs — one copy missed the overhaul.


## Technical Debt

🟡 **LOW** — Three TODO/FIXME markers remain in `dual_track.py` (precheck finding P2-001). → Resolve each marker or convert it into a tracked item outside the source.
🟠 **MEDIUM** — The `code`-mode filled stubs in `docs-self/` present themselves as completed fills but would fail validation. → Refill them properly and render with `--validate`, or record in the repository that the session-mode documents supersede them.
🟡 **LOW** — Passing `.` as the session target yields an empty display name and the fallback slug `session`, because the target's basename is empty. → Resolve the target path before deriving the slug in `_build_session_input()` and `run_session_render()`.

## Claim Sources

| Claim | Basis | Evidence |
|-------|-------|----------|
| The eight-gap overhaul preceded the first commit, so the diff cannot show before/after states of the fixes | 🤖 model inference | *(none — model judgment)* |
| The 16:42 filled stubs would fail the validation gate | 📄 stated in input | they would not pass the script's own validation gate and were never rendered |
| This session-mode run is the tool's first end-to-end test after the overhaul | 📄 stated in input | this session-mode run is itself the first end-to-end test of session prep, model fill, and validated render |
| The publication motive (a public home for the tool) is not recorded anywhere in the inputs | 🤖 model inference | *(none — model judgment)* |
| The empty-slug behaviour with `.` as target is a defect instead of intended | 🤖 model inference | *(none — model judgment)* |


---
**How to verify this document:**
`📄 stated in input` — the model's phrasing of something your source text said.
Find the matching line in the original to verify.
`🤖 model inference` — the model's own judgment or synthesis. Treat as opinion,
not measurement. Re-run on the same input and check whether specific numbers
stay consistent between runs.

*Session record. Developer track. Covers work done, not current code state.*