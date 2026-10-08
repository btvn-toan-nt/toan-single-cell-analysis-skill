---
name: toan-single-cell-analysis-skill
description: "Use when starting a new single-cell or omics analysis project (scRNA-seq, scATAC, CITE-seq, spatial), resuming an existing analysis (read its docs/ first), or when a script, notebook, or agent is about to touch raw data files (h5ad, h5, matrix.mtx, csv/tsv metadata). Keywords: scanpy, anndata, uv, Python 3.12 environment setup, raw data, QC."
---

# Single-Cell Analysis Framework

## Overview

Every analysis project follows one framework: a uv-managed Python 3.12 project, raw data treated as strictly read-only, and two handoff files — `docs/data.md` (what the data is) and `docs/journal.md` (what has been done) — so any agent can pick up the work cold. Derived outputs are always new files; raw inputs are never mutated.

## When to Use

- Starting a new single-cell / omics analysis environment (scRNA-seq, scATAC, CITE-seq, spatial...)
- Resuming any analysis project — read `docs/data.md` and `docs/journal.md` FIRST; if they don't exist, create them from what you can establish before doing anything else
- Any moment you are about to read, copy, move, re-save, or "clean up" data files
- NOT for: trivial one-off scripts with no data, non-analysis code, or when the user explicitly overrides a rule (note the override in the journal)

## Bootstrap: New Analysis Project

1. `uv init --python 3.12` in the project dir; verify `pyproject.toml` has `requires-python = ">=3.12"` and `.python-version` says `3.12` (`uv python pin 3.12` if not)
2. Add dependencies with `uv add` only as needed (typical start: `uv add scanpy anndata leidenalg python-igraph matplotlib`); commit `uv.lock`
3. Reference raw data IN PLACE — record absolute paths in `docs/data.md`, never copy or duplicate data into the project
4. Create `docs/data.md` and `docs/journal.md` (templates below); if the project has or gets an `AGENTS.md`, put pointers to both files at the top of it
5. Inventory the data → fill `docs/data.md`
6. Log the bootstrap (env, layout, initial data state, open questions) as the first `docs/journal.md` entry

## Rules

### 1. Environment: uv project mode, Python 3.12 floor

- Only `uv init` / `uv add` / `uv run` / `uv lock` / `uv python pin`. Never bare `pip install`, never `/venv/bin/pip`, never conda/mamba, no hand-rolled venvs
- `requires-python = ">=3.12"` (floor, not exact pin); `.python-version` = `3.12` so uv resolves a 3.12 interpreter; `uv.lock` provides reproducibility
- New dependencies go through `uv add` (updates `pyproject.toml` + lock together) — never an unpinned pip line buried in a README
- Inheriting an existing project: keep its pyproject/lock unless broken; raise the floor to 3.12 only when you can verify the deps support it

### 2. Data: strictly read-only

Never modify raw data files: no rename, move, delete, overwrite, re-save, re-format, re-compress, deduplicate, or in-place "fix". The bytes on disk are immutable.

- Messy filenames (spaces, `(1)`, ALL CAPS)? Live with them — quote paths in code; never "tidy" them
- Outputs and derived files (filtered h5ad, figures, tables) go to `results/` under new names — never back into the data directory
- Don't copy raw data into the project — reference by absolute path in `docs/data.md`
- Data looks corrupt, misnamed, or duplicated? Record it in `docs/data.md` (quirks) and ask the user — do not fix

| Excuse | Reality |
|--------|---------|
| "A rename/move doesn't modify contents" | It modifies the data on disk. Paths appear in scripts and manifests; renames break reproducibility |
| "Re-saving fixes the corruption" | No. Note the corruption and ask; the raw file may need re-delivery |
| "I copied it, so modifying the copy is fine" | The copy IS misplaced (rule: don't duplicate). Copying is not a license to edit |
| "Just compressing/reformatting to tidy up" | That rewrites the file byte-for-byte differently. No |
| "The user will obviously want this cleanup" | Not explicit = not allowed |

Modification is permitted ONLY when the user's own words explicitly ask for it, and then: say exactly what changed in `docs/journal.md`.

### 3. Documentation: data state + progress

`docs/data.md` — one section per data item; blank fields filled as learned:

```markdown
# Data

## <file name>
- Path: <absolute path> (read-only)
- Format: <10x h5 / h5ad / mtx+tsv / csv> | Size/checksum: <size, sha256>
- System: <cells x genes, samples, modality, species, chemistry>
- Provenance: <where it came from, when received>
- Quirks: <odd names, suspected duplicates, uncertain pairing, stub/placeholder status>
- Last verified: <date + what was checked>
- Open questions: <for the user>
```

`docs/journal.md` — dated entries, append at the end:

```markdown
# Analysis Journal

## YYYY-MM-DD — <one-line summary>
- Did: <steps, key commands>
- Decisions: <parameters chosen + why>
- Outputs: <files in results/ (paths)>
- Next: <planned steps>
- Open questions: <for user / next session>
```

Update discipline:

- Resuming = read both docs BEFORE touching any file
- New data appears, or you learn a format/quirk → update `docs/data.md` in the same session
- Every meaningful step (env setup, QC pass, new figure, failed attempt) → append a journal entry; never end a session without one

## Quick Reference

| Situation | Rule |
|-----------|------|
| Rename/delete/overwrite raw file | Never |
| Path with spaces/parens | Quote it; never rename |
| Copy raw data into project | Never — reference by path |
| Derived h5ad, figures, tables | `results/`, new filenames |
| Corrupt or misnamed file | Quirk in data.md, ask user |
| New dependency | `uv add <pkg>` |
| Run script / notebook | `uv run script.py` / `uv run jupyter lab` |
| Inherit project with conda/venv | Migrate to uv project mode; journal the migration |

## Common Mistakes

- `uv venv` + `pip install` instead of project mode (`uv init` + `uv add`) — same tool, wrong mode; nothing is recorded in pyproject/lock
- Duplicating raw data into `data/raw/` staging dirs — wasted gigabytes, two sources of truth
- Journal written once at bootstrap then abandoned — every session ends with an entry
- data.md listing files without quirks/open questions — the quirks ARE the value
- Touching a raw path before checking `docs/journal.md` — the journal may record why that path looks "wrong"
- Deciding to rename "just this once" — that is exactly the violation this framework exists to prevent

**Red flags — STOP and ask the user:** the modification is only implied, not explicit; you are about to touch a raw path before journaling; it's "a quick fix" to a data file; a copy "doesn't count"; the user would "obviously want this".
