# Backtest Configuration Guide

## Overview

The backtesting system uses the same stock configuration files from `config/stocks/` that are used for live trading. Each stock config file contains both live trading settings and backtesting settings. This ensures consistency and keeps everything in one place per stock.

## Configuration Structure

### Stock Configs (All-in-One)

Each stock's configuration in `config/stocks/{SYMBOL}.yaml` contains:
- **Stock Information**: Symbol, enabled status, description
- **Strategy Parameters**: min_confidence, break_threshold, etc. (for both live and backtest)
- **Risk Management**: Stock-specific risk overrides
- **Backtest Settings**: Date range, timeframe, data source, output settings

## How It Works

1. **Each stock config** contains all settings in one file:
   - Strategy parameters (used for both live and backtest)
   - Backtest-specific settings (date range, timeframe, etc.)
2. **To run backtest**, specify which stocks to test:
   ```bash
   python backtest.py --stocks AAPL MSFT TSLA
   ```
3. **For each stock**, the system:
   - Loads `config/stocks/{SYMBOL}.yaml`
   - Uses strategy parameters from that file
   - Uses backtest settings from that file
   - Runs backtest with those settings

## Timeframe Configuration

Each stock specifies its own timeframe in its config file:

```yaml
# config/stocks/AAPL.yaml
backtest:
  timeframe: "1 hour"  # AAPL uses 1 hour bars
```

```yaml
# config/stocks/TSLA.yaml
backtest:
  timeframe: "15 min"  # TSLA uses 15 min bars (more volatile)
```

Each stock can have a different timeframe optimized for its characteristics.

## Complete Example

### Stock Configs (All Settings)

```yaml
# config/stocks/AAPL.yaml
stock:
  symbol: "AAPL"
  enabled: true

strategy:
  name: "BreakAndRetestStrategy"
  min_confidence: 0.6
  break_threshold: 0.005
  # ... other parameters

backtest:
  enabled: true
  start_date: "2024-01-01"
  end_date: "2024-12-31"
  timeframe: "1 hour"
  data_source: "ib"
  commission_per_trade: 1.0
  slippage_percent: 0.001
  generate_charts: true
  generate_report: true
```

```yaml
# config/stocks/TSLA.yaml
stock:
  symbol: "TSLA"
  enabled: true

strategy:
  name: "BreakAndRetestStrategy"
  min_confidence: 0.7        # Higher for volatile TSLA
  break_threshold: 0.01       # Higher threshold
  # ... other parameters

backtest:
  enabled: true
  start_date: "2024-01-01"
  end_date: "2024-12-31"
  timeframe: "15 min"         # Shorter timeframe for volatile stock
  data_source: "ib"
  commission_per_trade: 1.0
  slippage_percent: 0.001
  generate_charts: true
  generate_report: true
```

## Benefits

1. **Everything in One Place**: All settings for a stock in one file
2. **No Separate Config**: No need for separate backtest.yaml file
3. **Consistency**: Same strategy configs for backtesting and live trading
4. **Stock-Specific**: Each stock has its own optimized parameters and timeframe
5. **Easy Management**: One file per stock, easy to add/remove stocks

## Workflow

1. **Configure each stock** in `config/stocks/{SYMBOL}.yaml`
   - Set strategy parameters (used for both live and backtest)
   - Set backtest settings (date range, timeframe, etc.)
   - Optimize for that stock's characteristics

2. **Run backtest** for specific stocks
   ```bash
   # Backtest single stock
   python backtest.py --stock AAPL
   
   # Backtest multiple stocks
   python backtest.py --stocks AAPL MSFT TSLA
   
   # Backtest all enabled stocks
   python backtest.py --all
   ```

3. **Analyze results**
   - Review performance
   - Check if parameters work well
   - Adjust stock configs if needed

4. **Use in live trading**
   - Same stock configs are used automatically
   - Strategy parameters are shared between backtest and live

## Timeframe Selection

### Considerations

- **Higher timeframes** (4 hour, 1 day):
  - Fewer signals
  - More reliable signals
  - Less noise
  - Good for stable stocks

- **Lower timeframes** (15 min, 1 hour):
  - More signals
  - Faster execution
  - More noise
  - Good for volatile stocks

### Recommendations

- **AAPL, MSFT** (stable): 1 hour or 4 hour
- **TSLA** (volatile): 15 min or 1 hour
- **Test multiple timeframes** by editing the stock config and re-running

## Example: Testing Different Timeframes

To test different timeframes for the same stock:

1. Edit `config/stocks/AAPL.yaml`
2. Change `backtest.timeframe` to desired value
3. Run backtest: `python backtest.py --stock AAPL`
4. Compare results
5. Adjust timeframe and repeat

## Notes

- Stock configs contain all settings in one place
- Timeframe is specified per stock in its config file
- Each stock can have different timeframe optimized for its characteristics
- Strategy parameters are shared between backtesting and live trading
