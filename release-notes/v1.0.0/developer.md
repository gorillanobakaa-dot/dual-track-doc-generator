# dual-track-doc-generator v1.0.0, for developers

## Changes (09f6bcd, merged as PR #1)

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

## Tests

`tests/test_dual_track.py` in Gorilla.Fieldkit: 9 tests, 5 fail on the previous version, all 9 pass. The release
workflow (`.github/workflows/release.yml`) compiles the script and runs `--help` on a clean Linux runner before it
publishes, and attaches `dual_track.py` to the release.

## Known limits

The tests live in Gorilla.Fieldkit, not in this repository. `--help` still says prep/render never makes a network
call; `--call-model` does, as the README says.
