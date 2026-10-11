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
