# Risk Management

## Overview

The risk management system consists of two main components:
- `RiskManager`: Enforces risk limits and daily loss limits
- `PositionSizer`: Calculates position sizes based on risk parameters

## RiskManager

### Purpose
Manages overall risk limits and prevents excessive losses.

### Key Features
- Maximum positions limit
- Daily loss limit enforcement
- Position sizing coordination
- Stop loss management

### Configuration
```yaml
trading:
  max_risk_per_trade: 0.02  # Risk 2% of account per trade
  max_positions: 5  # Maximum number of open positions
  stop_loss_percentage: 0.02  # 2% stop loss
  daily_loss_limit: 0.05  # Stop trading if daily loss exceeds 5%
```

### Methods

#### `can_open_position(current_positions: int) -> bool`
Checks if a new position can be opened based on maximum positions limit.

#### `check_daily_loss_limit(account_equity: float) -> tuple[bool, Optional[str]]`
Checks if daily loss limit has been exceeded. Returns:
- `(True, None)` if trading is allowed
- `(False, reason)` if limit exceeded

#### `calculate_position_size(account_equity, entry_price, stop_loss_price) -> int`
Calculates position size using PositionSizer.

#### `calculate_stop_loss_price(entry_price, is_long) -> float`
Calculates stop loss price based on percentage.

#### `reset_daily_tracking()`
Resets daily tracking at start of new trading day.

## PositionSizer

### Purpose
Calculates the appropriate number of shares to buy based on risk management rules.

### Key Features
- Risk-based position sizing
- Stop loss integration
- Account equity consideration

### Position Sizing Formula

```
Risk Amount = Account Equity × Max Risk Per Trade
Risk Per Share = |Entry Price - Stop Loss Price|
Position Size = Risk Amount / Risk Per Share
```

### Example

Given:
- Account Equity: $100,000
- Max Risk Per Trade: 2% (0.02)
- Entry Price: $150.00
- Stop Loss: 2% below entry = $147.00

Calculation:
- Risk Amount = $100,000 × 0.02 = $2,000
- Risk Per Share = |$150.00 - $147.00| = $3.00
- Position Size = $2,000 / $3.00 = 666 shares

### Methods

#### `calculate_position_size(account_equity, entry_price, stop_loss_price) -> int`
Calculates number of shares to purchase based on risk parameters.

#### `calculate_stop_loss_price(entry_price, is_long) -> float`
Calculates stop loss price based on percentage:
- Long positions: `entry_price × (1 - stop_loss_percentage)`
- Short positions: `entry_price × (1 + stop_loss_percentage)`

## Risk Limits

### Maximum Positions
Prevents over-diversification and over-leveraging:
- Configurable limit (default: 5 positions)
- Checked before opening new positions
- Enforced by RiskManager

### Daily Loss Limit
Prevents catastrophic daily losses:
- Configurable percentage (default: 5%)
- Calculated from starting equity each day
- Stops all trading if exceeded
- Resets at start of new trading day

### Stop Loss
Protects individual positions:
- Configurable percentage (default: 2%)
- Applied to each position
- Can be overridden by strategy
- Managed by RiskManager

## Integration with Trading Bot

1. **Before Opening Position**:
   - Check `can_open_position()`
   - Check `check_daily_loss_limit()`
   - Calculate position size
   - Calculate stop loss price

2. **During Position**:
   - Monitor stop loss levels
   - Track daily P&L
   - Update risk metrics

3. **After Closing Position**:
   - Update daily P&L tracking
   - Check if daily limit still valid

## Best Practices

1. **Start Conservative**: Use small risk percentages (1-2%) initially
2. **Test Limits**: Verify daily loss limits work correctly
3. **Monitor Closely**: Watch daily P&L during initial runs
4. **Adjust Gradually**: Increase limits only after proven performance
5. **Use Stop Losses**: Always set stop losses for protection

## Example Usage

```python
# Initialize risk manager
risk_manager = RiskManager(
    max_risk_per_trade=0.02,
    max_positions=5,
    stop_loss_percentage=0.02,
    daily_loss_limit=0.05
)

# Check if can open position
if risk_manager.can_open_position(current_positions=3):
    # Check daily loss limit
    can_trade, reason = risk_manager.check_daily_loss_limit(account_equity=100000)
    
    if can_trade:
        # Calculate position size
        position_size = risk_manager.calculate_position_size(
            account_equity=100000,
            entry_price=150.00
        )
        
        # Calculate stop loss
        stop_loss = risk_manager.calculate_stop_loss_price(
            entry_price=150.00,
            is_long=True
        )
```

## Risk Metrics

The system tracks:
- **Per-trade risk**: Percentage of account risked per trade
- **Daily P&L**: Realized and unrealized P&L for the day
- **Position count**: Number of open positions
- **Portfolio risk**: Overall exposure

## Error Handling

- Invalid risk parameters: Defaults to safe values
- Division by zero: Handles edge cases in position sizing
- Negative equity: Prevents trading with negative balance
- Missing data: Graceful degradation
