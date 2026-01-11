# Development Workflow

**Always use `uv`, not `pip` or `poetry`.**
**Always use `gh` for GitHub operations.**

## Before Starting Any Task

**ALWAYS follow this sequence at the start of every new task:**

```sh
# 0. Initial setup (first time only)
gh repo set-default
gh auth login

# 1. Switch to main/master branch and get latest changes
git checkout main                # or 'git checkout master' if using master
git pull origin main             # Pull latest changes from remote
# If using master: git pull origin master

# 2. Create a new feature branch
git checkout -b feature/descriptive-task-name
# Branch naming: feature/*, fix/*, docs/*, test/*, refactor/*
# Example: feature/add-user-auth, fix/database-connection

# 3. Make changes and run code
uv run python your_script.py

# If errors occur:
# - Read the error message carefully
# - Fix the issue in the code
# - Run again: uv run python your_script.py
# - Repeat until successful
# Use Claude Code to help debug if needed

# 4. Run tests (if tests exist)
uv run pytest                    # Run all tests
uv run pytest tests/test_file.py # Specific test file
uv run pytest -k "test_name"     # Specific test
# Fix any failing tests before proceeding

# 5. Typecheck (REQUIRED)
uv tool run mypy .
# Fix all type errors

# 6. Lint before committing (REQUIRED)
uv tool run ruff check .         # Check all files
uv tool run ruff format .        # Format all files
uv tool run ruff check path/to/file.py  # Specific files
# Fix all lint errors before committing

# 7. Commit changes (only after all checks pass)
git add .
git commit -m "feat: your feature description"
git push -u origin feature/your-feature-name
# Follow Conventional Commits format

# 8. Before creating PR - final check
uv tool run ruff check . && uv tool run ruff format . && uv tool run mypy . && uv run pytest

# 9. Create Pull Request (REQUIRED - do not skip this step)
gh pr create --title "feat: Feature description" --body "Detailed explanation of changes"
# This will open your browser or prompt for PR details
# Make sure the PR is successfully created before proceeding

# 10. Self-review (REQUIRED)
gh pr view --web
# Review your own PR using the checklist below
gh pr comment --body "✅ Self-review complete: [your findings]"

# 11. Address feedback
# Read your self-review comments
gh pr view --comments
# Fix issues found in self-review
# Repeat steps 3-8 until all issues are resolved
# Push changes
git add .
git commit -m "fix: address self-review feedback"
git push
```

## Critical: Task Start Checklist

**Before starting ANY new task, verify you have completed:**

- [ ] Switched to main/master branch: `git checkout main`
- [ ] Pulled latest changes: `git pull origin main`
- [ ] Created new feature branch: `git checkout -b feature/task-name`
- [ ] Confirmed you are on the new branch: `git branch --show-current`

**After completing the task, verify:**

- [ ] All tests pass
- [ ] All lints pass
- [ ] All type checks pass
- [ ] Changes committed and pushed
- [ ] **Pull Request created** (use `gh pr create`)
- [ ] Self-review completed with comments

## Package Management

```sh
# Add dependencies
uv add package-name              # Add runtime dependency
uv add --dev package-name        # Add development dependency

# Remove dependencies
uv remove package-name

# Sync dependencies
uv sync                          # Install all dependencies from lock file

# Update lock file
uv lock                          # Regenerate uv.lock
```

## Working with Claude Code

```sh
# Ask Claude Code to implement features
claude-code "Implement user authentication with JWT"

# Ask for debugging help
claude-code "Debug: TypeError in parse_data function, error message: [paste error]"

# Ask for test generation
claude-code "Generate pytest tests for UserService class"

# Ask for refactoring
claude-code "Refactor database.py to use async/await pattern"

# Ask for type annotations
claude-code "Add type annotations to all functions in utils.py"
```

## Rules

