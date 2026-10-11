# dual-track-doc-generator v1.0.0, in plain language

## What changed

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

## How we know

Nine tests in Gorilla Fieldkit run the tool. Five of them failed on the previous version; all nine pass on this one.

## What it cannot do yet

It does not write the documentation itself without a model, and its checks catch the kinds of mistakes they were
built for, not every mistake.
