# AGENTS.md

This file defines the unified behavior guidelines for development, Issue analysis, and PR review in this repository.

## 1. General Collaboration Principles

- **Language & Stack**: Python 3.10+, follow repository architecture and directory boundaries.
- **Configuration**: Use `.env` (see `.env.example`).
- **Code Quality**: Prioritize runnable, regression-verifiable, traceable code (clear logs/errors).
- **Style Constraints**:
  - Line width: 120
  - `black` + `isort` + `flake8`
  - Critical changes require syntax check (`py_compile`) or corresponding test verification.
  - New or modified code comments must use English.
- **Git Constraints**:
  - Do not execute `git commit` without explicit confirmation.
  - Commit messages exclude `Co-Authored-By` attribution.
  - All subsequent commit messages must be in English.

## 2. Build, Lint & Test Commands

### Quick Reference

```bash
# Syntax check (all critical files)
python -m py_compile main.py src/config.py src/notification.py src/analyzer.py

# Syntax check specific modules
python -m py_compile src/*.py data_provider/*.py

# Flake8 critical checks only
flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics

# Full flake8 with project line length
flake8 main.py src/ --max-line-length=120

# Run single test scenario
./test.sh quick       # Quick test (single stock)
./test.sh syntax     # Syntax check only
./test.sh code       # Code recognition tests
./test.sh yfinance   # YFinance conversion tests
./test.sh flake8     # Flake8 linting
./test.sh market    # Market review test
./test.sh a-stock   # A-share stock test
./test.sh us-stock  # US stock test
./test.sh dry-run   # Dry-run mode

# Run all tests
./test.sh all

# CI gate (runs in CI)
./scripts/ci_gate.sh
```

### Test Scenarios Available

| Scenario | Description |
|----------|-------------|
| `quick` | Fast single stock test (recommended) |
| `syntax` | Python syntax check |
| `code` | Stock code recognition tests |
| `yfinance` | YFinance code conversion tests |
| `flake8` | Static analysis check |
| `market` | Market review only |
| `a-stock` | A-share stock analysis |
| `etf` | ETF analysis |
| `hk-stock` | Hong Kong stock analysis |
| `us-stock` | US stock analysis |
| `mixed` | Mixed market analysis |
| `single` | Single stock push mode |
| `dry-run` | Data fetch only, no AI analysis |
| `full` | Complete flow test |
| `all` | Run all tests |

## 3. Code Style Guidelines

### Formatting

- **Line length**: 120 characters max
- **Formatter**: `black` (auto-formats on save)
- **Import sort**: `isort` (sorted by type: stdlib → third-party → local)
- **No trailing whitespace**

### Imports

```python
# Standard library
import os
import re
from pathlib import Path
from typing import List, Optional, Tuple

# Third-party
from dotenv import load_dotenv
from dataclasses import dataclass, field

# Local (relative imports)
from src.config import get_config
from src.analyzer import AnalysisResult
```

### Type Hints

- Use type hints for all function parameters and return values
- Use `Optional[X]` instead of `X | None`
- Use `List[X]`, `Dict[X, Y]` from typing (not built-in list/dict)

```python
def get_config() -> Config:
    ...

def validate(self) -> List[str]:
    ...
```

### Naming Conventions

- **Classes**: `PascalCase` (e.g., `NotificationService`, `Config`)
- **Functions/methods**: `snake_case` (e.g., `get_config()`, `_detect_all_channels()`)
- **Constants**: `UPPER_SNAKE_CASE` (e.g., `WECHAT_IMAGE_MAX_BYTES`)
- **Private methods**: prefix with `_` (e.g., `_is_telegram_configured()`)
- **Variables**: `snake_case` (e.g., `stock_list`, `wechat_url`)

### Docstrings

Use Google-style docstrings in English:

```python
def get_config() -> Config:
    """
    Get global configuration instance.

    Returns:
        Config: The singleton configuration instance.
    """
```

### Error Handling

- Never suppress errors with `except: pass` or bare `except:`
- Log errors before re-raising or handling
- Use specific exception types
- Always handle cleanup in `finally` blocks when needed

```python
try:
    result = fetcher.fetch()
except requests.RequestException as e:
    logger.error(f"Failed to fetch data: {e}")
    raise
```

### Logging

- Use `logger = logging.getLogger(__name__)` at module level
- Log levels: `DEBUG` (dev), `INFO` (normal), `WARNING` (attention), `ERROR` (failure)

### Constants

- Put module-level constants at the top of the file after imports
- Use meaningful names (e.g., `MAX_RETRIES = 3` not `MAX = 3`)

## 4. Issue Analysis Principles

Each Issue must answer 3 questions:

1. **Reasonable**
   - Does it describe real impact (bug, data error, performance, UX)?
   - Is there verifiable evidence (logs, screenshots, steps)?
   - Is it relevant to project goals?

2. **Valid Issue**
   - Is it a defect/feature missing/regression/docs error?
   - Can it be traced to repository responsibility?
   - If it's a usage problem, convert to docs improvement

3. **Solvable**
   - Can it be reliably reproduced?
   - Are dependencies controllable?
   - What's the risk level?
   - Is there a workaround?

### Issue Conclusion Template

- **Conclusion**: `Valid / Partially Valid / Invalid`
- **Category**: `bug / feature / docs / question / external`
- **Priority**: `P0 / P1 / P2 / P3`
- **Difficulty**: `easy / medium / hard`
- **Suggested Action**: `Fix Now / Schedule / Doc Clarification / Close`

## 5. PR Analysis Principles

### Review Order

1. **Necessity**: Does it solve a clear problem with business value?
2. **Traceability**: Is there a linked Issue (`Fixes #xxx` or `Refs #xxx`)?
3. **Type**: `fix / feat / refactor / docs / chore / test`
4. **Description Completeness**: Must include:
   - Background & problem
   - Change scope
   - Verification method & results
   - Compatibility notes
   - Rollback plan
   - Issue closing statement if applicable
5. **Merge Readiness**: Target clear, changes match description, tests pass, no blocking risks

### PR Review Output

- **Necessity**: Pass/Fail
- **Has Issue**: Yes/No (issue #)
- **Type**: fix/feat/...
- **Description**: Complete/Incomplete (missing items)
- **Ready to Merge**: Yes/No + required changes

## 6. Delivery & Release Sync

- Feature/fix completion requires docs update:
  - `README.md` (user-facing capabilities, config changes)
  - `docs/CHANGELOG.md` (version changes, impact, compatibility)
- Version tags in commit messages:
  - `#patch`: Fixes, small changes
  - `#minor`: New features, backward compatible
  - `#major`: Breaking changes, major architecture
  - `#skip` / `#none`: No version tag
- PRs fixing issues must include closing statement (`Fixes #xxx`)

## 7. Quick Check Commands

```bash
./test.sh syntax
python -m py_compile main.py src/*.py data_provider/*.py
flake8 main.py src/ --max-line-length=120
```
