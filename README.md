# Chronicle

A work ledger for agents and humans: capture observed events, recover recorded file
versions, and preserve the intent that a trace cannot infer.

Includes a Python CLI, an installable Agent Skill, optional capture hooks, and a local web
canvas for exploring evidence. The capture core and CLI use the Python standard library.

## Install

Python 3.9 or later on macOS or Linux:

```bash
pipx install 'git+https://github.com/AntreasAntoniou/chronicle.git'
npx skills add AntreasAntoniou/chronicle
chron --help
```

Or clone this repository and run `pip install -e '.[dev,canvas]'` in a virtual environment.
This release comes from GitHub, not the unrelated PyPI package of the same name.

## Record intent

```bash
chron resume
chron open "continue search" --state "empty results are intentional staging"
chron decision "keep the index" --why "measured query latency is sufficient"
chron arm "replace the staging index" --class R1 --restore "verified snapshot and restore command"
chron landed "search deployed" --ext "url=https://search.example.com rollback=previous-image"
chron close "stopped after verification" --not-done "production rollout remains pending"
```

Use measured evidence. Correct earlier claims by appending `chron correct`, preserving the
original. Optional narration always carries an `inferred` label and cannot create an ARM
or CLOSE.

## Enable capture

Capture covers only installed, active, verified integrations. It cannot reconstruct a
version that was never captured. Excluded, oversized, or unreadable files can leave gaps.

```bash
chron install-hooks --dry-run
chron install-hooks --shell --git
chron doctor
```

The local installer adds Claude Code hooks by default. `--shell` additionally installs a
zsh hook; `--git` installs hooks in registered repositories. Configuration is merged with
backups. Make a harmless edit and inspect its event before relying on coverage. Shell
capture requires the shell hook to be sourced.

Codex hooks are experimental and version-dependent. Read
[the integration notes](references/codex.md). Chronicle never grants or renews hook trust.
Configuration alone does not prove that a hook executed.

## Read and recover

```bash
chron history src/app.py
chron show src/app.py --at 2h
chron restore src/app.py --at 2h --to /tmp/recovered-app.py
chron search "search index"
chron doctor
```

Install the `canvas` extra and run `chron canvas` for the optional web interface. Keep its
default loopback binding: it exposes private history and has no authentication layer.

## Storage and synchronization

Events live in append-only JSONL lanes under `.chronicle/` or `~/.chronicle/lanes/`.
Captured content lives in `~/.chronicle/cas/`; curated narrative lives in `CHRONICLE.md`.
Keep all of these private.

New captures use gzip across supported Python versions. Legacy Zstandard-only blobs
require Python 3.14 or later to read; a gzip copy is preferred when both exist.

`chron sync` stays local by default. Set `CHRONICLE_REMOTES` to a space-separated list of
SSH hosts for remote pulls. `CHRONICLE_SPINE` or `--spine` selects the destination. Blobs
are encrypted with `age` on the way to the spine, or skipped when recipients are missing.
Configure public recipients through `CHRONICLE_RECIPIENTS` or
`~/.chronicle/age_recipients.txt`. Event metadata remains plaintext and can be sensitive;
the spine must remain private even when blobs are encrypted.

Redaction cannot guarantee detection of every embedded secret. Read
[SECURITY.md](SECURITY.md) before enabling capture.

## Optional narration

Narration sends selected trace and transcript content through your Claude CLI and can
incur costs. Review `chron narrate --dry-run`, then set `CHRONICLE_NARRATOR_MODEL` to a model
available to your account. `CHRONICLE_NARRATOR_BUDGET` bounds prompt bytes, not spend.
Butler checks a project budget when installed; absent Butler is not a spending cap.
Capture, the CLI, and the canvas require no model calls.

## Development

```bash
pip install -e '.[dev]'
pytest -q
python src/chronicle/capture.py selftest
```

Tests cover concurrent append, interrupted writes, excluded content, command
classification, additive installation, and inferred-entry boundaries. The hook gate is a
coordination aid; enforcement depends on the host invoking and honoring it.

MIT licensed. See [CONTRIBUTING.md](CONTRIBUTING.md).
