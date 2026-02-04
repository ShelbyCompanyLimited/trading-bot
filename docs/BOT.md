# Trading Bot Implementation

## Overview

The `TradingBot` class (`src/bot.py`) is the main orchestrator that coordinates all components of the trading system.

## Architecture

```
TradingBot
├── IBClient (API connection)
├── Strategy (signal generation)
├── RiskManager (risk limits)
└── PortfolioManager (position tracking)
```

## Class Structure

```python
class TradingBot:
    def __init__(self, config: Dict)
    async def initialize(self) -> bool
    async def start(self)
    async def stop(self)
    async def run(self)
    
    async def _evaluate_strategy(self)
    async def _process_signals(self, signals: Dict)
    async def _check_exit_signals(self)
    async def _update_positions(self)
    async def _monitor_stop_losses(self)
```

## Initialization

### Setup Process

1. **Load Configuration**: Load from `config.yaml` and `.env`
2. **Initialize Logger**: Set up logging with file and console handlers
3. **Create IBClient**: Initialize connection to Interactive Brokers
4. **Create Strategy**: Instantiate strategy based on config
5. **Create RiskManager**: Initialize with risk parameters
6. **Create PortfolioManager**: Initialize portfolio tracking
7. **Connect to IB**: Establish API connection

### Error Handling

- Configuration errors: Log and exit gracefully
- Connection failures: Retry with exponential backoff
- Component initialization: Validate each component

## Main Event Loop

### Execution Flow

1. **Connect to IB**: Establish API connection
2. **Subscribe to Market Data**: Subscribe to symbols in strategy
3. **Load Historical Data**: Get historical data for strategy
4. **Enter Main Loop**:
   - Evaluate strategy for each symbol
   - Process buy/sell signals
   - Check exit conditions
   - Update positions
   - Monitor stop losses
   - Sleep for update interval

### Update Cycle

```python
while running:
    # Evaluate strategy
    await self._evaluate_strategy()
    
    # Process new signals
    await self._process_signals(signals)
    
    # Check exit signals
    await self._check_exit_signals()
    
    # Update positions
    await self._update_positions()
    
    # Monitor stop losses
    await self._monitor_stop_losses()
    
    # Wait for next cycle
    await asyncio.sleep(update_interval)
```

## Strategy Evaluation

### Process

1. **Get Market Data**: Retrieve current and historical data for each symbol
2. **Generate Signals**: Call strategy's `generate_signals()` method
3. **Filter Signals**: Apply confidence thresholds
4. **Return Signals**: Return actionable signals

### Signal Processing

For each signal:
- **BUY**: Check risk limits → Calculate position size → Place order
- **SELL**: Check if position exists → Place exit order
- **HOLD**: No action, continue monitoring

## Order Execution

### Buy Orders

1. **Risk Check**: Verify can open position and daily limits
2. **Position Sizing**: Calculate shares based on risk
3. **Stop Loss**: Calculate stop loss price
4. **Place Order**: Submit order through IBClient
5. **Track Order**: Monitor order status
6. **Update Portfolio**: Add position on fill

### Sell Orders

1. **Verify Position**: Check position exists
2. **Place Order**: Submit exit order
3. **Track Order**: Monitor order status
4. **Update Portfolio**: Remove position on fill

### Order Types

- **Market Orders**: Immediate execution
- **Limit Orders**: Execute at specified price
- **Stop Loss Orders**: Triggered at stop price

## Position Management

### Monitoring

- **Price Updates**: Update positions with latest prices
- **P&L Calculation**: Calculate unrealized P&L
- **Stop Loss Monitoring**: Check if stop loss triggered
- **Exit Signals**: Evaluate strategy exit conditions

### Stop Loss Management

1. **Check Prices**: Compare current price to stop loss
2. **Trigger Stop**: If price hit, place exit order
3. **Update Portfolio**: Remove position
4. **Log Event**: Record stop loss execution

## Risk Management Integration

### Pre-Trade Checks

- Maximum positions limit
- Daily loss limit
- Account equity validation
- Position size limits

### During Trade

- Stop loss monitoring
- Daily P&L tracking
- Position size validation

### Post-Trade

- Update daily P&L
- Check daily loss limit
- Update position count

## Error Handling

### Connection Errors

- Automatic reconnection
- Exponential backoff
- Connection status monitoring
- Graceful degradation

### Order Errors

- Error logging
- Retry logic for transient errors
- Order rejection handling
- Position reconciliation

### Data Errors

- Missing data handling
- Invalid price detection
- Data gap management
- Historical data validation

## Logging

### Events Logged

- Connection status changes
- Strategy evaluations
- Signal generation
- Order placement and fills
- Position updates
- Error conditions
- Risk limit violations

### Log Levels

- **DEBUG**: Detailed execution flow
- **INFO**: Normal operations
- **WARNING**: Non-critical issues
- **ERROR**: Error conditions
- **CRITICAL**: System failures

## State Management

### Running State

- `running`: Boolean flag for main loop
- `connected`: IB connection status
- `trading_enabled`: Whether trading is active

### Position State

- Tracked in PortfolioManager
- Synced with IB positions
- Updated on order fills

## Shutdown Process

1. **Stop Main Loop**: Set `running = False`
2. **Cancel Pending Orders**: Cancel any open orders
3. **Close Positions** (optional): Exit all positions
4. **Disconnect from IB**: Close API connection
5. **Save State** (optional): Persist portfolio state
6. **Close Loggers**: Flush and close log files

## Example Usage

```python
# Initialize bot
config = load_config("config/config.yaml")
bot = TradingBot(config)

# Start bot
await bot.start()

# Bot runs until stopped
# (In production, this would run indefinitely)

# Stop bot
await bot.stop()
```

## Performance Considerations

### Optimization

- Async operations for non-blocking I/O
- Efficient data structures for position tracking
- Minimal strategy evaluation overhead
- Cached market data where possible

### Resource Usage

- Memory: Positions, market data, logs
- CPU: Strategy calculations, signal processing
- Network: IB API communication
- Disk: Log files

## Testing

### Unit Tests

- Component initialization
- Signal processing logic
- Risk limit checks
- Position calculations

### Integration Tests

- IB connection
- Order execution
- Position tracking
- Error handling

### Paper Trading

- Test with live market data
- Validate order execution
- Monitor performance
- Verify risk limits

## Monitoring

### Key Metrics

- Connection status
- Open positions count
- Total P&L (realized + unrealized)
- Daily P&L
- Order fill rate
- Strategy signal frequency

### Alerts

- Connection failures
- Daily loss limit reached
- Order rejections
- Position limit exceeded
- Stop loss triggers
