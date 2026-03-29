# pylinter-assist — OpenClaw Skill

name: pylinter-assist
version: 0.4.0+0+0

## Prerequisites

Before installing this skill, ensure you have the following installed:

### uv (Python package manager)

[Install via official script](https://astral.sh/uv/install.sh):

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

**Important:** After installation, add uv to your PATH if not already present:

```bash
# Add to your shell config (bash/zsh/fish)
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc  # or ~/.zshrc
source ~/.bashrc  # or source ~/.zshrc

# Verify installation
which uv
uv --version
```

If `uv` is not found, check that `~/.local/bin` is in your PATH:
```bash
echo $PATH | grep -o '.*/.local/bin'
```

### npm (Node.js package manager, for ClawHub CLI)

```bash
# macOS/Linux
brew install node  # or your preferred package manager

# Or download from https://nodejs.org/
```

### clawhub CLI (OpenClaw skill installer)

```bash
npm i -g clawhub
```

## Description

Context-aware Python linting with smart pattern heuristics for PR review.
Automatically integrates into GitHub workflows to lint pull requests and provide feedback.

## Features

- Combines Pylint with custom pattern checks (hardcoded secrets, missing docstrings, etc.)
- Auto-lints PRs when integrated via GitHub Actions
- Supports multiple output formats (markdown, text, json)
- Configurable rules via `.linting-rules.yml`

## Installation via ClawHub

Install the skill using the ClawHub CLI:

```bash
clawhub install pylinter-assist
```

The skill will be installed to `~/.openclaw/workspace/skills/pylinter-assist`.

## Integration Steps

### Step 0: Post-Installation Setup

After the skill is installed as a snapshot, run this to register the CLI in your PATH:

```bash
cd ~/.openclaw/workspace/skills/pylinter-assist
uv sync
python -m venv .venv-lint
source .venv-lint/bin/activate
pip install -e .
```

This creates an isolated virtual environment and registers the `lint-pr` command.

Deactivate when done: `deactivate`

### Step 1: Use the CLI

Choose one of these methods:

**Method A: Install via pip in standard venv (recommended)**
```bash
uv sync
python -m venv .venv-lint
source .venv-lint/bin/activate
pip install -e .
lint-pr pr 42 --post-comment
```

**Method B: Use uv run (no installation required)**
```bash
uv sync
uv run lint-pr pr 42 --post-comment
```

**Method C: Run wrapper script directly**
```bash
uv sync
uv run python scripts/lint_pr.py pr 42 --post-comment
```

### Step 2: Add GitHub Actions Workflow

Copy the workflow file to your project:

```bash
mkdir -p .github/workflows
cp /path/to/pylinter-assist/.github/workflows/lint-pr.yml .github/workflows/
```

Or download directly:

```bash
curl -o .github/workflows/lint-pr.yml \
  https://raw.githubusercontent.com/claytantor/pylinter-assist/main/.github/workflows/lint-pr.yml
```

### Step 3: Add Configuration (Optional)

Copy and customize the rules file:

```bash
cp /path/to/pylinter-assist/.linting-rules.yml .linting-rules.yml
```

### Step 4: Set Repository Permissions

Ensure your repository has these permissions enabled:

```yaml
permissions:
  contents: read
  pull-requests: write
  issues: write
```

### Step 5: Commit and Push

The workflow will automatically trigger on new PRs.

## Usage

### Direct CLI Commands

After installation (Method A), run:

```bash
lint-pr [TARGET] [OPTIONS]
```

Or use uv run (Method B):

```bash
uv run lint-pr [TARGET] [OPTIONS]
```

Or use wrapper script (Method C):

```bash
uv run python scripts/lint_pr.py [TARGET] [OPTIONS]
```

### Targets

| Target | Description |
|--------|-------------|
| `pr <number>` | Lint all files changed in a GitHub PR |
| `staged` | Lint git-staged files |
| `diff <file>` | Lint files from a unified diff file |
| `files <path>...` | Lint explicit files or directories |

### Examples

```bash
# Activate venv first (Method A only)
source .venv-lint/bin/activate

# Lint PR #42 and post comment
lint-pr pr 42 --post-comment

# Lint staged changes
lint-pr staged

# Lint a diff file
lint-pr diff /tmp/changes.diff

# Lint specific files
lint-pr files src/ tests/

# Or use uv run (no venv activation needed)
uv run lint-pr pr 42 --post-comment
```

### Options

| Flag | Description |
|------|-------------|
| `--format text\|json\|markdown` | Output format (default: markdown) |
| `--config <path>` | Custom `.linting-rules.yml` path |
| `--post-comment` / `--no-post-comment` | Post result as GitHub PR comment |
| `--fail-on-warning` | Also fail on warnings (default: errors only) |

## Checks Performed

| Check | Code | Severity | Catches |
|-------|------|----------|---------|
| Pylint | C/W/R/E/F | varies | Standard Python quality issues |
| Hardcoded password/secret | HCS001 | ERROR | `password = "abc123"` |
| Credentials in URL | HCS002 | ERROR | `https://user:pass@host` |
| Hardcoded IP address | HCS003 | ERROR | `HOST = "10.0.0.5"` |
| Hardcoded localhost URL | HCS004 | ERROR | `"http://localhost:8000"` |
| AWS/GCP access key | HCS005 | ERROR | `AKIAIOSFODNN7EXAMPLE` |
| FastAPI missing docstring | FAD001 | WARNING | `@router.get("/")` without docstring |
| useEffect missing deps | RUE001 | WARNING | React useEffect with no deps array |
| useEffect suspicious deps | RUE002 | INFO | useEffect referencing outer vars |

## Configuration

Copy `.linting-rules.yml` to your project root and customize:

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

## Workflow Triggers

- **pull_request**: Auto-lints all files changed in a PR
- **workflow_dispatch**: Manual trigger via GitHub UI or API

## Manual Trigger Examples

**GitHub CLI:**
```bash
gh workflow run lint-pr.yml -f pr_number=42 -f format=markdown -f post_comment=true
```

**REST API:**
```bash
curl -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/OWNER/REPO/actions/workflows/lint-pr.yml/dispatches \
  -d '{"ref":"main","inputs":{"pr_number":"42","format":"markdown","post_comment":"true"}}'
```

## Output

- Markdown reports posted as PR comments (when enabled)
- JSON artifacts uploaded for 14 days retention
- Exit code 1 if errors found (when `fail_on_error: true`)

## Troubleshooting

### `command not found: uv`

If you get "command not found: uv" after installation:

```bash
# Check if uv is installed
ls -la ~/.local/bin/uv

# Add to PATH if missing
export PATH="$HOME/.local/bin:$PATH"

# Make it permanent
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

### `command not found: lint-pr` after installation

The `lint-pr` CLI is installed to the virtual environment's bin directory:

```bash
# Check if it exists
ls -la .venv-lint/bin/lint-pr

# Either activate the venv
source .venv-lint/bin/activate

# Or call it directly
./.venv-lint/bin/lint-pr pr 42
```

### uv sync fails with "no Python found"

Ensure Python is installed:

```bash
python3 --version

# If using pyenv or similar, ensure Python is available
pyenv versions
```

