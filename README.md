# toan-single-cell-analysis-skill

A skill that establishes the framework for every new single-cell analysis project so that any agent (or human) can pick up the current state of the work cold.

## What it enforces

- **uv project mode, Python 3.12 floor** — `uv init --python 3.12`, `uv add`, `uv run`; never `pip install`, never conda
- **Raw data is read-only** — never renamed, moved, re-saved, or "cleaned up" unless the user explicitly asks
- **`docs/data.md`** — inventory and state of every data file: format, shape, provenance, quirks, checksums
- **`docs/journal.md`** — dated log of what was done, decisions, outputs, next steps

## Install

```sh
npx skills add btvn-toan-nt/toan-single-cell-analysis-skill
```

[![skills.sh](https://skills.sh/b/btvn-toan-nt/toan-single-cell-analysis-skill)](https://skills.sh/btvn-toan-nt/toan-single-cell-analysis-skill)
