# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A stock intelligence analysis system for A-shares, HK stocks, and US stocks. It uses AI models to analyze stocks and sends decision dashboards to notification channels (WeCom, Feishu, Telegram, Email).

## Common Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Run analysis (CLI mode)
python main.py

# Run with Web UI
python main.py --webui       # Web UI + scheduled analysis
python main.py --webui-only  # Web UI only

# Run tests
./test.sh quick              # Fast single stock test (recommended)
./test.sh syntax             # Python syntax check
./test.sh code               # Stock code recognition tests
./test.sh yfinance           # YFinance code conversion tests
./test.sh flake8             # Static analysis
./test.sh market             # Market review test
./test.sh a-stock            # A-share analysis
./test.sh us-stock           # US stock analysis
./test.sh all                # Run all tests

# CI gate (syntax + flake8 + tests)
./scripts/ci_gate.sh

# Syntax check
python -m py_compile main.py src/*.py data_provider/*.py
flake8 main.py src/ --max-line-length=120
```

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        main.py                                   │
│  Entry point: handles CLI args (--webui, --webui-only, --serve)│
└─────────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
┌───────────────┐    ┌───────────────┐    ┌───────────────┐
│   analyzer    │    │   scheduler   │    │  api/app.py   │
│ stock_analyzer │    │ (cron jobs)   │    │  (FastAPI)    │
│ market_analyzer│    └───────────────┘    └───────────────┘
└───────────────┘                              │
        │                              ┌──────┴──────┬──────────┐
        ▼                              ▼             ▼          ▼
┌───────────────┐              ┌───────────┐  ┌──────────┐ ┌───────┐
│ data_provider │              │ endpoints │  │ services │ │ agent │
│ (fetchers)    │              │ (REST API)│  │          │ │(LLM)  │
└───────────────┘              └───────────┘  └──────────┘ └───────┘
```

### Core Modules

| Directory | Purpose |
|-----------|---------|
| `src/` | Core application code |
| `src/analyzer.py` | Main stock analysis logic |
| `src/stock_analyzer.py` | Individual stock analysis |
| `src/market_analyzer.py` | Market overview & review |
| `src/notification.py` | Multi-channel notifications (WeCom, Feishu, Telegram, Email) |
| `src/config.py` | Configuration management |
| `src/agent/` | AI Agent for strategy Q&A |
| `src/services/` | Business logic services |
| `src/core/` | Core utilities (backtest, pipeline, config) |
| `data_provider/` | Market data fetchers (Akshare, Tushare, YFinance, etc.) |
| `api/` | FastAPI REST endpoints |
| `bot/` | Telegram/Discord bot handlers |
| `apps/dsa-web/` | React frontend (built to `static/`) |

### Data Flow

1. **CLI/Scheduler** triggers `main.py` or `scheduler.py`
2. **Analyzer** fetches stock data via `data_provider/*` fetchers
3. **AI Analysis** uses configured LLM (Gemini, Claude, OpenAI-compatible)
4. **Notification** sends results via `notification.py` to configured channels

### Agent System

The `AGENT_MODE=true` enables conversational stock strategy Q&A:
- Strategies defined in `strategies/*.yaml`
- Agent uses tools from `src/agent/tools/` (market data, search, analysis)
- Web UI at `/chat`, Bot via `/ask <code> [strategy]`

## Configuration

All config via environment variables (`.env`). Key variables:
- `STOCK_LIST`: Stock codes to analyze (e.g., `600519,hk00700,AAPL`)
- AI keys: `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, `GEMINI_API_KEY`, etc.
- Notification: `WECHAT_WEBHOOK_URL`, `TELEGRAM_BOT_TOKEN`, etc.
- `AGENT_MODE`: Enable agent strategy chat

See `.env.example` for full list.

## Code Style

- Line width: 120
- Formatter: `black` + `isort` + `flake8`
- Type hints: Use `List[X]`, `Dict[X,Y]`, `Optional[X]` (not built-in)
- Docstrings: Google-style in English
- Comments: English

See `AGENTS.md` for detailed guidelines.
