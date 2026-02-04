# Configuration Guide

## Overview

The trading bot is configured through `config/config.yaml` and optional environment variables in `.env`.

## Configuration File Structure

The bot uses a modular configuration structure:

1. **`config/ib_connection.yaml`** - IB connection and global settings (required)
2. **`config/stocks/*.yaml`** - One configuration file per stock (required for each stock you want to trade)

### IB Connection Configuration (`config/ib_connection.yaml`)

Contains shared settings for all stocks:

#### IB Connection Settings

```yaml
ib:
  host: "127.0.0.1"          # IB Gateway/TWS host (usually localhost)
  port: 7497                  # API port (7497 = TWS paper, 4001 = Gateway paper)
  client_id: 1               # Unique client ID (1-100)
  account: ""                # Account number (empty = default account)
```

**Port Numbers**:
- `7497`: TWS Paper Trading
- `4001`: IB Gateway Paper Trading
- `7496`: TWS Live Trading
- `4002`: IB Gateway Live Trading

#### Global Trading Parameters

```yaml
trading:
  max_risk_per_trade: 0.02   # Risk 2% of account per trade (0.02 = 2%)
  max_positions: 5           # Maximum number of open positions
  stop_loss_percentage: 0.02 # Default stop loss percentage (0.02 = 2%)
  daily_loss_limit: 0.05     # Stop trading if daily loss > 5%
  
  order_type: "MKT"          # "MKT" for market, "LMT" for limit orders
  limit_order_offset: 0.01   # Offset for limit orders (0.01 = 1 cent)
```

**Risk Parameters**:
- `max_risk_per_trade`: Default percentage of account equity risked per trade (can be overridden per stock)
- `max_positions`: Maximum concurrent open positions across all stocks
- `stop_loss_percentage`: Default stop loss distance from entry (can be overridden per stock)
- `daily_loss_limit`: Maximum daily loss before stopping trading

### Stock-Specific Configuration (`config/stocks/*.yaml`)

Each stock has its own configuration file. The filename must match the stock symbol:

- `config/stocks/AAPL.yaml` - Apple Inc.
- `config/stocks/TSLA.yaml` - Tesla Inc.
- `config/stocks/MSFT.yaml` - Microsoft Corporation

#### Stock Configuration Structure

```yaml
# Stock Information
stock:
  symbol: "AAPL"            # Stock symbol (must match filename)
  enabled: true             # Enable/disable trading for this stock
  description: "Apple Inc."

# Break and Retest Strategy Configuration
strategy:
  name: "BreakAndRetestStrategy"
  
  # Level Identification
  lookback_period: 50
  min_touch_count: 2
  level_tolerance: 0.005
  min_level_strength: 0.5
  
  # Breakout Detection
  break_threshold: 0.005
  require_volume_confirmation: true
  volume_multiplier: 1.2
  breakout_bars: 2
  
  # Retest Settings
  retest_threshold: 0.01
  max_retest_bars: 10
  retest_bounce_threshold: 0.003
  
  # Entry/Exit
  min_confidence: 0.6
  entry_on_bounce: true
  entry_buffer: 0.002
  profit_target_multiplier: 1.5
  use_trailing_stop: false
  trailing_stop_percentage: 0.01

# Stock-Specific Risk Management (optional overrides)
risk:
  max_risk_per_trade: 0.015  # Override global setting
  stop_loss_percentage: 0.025  # Override global setting
```

**Key Points**:
- Each stock can have different strategy parameters
- Stock configs inherit global settings from `ib_connection.yaml`
- Stock-specific overrides take precedence
- Only stocks with `enabled: true` will be traded

### Logging Configuration

```yaml
logging:
  level: "INFO"              # DEBUG, INFO, WARNING, ERROR, CRITICAL
  file: "logs/trading_bot.log"
  max_bytes: 10485760       # 10MB log file size before rotation
  backup_count: 5           # Number of backup log files to keep
  console: true             # Log to console
```

### Bot Settings

```yaml
bot:
  update_interval: 60       # Seconds between strategy evaluations
  enable_trading: false     # Set to true to enable live trading
```

**Important**: Keep `enable_trading: false` until you're ready for live trading!

## Environment Variables

Create a `.env` file (copy from `.env.example`) to override config values:

```bash
# IB Connection
IB_HOST=127.0.0.1
IB_PORT=7497
IB_CLIENT_ID=1
IB_ACCOUNT=

# Trading
ENABLE_TRADING=false
```

Environment variables take precedence over YAML config.

## Configuration Examples

### Conservative Setup (Paper Trading)

```yaml
ib:
  host: "127.0.0.1"
  port: 7497
  client_id: 1

trading:
  max_risk_per_trade: 0.01    # 1% risk per trade
  max_positions: 3             # Only 3 positions
  stop_loss_percentage: 0.02   # 2% stop loss
  daily_loss_limit: 0.03       # 3% daily loss limit

bot:
  enable_trading: false         # Paper trading only
```

