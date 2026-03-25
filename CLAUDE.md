# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FinRL is a financial reinforcement learning framework for automated stock trading. It supports training, backtesting, and paper trading DRL agents across multiple asset classes (stocks, crypto, portfolio). Version 0.3.8. The project is transitioning toward FinRL-X (FinRL-Trading) as a next-generation production version.

## Development Commands

### Installation
```bash
pip install -e .
# or
poetry install
```

### Testing
```bash
pytest unit_tests/
# Run in Docker (CI environment)
bash docker/bin/build_container.sh && bash docker/bin/test.sh
```

### Code Quality
```bash
pre-commit run --all-files   # runs black, isort, flake8, pyupgrade
black .
isort .
flake8 .
mypy .
```

### Training Pipeline (examples/)
```bash
python examples/FinRL_StockTrading_2026_1_data.py    # download & preprocess
python examples/FinRL_StockTrading_2026_2_train.py   # train agents
python examples/FinRL_StockTrading_2026_3_Backtest.py # backtest & compare
```

### CLI Entry Point
```bash
python -m finrl --mode train   # or test, trade
```

## Architecture

### Three-Layer Design
1. **Applications** (`finrl/applications/`) — domain-specific workflows (stock trading, crypto, portfolio, HFT, imitation learning)
2. **Agents** (`finrl/agents/`) — DRL algorithm backends: ElegantRL, Stable Baselines 3, Ray RLlib, and portfolio optimization methods
3. **Meta/Environments** (`finrl/meta/`) — gym environments and data processors

### Train-Test-Trade Pipeline
- `finrl/train.py` — downloads data → preprocesses → creates gym env → trains agent → saves model
- `finrl/test.py` — loads test data → runs trained agent → returns performance metrics
- `finrl/trade.py` — backtesting mode or live Alpaca paper trading

### Data Flow
```
Data Source → DataProcessor.download_data()
           → clean_data()
           → add_technical_indicator()   # MACD, RSI, Bollinger Bands, etc.
           → add_vix() / add_turbulence()
           → df_to_array()               # (price_array, tech_array, turbulence_array)
           → StockTradingEnv             # gym environment
           → DRL Agent training
```

### Key Files
- `finrl/config.py` — training/test date ranges, technical indicators list, agent hyperparameters (A2C_PARAMS, PPO_PARAMS, etc.)
- `finrl/config_tickers.py` — ticker lists (DOW_30_TICKER, etc.)
- `finrl/config_private.py` — API keys (git-ignored; create locally for Alpaca trading)
- `finrl/meta/data_processor.py` — unified interface dispatching to source-specific processors
- `finrl/meta/data_processors/` — adapters: Yahoo Finance (default), Alpaca, WRDS, CCXT, and others

### Environment Variants (`finrl/meta/env_stock_trading/`)
- `env_stocktrading.py` — standard gym environment
- `env_stocktrading_np.py` — numpy-based (faster) version used for training
- `env_stocktrading_stoploss.py` — adds stop-loss logic
- `env_stock_papertrading.py` — Alpaca live paper trading integration

## Code Style
- Max line length: 127 (flake8)
- Formatter: black
- Import order: isort with `--py37-plus`
- Type hints: use `from __future__ import annotations` at top of files
- Ignored flake8 rules: F401 (unused imports), W503, E203
