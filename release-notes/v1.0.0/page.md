# dual-track-doc-generator v1.0.0

## What this is

One Python file, `dual_track.py`, that writes two explanations of the same piece of work: one for the engineer who
will maintain it, one in plain language for everyone else. It reads source code, a changelog, a diff or a work
session, prepares the request for a model, and checks the result before it is accepted: no advertising words, no
self-praise, the two explanations really different, every claim marked as quoted or guessed. It needs only Python 3
and makes no network call unless you pass `--call-model`.

## Should you download it?

Yes, if you write documentation for work done by you or by an AI assistant and want it honest and readable by people
who do not code. Yes, if you used an earlier copy on Windows: session records came out empty there, and this release
fixes that. No, if you want finished documentation without running a model: this tool prepares and checks, a model
writes.

## Why this matters

A report that says "done and tested" when nothing was tested is worse than no report. This tool exists to stop that,
and the release makes it safer to run: a value you pass can no longer be run as a command on your machine. It is
free, and stays free.
---
# In plain language: everything in this release

This is the complete explanation, not a summary of one. Nothing below is behind a link.

<!-- plain-language track: in full, on this page -->

### What changed

- **Safer to run.** Some of the values you type (a date, a file name) used to be handed to the computer's command
  line as text. A value with a quote or a semicolon in it could run something you did not intend. Now no value ever
  goes through the command line.
- **Works on Windows.** On Windows the history of a work session came out empty, so the record of what was done was
  missing its main evidence. It is filled in now.
- **Describes the right period.** Asked to describe the work since a given point, the tool listed the history from
  before that point. It now lists the work since, matching the changes it shows beside it.
- **Its own suggested commands work.** The tool prints the next command to run; that command failed with an error.
  It runs now.
- **No hidden rules.** A project can carry its own extra rules for the tool. When one is found, the tool now says so
  before using it, and you can switch it off.
- **The description matches the program.** The README now says what the program does: it goes online only when you
  ask it to.

### How we know

Nine tests in Gorilla Fieldkit run the tool. Five of them failed on the previous version; all nine pass on this one.

### What it cannot do yet

It does not write the documentation itself without a model, and its checks catch the kinds of mistakes they were
built for, not every mistake.

---

# For developers: how it works, and how to check it

Written for someone who will audit, fork or change the code. It covers the same release as the plain-language part above; neither is a summary of the other.

<!-- developer track: in full, on this page -->

### Changes (09f6bcd, merged as PR #1)

- Every git call goes through `_git_out(args: list)`; nothing uses `shell=True`. Before, `--since` was interpolated
  into shell strings and `cleanup` quoted file names in single quotes, so a value containing `'` or `;` reached the
  shell. `--since` is resolved first with `git rev-parse --verify`, so a value starting with `-` cannot be read as a
  git option.
- Windows: `cmd.exe` understood neither the single quotes nor `| head`, so `session` diffs and git context were
  empty. The list form fixes both.
- `session prep` logs `{since}..HEAD`. It ran `git log {since}`, the history before the point, while the diff beside
  it covered the history after.
- `code render`, `session render` and `release render` accept `--validate` (the default). `prep` prints render
  commands with it and the docstring tells you to pass it; they failed with "unrecognized arguments".
- A `.dual-track-rules.py` found by walking up the tree is named on stderr before it runs; `--no-rules` (precheck,
  code prep) skips it.
- README: no network call unless `--call-model`.
- Removed in this release: `__pycache__/dual_track.cpython-313.pyc`, committed by mistake in 09f6bcd; `__pycache__/`
  is now ignored.

### Tests

`tests/test_dual_track.py` in Gorilla.Fieldkit: 9 tests, 5 fail on the previous version, all 9 pass. The release
workflow (`.github/workflows/release.yml`) compiles the script and runs `--help` on a clean Linux runner before it
publishes, and attaches `dual_track.py` to the release.

### Known limits

The tests live in Gorilla.Fieldkit, not in this repository. `--help` still says prep/render never makes a network
call; `--call-model` does, as the README says.