### Aggressive Setup (Live Trading - Use with Caution!)

```yaml
ib:
  host: "127.0.0.1"
  port: 7496                    # Live trading port
  client_id: 1

trading:
  max_risk_per_trade: 0.03      # 3% risk per trade
  max_positions: 10             # Up to 10 positions
  stop_loss_percentage: 0.015   # 1.5% stop loss
  daily_loss_limit: 0.10        # 10% daily loss limit

bot:
  enable_trading: true          # Live trading enabled
```

### Break and Retest Strategy Setup

```yaml
strategy:
  name: "BreakAndRetestStrategy"
  symbols:
    - "AAPL"
    - "MSFT"
    - "GOOGL"
  lookback_period: 50
  min_touch_count: 2
  break_threshold: 0.005
  retest_threshold: 0.01
  volume_multiplier: 1.2
  min_confidence: 0.6
```

**Note**: The architecture supports future extension to multiple strategies, though currently one strategy is used per bot instance.

## Configuration Validation

The bot validates configuration on startup:
- Required fields present
- Valid port numbers
- Valid risk percentages (0-1 range)
- Valid symbol formats
- Log file paths writable

## Dynamic Configuration

Some settings can be changed at runtime:
- Strategy parameters (requires restart)
- Logging level (can be changed)
- Trading enabled/disabled (requires restart)

## Security Considerations

1. **Never commit `.env` file** to version control
2. **Use paper trading ports** for testing
3. **Start with `enable_trading: false`**
4. **Use conservative risk parameters** initially
5. **Monitor logs** for configuration errors

## Troubleshooting

### Connection Issues
- Check TWS/Gateway is running
- Verify API is enabled in TWS settings
- Check port number matches TWS/Gateway
- Ensure client_id is unique

### Configuration Errors
- Check YAML syntax (indentation matters)
- Validate all required fields present
- Check file paths are correct
- Review log files for errors

### Trading Issues
- Verify `enable_trading: true` for live trading
- Check account has sufficient buying power
- Validate symbols are tradeable
- Review risk limits aren't too restrictive

## Best Practices

1. **Start Conservative**: Use low risk percentages initially
2. **Test Thoroughly**: Paper trade before going live
3. **Monitor Closely**: Watch logs and performance
4. **Adjust Gradually**: Change one parameter at a time
5. **Document Changes**: Keep notes on configuration changes
6. **Backup Configs**: Save working configurations

## Understanding min_confidence

**Important**: `min_confidence` is a **threshold parameter** you configure, not a calculated value.

- **`min_confidence`**: A threshold you set (e.g., 0.6 = 60%) - only trades with confidence >= this value are executed
- **Confidence Score**: A calculated value (0.0-1.0) based on level strength, breakout quality, retest quality, and market context

See `docs/CONFIDENCE_CALCULATION.md` for detailed explanation of how confidence scores are calculated.

## Configuration File Examples

Example configuration files are provided in the `config/` directory:

- **`config/ib_connection.yaml.example`** - IB connection and global settings template
- **`config/stocks/*.yaml.example`** - Stock-specific configuration templates
  - `AAPL.yaml.example` - Example for stable stock
  - `TSLA.yaml.example` - Example for volatile stock
  - `MSFT.yaml.example` - Example for standard stock

### Quick Start

1. Copy IB connection config:
   ```bash
   cp config/ib_connection.yaml.example config/ib_connection.yaml
   ```

2. Edit `config/ib_connection.yaml`:
   - Set IB connection settings (host, port, client_id)
   - Configure global trading parameters
   - Set logging and bot settings

3. Create stock configurations:
   ```bash
   # Create stocks directory if it doesn't exist
   mkdir -p config/stocks
   
   # Copy example stock configs
   cp config/stocks/AAPL.yaml.example config/stocks/AAPL.yaml
   cp config/stocks/TSLA.yaml.example config/stocks/TSLA.yaml
   # Add more stocks as needed
   ```

4. Edit each stock config file:
   - Verify symbol matches filename
   - Adjust strategy parameters for that stock
   - Set `enabled: true` to enable trading
   - Override risk settings if needed

### Configuration File Priority

1. **`ib_connection.yaml`** - Loaded first, provides global settings
2. **`stocks/*.yaml`** - Loaded for each enabled stock, inherits global settings
3. **Stock-specific overrides** - Take precedence over global settings
4. **Environment variables** - Override all config files (if set)

### Adding a New Stock

1. Copy an example stock config:
   ```bash
   cp config/stocks/AAPL.yaml.example config/stocks/NEWSTOCK.yaml
   ```

2. Edit the new file:
   - Change `symbol` to match the stock (e.g., "GOOGL")
   - Adjust strategy parameters as needed
   - Set `enabled: true`
   - Add risk overrides if needed

3. The bot automatically loads all enabled stocks from `config/stocks/`
