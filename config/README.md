# Configuration Files Structure

## Overview

The trading bot uses a modular configuration structure:
- **One config file per stock** - Each stock has its own configuration file
- **Separate IB connection config** - Shared connection settings

## Configuration Files

### 1. IB Connection Configuration

**File**: `config/ib_connection.yaml`

Contains:
- Interactive Brokers connection settings (host, port, client_id, account)
- Global trading parameters (risk management, order settings)
- Logging configuration
- Bot settings

This file is shared across all stocks.

### 2. Stock-Specific Configurations

**Location**: `config/stocks/`

Each stock has its own configuration file named after its symbol:
- `config/stocks/AAPL.yaml` - Apple Inc. configuration
- `config/stocks/TSLA.yaml` - Tesla Inc. configuration
- `config/stocks/MSFT.yaml` - Microsoft Corporation configuration
- etc.

Each stock config file contains:
- **Stock Information**: Symbol, enabled status, description
- **Strategy Parameters**: Break and retest strategy settings (used for both live trading and backtesting)
- **Risk Management**: Stock-specific risk overrides (optional)
- **Backtest Settings**: Date range, timeframe, data source, output settings (for backtesting only)

## Setup Instructions

### Step 1: Copy IB Connection Config

```bash
cp config/ib_connection.yaml.example config/ib_connection.yaml
```

Edit `ib_connection.yaml` with your IB connection settings.

### Step 2: Create Stock Configurations

For each stock you want to trade:

```bash
# Copy example stock config
cp config/stocks/AAPL.yaml.example config/stocks/AAPL.yaml

# Edit the file to customize:
# - Strategy parameters (for live trading and backtesting)
# - Backtest settings (date range, timeframe, etc.)
# - Risk settings (optional overrides)
```

### Step 3: Configure Backtest Settings

In each stock's config file, configure backtest section:

```yaml
backtest:
  enabled: true              # Enable backtesting for this stock
  start_date: "2024-01-01"   # Backtest start date
  end_date: "2024-12-31"     # Backtest end date
  timeframe: "1 hour"        # Bar size for backtesting
  data_source: "ib"          # Data source
  # ... other backtest settings
```

### Step 4: Enable/Disable Stocks

In each stock's config file, set:

```yaml
stock:
  enabled: true   # Set to false to disable trading for this stock
```

## Configuration Priority

1. **IB Connection Config** (`ib_connection.yaml`) - Loaded first
2. **Stock Configs** (`stocks/*.yaml`) - Loaded for each enabled stock
3. **Stock configs override** global settings where specified

## Example Structure

```
config/
├── ib_connection.yaml          # IB connection (shared)
├── ib_connection.yaml.example  # Example template
├── stocks/
│   ├── AAPL.yaml              # Apple stock config
│   ├── AAPL.yaml.example      # Example template
│   ├── TSLA.yaml              # Tesla stock config
│   ├── TSLA.yaml.example      # Example template
│   ├── MSFT.yaml              # Microsoft stock config
│   └── MSFT.yaml.example      # Example template
└── README.md                  # This file
```

## Adding a New Stock

1. Copy an example stock config:
   ```bash
   cp config/stocks/AAPL.yaml.example config/stocks/NEWSTOCK.yaml
   ```

2. Edit the new file:
   - Change `symbol` to match the stock
   - Adjust strategy parameters as needed
   - Set `enabled: true`

3. The bot will automatically load all enabled stocks from `config/stocks/`

## Stock-Specific Overrides

Each stock can override:
- Strategy parameters (break_threshold, min_confidence, etc.)
- Risk management (max_risk_per_trade, stop_loss_percentage)
- Entry/exit parameters

This allows you to:
- Use tighter parameters for stable stocks (AAPL, MSFT)
- Use wider parameters for volatile stocks (TSLA)
- Customize each stock based on its characteristics

## Benefits of This Structure

1. **Modular**: Easy to add/remove stocks
2. **Organized**: Each stock's settings in one place
3. **Flexible**: Different parameters per stock
4. **Maintainable**: Easy to update individual stocks
5. **Scalable**: Add as many stocks as needed

## Notes

- Stock config files must be named after the stock symbol (e.g., `AAPL.yaml`)
- Only stocks with `enabled: true` will be traded
- Stock configs inherit global settings from `ib_connection.yaml`
- Stock-specific overrides take precedence over global settings