### MUST
- Use `uv` for all Python execution and package management
- Use `gh` for all GitHub operations (git allowed only for branch/commit operations)
- Always create a working branch before making changes
- Run and fix code recursively until execution succeeds
- Add type annotations to all new functions and classes
- Run `uv tool run mypy .` and fix all type errors before committing
- Run `uv tool run ruff check .` and `uv tool run ruff format .` before every commit
- Run tests with `uv run pytest` if tests exist
- All checks must pass before creating PR
- Create PR only after all errors are fixed
- Perform self-review using the checklist and add comments to PR
- Fix issues recursively based on review comments
- Follow Conventional Commits format for commit messages

### SHOULD
- Write tests for new features using pytest
- Keep functions small and focused
- Write clear docstrings for public functions
- Update documentation when changing functionality

### Commit Message Format
Follow Conventional Commits:
- `feat: add user authentication`
- `fix: resolve database connection issue`
- `docs: update README with setup instructions`
- `test: add tests for payment module`
- `refactor: simplify error handling logic`
- `chore: update dependencies`

## Self-Review Checklist

Before approving your own PR, verify:

- [ ] Code follows project style (Ruff passes with no errors)
- [ ] Type annotations are complete (mypy passes with no errors)
- [ ] Tests are added/updated and all passing
- [ ] No debug code, print statements, or commented-out code left
- [ ] Error handling is appropriate and comprehensive
- [ ] Documentation/docstrings are clear and up-to-date
- [ ] No sensitive data, API keys, or secrets in code
- [ ] Variable and function names are descriptive
- [ ] Code is DRY (Don't Repeat Yourself)
- [ ] Edge cases are handled
- [ ] Performance considerations are addressed

Post your review as a comment:
```sh
gh pr comment --body "
✅ Self-review complete

**What I checked:**
- All lints pass
- Type checking passes
- Tests added and passing
- No sensitive data

**Potential concerns:**
- [Any concerns or areas that need extra attention]
"
```

## Project Setup

### Ruff Configuration
Add to `pyproject.toml`:
```toml
[tool.ruff]
line-length = 88
target-version = "py311"

[tool.ruff.lint]
select = [
    "E",   # pycodestyle errors
    "F",   # pyflakes
    "I",   # isort
    "N",   # pep8-naming
    "W",   # pycodestyle warnings
    "UP",  # pyupgrade
    "B",   # flake8-bugbear
]
ignore = []

[tool.ruff.format]
quote-style = "double"
indent-style = "space"
```

### Mypy Configuration
Add to `pyproject.toml`:
```toml
[tool.mypy]
python_version = "3.11"
strict = true
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
```

### Pytest Configuration
Add to `pyproject.toml`:
```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py"]
python_functions = ["test_*"]
addopts = "-v --tb=short"
```

## Security Best Practices

- **NEVER** commit secrets, API keys, passwords, or tokens
- Use `.env` files for environment variables (add `.env` to `.gitignore`)
- Use `python-dotenv` to load environment variables
- Review diffs before committing: `gh pr diff`
- Use `git-secrets` or similar tools to prevent accidental commits
- Rotate any accidentally committed secrets immediately

Example `.env` usage:
```python
from dotenv import load_dotenv
import os

load_dotenv()
api_key = os.getenv("API_KEY")
```

## Troubleshooting

### Common Issues

**uv command not found:**
```sh
curl -LsSf https://astral.sh/uv/install.sh | sh
```

**gh command not found:**
```sh
# macOS
brew install gh

# Linux
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
```

**Ruff/mypy not found:**
```sh
uv tool install ruff
uv tool install mypy
```

**Type errors in third-party packages:**
```sh
uv add --dev types-package-name
```

## Reference

- [CLAUDE.md best practices](https://zenn.dev/explaza/articles/a387d2bf1cb448)
- [uv documentation](https://docs.astral.sh/uv/)
- [Ruff documentation](https://docs.astral.sh/ruff/)
- [mypy documentation](https://mypy.readthedocs.io/)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [GitHub CLI documentation](https://cli.github.com/manual/)