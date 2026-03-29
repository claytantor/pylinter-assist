# pylinter-assist

Context-aware Python linting with smart pattern heuristics for PR review.

A soft PR reviewer that combines Pylint with custom pattern checks to catch hardcoded secrets, missing FastAPI docstrings, and other issues traditional linters miss. Complements hard CI tests; branches are assignable by the user.

## Features

| Check | Code | Severity | Catches |
|-------|------|----------|---------|
| Pylint | C/W/R/E/F | varies | Standard Python quality issues |
| Hardcoded password/secret | HCS001 | ERROR | `password = "abc123"` |
| Credentials in URL | HCS002 | ERROR | `https://user:pass@host` |
| Hardcoded IP address | HCS003 | ERROR | `HOST = "10.0.0.5"` |
| Hardcoded localhost URL | HCS004 | ERROR | `"http://localhost:8000"` |
| AWS/GCP access key | HCS005 | ERROR | `AKIAIOSFODNN7EXAMPLE` |
| FastAPI missing docstring | FAD001 | WARNING | `@router.get("/")` without docstring |
| useEffect missing deps | RUE001 | WARNING | `useEffect(() => { ... })` with no `[]` |
| useEffect suspicious deps | RUE002 | INFO | `useEffect(..., [])` references outer vars |

## Installation

```bash
uv sync
```

## Usage

```bash
uv run lint-pr [TARGET] [OPTIONS]
```

### Targets

| Target | Description |
|--------|-------------|
| `pr <number>` | Lint all files changed in a GitHub PR |
| `staged` | Lint git-staged files |
| `diff <file>` | Lint files from a unified diff file |
| `files <path>...` | Lint explicit files or directories |

### Options

| Flag | Description |
|------|-------------|
| `--format text\|json\|markdown` | Output format (default: markdown) |
| `--config <path>` | Custom `.linting-rules.yml` path |
| `--post-comment` / `--no-post-comment` | Post result as GitHub PR comment |
| `--fail-on-warning` | Also fail on warnings (default: errors only) |

### Examples

```bash
# Lint PR #42 and post a comment
uv run lint-pr pr 42 --post-comment

# Lint staged files before commit
uv run lint-pr staged --format text

# Lint from a saved diff
uv run lint-pr diff changes.patch

# Lint a whole directory, output JSON
uv run lint-pr files src/ --format json

# Use custom rules
uv run lint-pr files src/ --config my-rules.yml
```

## Configuration

Copy `.linting-rules.yml` to your project root and edit:

```yaml
pylint:
  enabled: true
  disable: [C0114, C0115]   # suppress module/class docstring warnings

hardcoded_secrets:
  enabled: true
  skip_ip_check: false

fastapi_docstring:
  severity: warning

react_useeffect_deps:
  severity: warning

github:
  post_comment: true
  fail_on_error: true
  fail_on_warning: false
```

## Publishing to ClawHub

```bash
# Install dependencies
uv sync

# Publish the skill
make publish

# Auto-increment minor version and publish
make publish-bump

# Dry run before publishing
make publish-dry-run
```

## Development

```bash
# Run tests
uv run pytest

# Format code
uv run ruff format .

# Lint code
uv run pylint pylinter_assist
```

## License

MIT
