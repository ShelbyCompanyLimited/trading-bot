# Interactive Brokers Trading Bot

A production-ready Python trading bot that connects to Interactive Brokers via their API. The bot uses `ib_async` (modern async-based library) for API communication and includes core trading functionality with a flexible strategy framework.

## Project Structure

```
trading-bot/
├── src/
│   ├── bot.py                 # Main TradingBot class
│   ├── ib_client.py           # IB API connection wrapper
│   ├── strategy/
│   │   ├── base_strategy.py        # Base strategy interface
│   │   └── break_retest_strategy.py # Break and retest strategy
│   ├── risk/
│   │   ├── risk_manager.py    # Risk management logic
│   │   └── position_sizer.py  # Position sizing calculations
│   ├── portfolio/
│   │   └── portfolio_manager.py # Portfolio tracking
│   └── utils/
│       ├── logger.py          # Logging setup
│       └── config.py          # Configuration loader
├── config/
│   ├── ib_connection.yaml     # IB connection and global settings
│   └── stocks/
│       ├── AAPL.yaml          # Apple stock configuration
│       ├── TSLA.yaml          # Tesla stock configuration
│       └── ...                # One file per stock
├── logs/                      # Log files directory
├── requirements.txt           # Python dependencies
├── .env.example              # Environment variables template
├── .gitignore
└── README.md                  # This file
```

## Features

- **Connection Management**: Automatic reconnection to IB Gateway/TWS
- **Order Execution**: Market and limit orders with proper error handling
- **Real-time Data**: Streaming market data for decision making
- **Risk Management**: Position sizing, stop losses, and risk limits
- **Portfolio Tracking**: Real-time position and P&L monitoring
- **Backtesting**: Test strategies on historical data with visualizations
- **Performance Analysis**: Comprehensive metrics and trade analysis
- **Logging**: Comprehensive logging for debugging and audit trail
- **Strategy Framework**: Easy to extend with custom strategies

## Setup

1. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

2. Set up IB Gateway or TWS with API enabled:
   - Enable API connections in TWS/Gateway settings
   - Set API port (default: 7497 for paper trading)
   - Enable "Download open orders on connection"

3. Configure the bot:
   - Copy IB connection config:
     ```bash
     cp config/ib_connection.yaml.example config/ib_connection.yaml
     ```
   - Create stock configurations:
     ```bash
     mkdir -p config/stocks
     cp config/stocks/AAPL.yaml.example config/stocks/AAPL.yaml
     # Add more stocks as needed
     ```
   - Edit `config/ib_connection.yaml` with your IB connection settings
   - Edit each stock config in `config/stocks/` to customize strategy parameters
   - Copy `.env.example` to `.env` for environment variables (optional)

4. Run the bot:
   ```bash
   python main.py
   ```

5. Run backtesting (optional):
   ```bash
   # Edit stock config files to set backtest settings
   # Each stock config (config/stocks/{SYMBOL}.yaml) contains:
   # - Strategy parameters (for both live and backtest)
   # - Backtest settings (date range, timeframe, etc.)
   
   # Run backtest for specific stock
   python backtest.py --stock AAPL
   
   # Run backtest for multiple stocks
   python backtest.py --stocks AAPL MSFT TSLA
   ```
   
   **Note**: All backtest settings are in each stock's config file. No separate backtest config needed.

## Configuration

See `docs/CONFIGURATION.md` for detailed configuration options.

## Strategy

The bot currently implements a **Break and Retest** strategy that:
- Identifies support/resistance levels from price history
- Trades on breakouts with retest confirmation
- Uses volume analysis for signal validation

See `docs/STRATEGY.md` for the strategy framework and `docs/BREAK_RETEST_STRATEGY.md` for detailed strategy documentation.

The architecture is designed to support multiple strategies in the future, though currently one strategy is used per bot instance.

## Documentation

- [Architecture Overview](docs/ARCHITECTURE.md)
- [IB Client Implementation](docs/IB_CLIENT.md)
- [Risk Management](docs/RISK_MANAGEMENT.md)
- [Portfolio Management](docs/PORTFOLIO.md)
- [Strategy Framework](docs/STRATEGY.md)
- [Break and Retest Strategy Guide](docs/BREAK_RETEST_STRATEGY.md)
- [Confidence Score Calculation](docs/CONFIDENCE_CALCULATION.md)
- [Backtesting System](docs/BACKTESTING.md)
- [Backtest Visualizations](docs/BACKTEST_VISUALIZATIONS.md)
- [Backtest Configuration Guide](docs/BACKTEST_CONFIG_GUIDE.md)
- [Configuration Guide](docs/CONFIGURATION.md)

## Important Notes

- **Always test with paper trading first!**
- Set `enable_trading: false` in config until you're ready
- Monitor logs carefully during initial runs
- Understand the risks before using real money

## License

MIT License
