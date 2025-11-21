# Repository Guidelines

## Project Structure & Modules
- Core package lives in `files_to_prompt/`: `cli.py` exposes the Click-based CLI, `__main__.py` enables `python -m files_to_prompt`, and `__init__.py` holds package metadata.
- Tests sit in `tests/test_files_to_prompt.py`; keep new tests in this suite to mirror CLI behaviors.
- Tooling and metadata are defined in `pyproject.toml`. Runtime dependency footprint is minimal (`click` only); avoid introducing new ones without discussion.

## Setup, Build, and Development Commands
- Create an isolated environment: `python -m venv venv && source venv/bin/activate`.
- Install dependencies for local work: `pip install -e '.[test]'`.
- Run the CLI during development without installing globally: `python -m files_to_prompt path/to/dir -m` or `python -m files_to_prompt --help`.
- Package metadata is driven by `pyproject.toml`; bump versions there when preparing releases.

## Coding Style & Naming Conventions
- Follow standard Python 3.8+ conventions: 4-space indentation, snake_case for functions/variables, and meaningful, lowercase CLI option names.
- Keep functions small and reuse existing helpers (e.g., `process_path`, `print_as_markdown`) rather than adding branching in `cli()`.
- Prefer stdlib solutions; avoid new dependencies unless they deliver clear value.
- Match current import style (stdlib first, blank line, third-party).

## Testing Guidelines
- Primary test runner: `pytest`.
- Add or extend cases in `tests/test_files_to_prompt.py` when modifying CLI flags, output formats, or ignore rules.
- Include edge cases (hidden files, ignored paths, mixed encodings) and assert on concrete output strings, since the tool is output-centric.
- Run `pytest` before opening a PR; aim for coverage of new code paths rather than global percentage targets.

## Commit & Pull Request Guidelines
- Use concise, imperative commit subjects (e.g., `Add markdown line-number option`, `Refine ignore patterns`). One logical change per commit where possible.
- PRs should summarize intent, list main changes, and note testing performed (`pytest`). Link issues when they exist and include sample CLI output for user-facing changes.
- Keep diffs small and readable; prefer refactors in separate PRs from feature additions.

## Security & Configuration Tips
- The CLI reads files recursively; respect repository `.gitignore` handling and default hidden-file exclusion to avoid leaking secrets in prompts.
- When adding new options that affect file selection or output destinations, validate user inputs and maintain safe defaults (no overwriting unless requested).
