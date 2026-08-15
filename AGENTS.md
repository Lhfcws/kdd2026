# Repository Guidelines

## Project Structure & Module Organization

This repository is a KDD 2026 paper-reading corpus, not an application codebase. Root-level files provide indexing and synthesis: `README.md` lists paper notes and tag statistics, `manifest.json` maps each paper to its artifacts, `accepted_paper.csv` tracks accepted papers, and `plan.md` plus `kdd2026_*overview/observation*.md` contain planning or summary notes.

Paper artifacts live in `papers_and_notes/`. Each paper should keep the same title stem across `.pdf`, `.txt`, and `.md` files. Expanded reading pages live in `deep_read/*.html`.

## Build, Test, and Development Commands

There is no build step. Use these checks before committing content changes:

- `rg --files` lists repository content quickly.
- `python3 -m json.tool manifest.json >/dev/null` validates manifest JSON.
- `find papers_and_notes -maxdepth 1 -name '*.md' | wc -l` counts paper notes.
- `git diff -- README.md manifest.json '*.md'` reviews index and note edits.

When adding or moving papers, manually verify that `README.md` links and `manifest.json` paths point to existing files.

## Coding Style & Naming Conventions

Use Markdown for notes and keep headings descriptive. Paper note files should preserve the existing five-part structure: background and idea, core method, performance summary, value and limitations, and industrial feasibility.

Name paper artifacts as `<English Title With Spaces_ Subtitle>.<ext>`, matching the current files. Avoid renaming existing artifacts unless `README.md` and `manifest.json` are updated in the same change.

## Testing Guidelines

No automated tests or coverage requirements are defined. Treat validation as content integrity checking:

- Confirm every `manifest.json` entry has existing `.pdf`, `.txt`, and `.md` paths.
- Confirm every `README.md` paper link resolves under `papers_and_notes/`.
- Spot-check generated `.txt` and `.md` content against the source PDF for title, claims, and terminology.

## Commit & Pull Request Guidelines

Recent commits use short imperative or scoped messages, such as `rename: index.md -> README.md` and `papers: move artifacts into papers_and_notes`. Prefer `file-or-scope: action` when a change is localized.

Pull requests should summarize changed papers or synthesis files, note regenerated artifacts, and call out index or manifest updates. Include screenshots only for rendered `deep_read/*.html` changes.

## Agent-Specific Instructions

Before editing, check whether the target file exists and preserve user-authored changes. Do not overwrite paper artifacts blindly. For new papers, update the artifact set, `manifest.json`, and `README.md` together so the corpus remains navigable.
