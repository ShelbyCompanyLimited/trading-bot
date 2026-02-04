# Portfolio Management

## Overview

The `PortfolioManager` class tracks all open positions, calculates P&L, and maintains portfolio statistics.

## Key Features

- Real-time position tracking
- Unrealized P&L calculation
- Realized P&L tracking
- Position updates from market data
- Portfolio statistics

## Position Data Structure

```python
@dataclass
class Position:
    symbol: str
    quantity: int
    avg_price: float
    current_price: float
    unrealized_pnl: float
    unrealized_pnl_percent: float
    stop_loss_price: Optional[float]
```

## Core Methods

### Adding Positions

#### `add_position(symbol, quantity, avg_price, stop_loss_price)`
Adds a new position or updates existing position:
- If position exists: Averages in new shares
- If new: Creates new position entry
- Updates average price for averaged positions

### Removing Positions

#### `remove_position(symbol, exit_price) -> Optional[float]`
Removes position and calculates realized P&L:
- Calculates: `(exit_price - avg_price) × quantity`
- Records in closed positions history
- Updates total realized P&L
- Returns realized P&L amount

### Updating Prices

#### `update_position_price(symbol, current_price)`
Updates current price and recalculates unrealized P&L:
- Updates `current_price`
- Recalculates `unrealized_pnl`
- Recalculates `unrealized_pnl_percent`

### Position Queries

#### `get_position(symbol) -> Optional[Position]`
Returns position object for symbol or None.

#### `has_position(symbol) -> bool`
Checks if position exists for symbol.

#### `get_all_positions() -> Dict[str, Position]`
Returns dictionary of all open positions.

#### `get_position_count() -> int`
Returns number of open positions.

## P&L Calculations

### Unrealized P&L
Calculated for each position:
```
Unrealized P&L = (Current Price - Avg Price) × Quantity
Unrealized P&L % = ((Current Price - Avg Price) / Avg Price) × 100
```

### Realized P&L
Calculated when position is closed:
```
Realized P&L = (Exit Price - Avg Price) × Quantity
```

### Total P&L
```
Total P&L = Total Realized P&L + Total Unrealized P&L
```

## Portfolio Statistics

### `get_portfolio_summary() -> Dict`
Returns comprehensive portfolio statistics:
```python
{
    "open_positions": 3,
    "total_unrealized_pnl": 1250.50,
    "total_realized_pnl": 500.00,
    "total_pnl": 1750.50,
    "positions": {
        "AAPL": {
            "quantity": 100,
            "avg_price": 150.00,
            "current_price": 152.50,
            "unrealized_pnl": 250.00,
            "unrealized_pnl_percent": 1.67,
            "stop_loss_price": 147.00
        },
        # ... more positions
    }
}
```

## Position Averaging

When adding to an existing position:
- New average price = `(old_cost + new_cost) / total_quantity`
- Preserves stop loss if already set
- Updates quantity to total

Example:
- Existing: 100 shares @ $150.00
- Adding: 50 shares @ $148.00
- New: 150 shares @ $149.33

## Closed Positions History

All closed positions are recorded with:
- Symbol
- Entry price
- Exit price
- Quantity
- Realized P&L
- Realized P&L percentage
- Close timestamp

## Integration with Trading Bot

1. **On Order Fill**:
   - Add position with fill price
   - Set stop loss if provided

2. **On Price Update**:
   - Update position price
   - Recalculate unrealized P&L
   - Check stop loss triggers

3. **On Exit Signal**:
   - Remove position with exit price
   - Record realized P&L
   - Update statistics

4. **On Stop Loss Trigger**:
   - Remove position at stop price
   - Record realized P&L (likely negative)

## Example Usage

```python
# Initialize portfolio manager
portfolio = PortfolioManager()

# Add position
portfolio.add_position(
    symbol="AAPL",
    quantity=100,
    avg_price=150.00,
    stop_loss_price=147.00
)

# Update price
portfolio.update_position_price("AAPL", 152.50)

# Check position
position = portfolio.get_position("AAPL")
print(f"Unrealized P&L: ${position.unrealized_pnl:.2f}")

# Close position
realized_pnl = portfolio.remove_position("AAPL", exit_price=155.00)
print(f"Realized P&L: ${realized_pnl:.2f}")

# Get summary
summary = portfolio.get_portfolio_summary()
print(f"Total P&L: ${summary['total_pnl']:.2f}")
```

## Thread Safety

- All operations are synchronous
- Designed for single-threaded async event loop
- No locking required for async usage

## Performance Considerations

- In-memory storage (fast access)
- O(1) position lookups by symbol
- Minimal overhead for price updates
- Efficient P&L calculations

## Error Handling

- Missing positions: Returns None gracefully
- Invalid prices: Validates before calculations
- Division by zero: Handles edge cases
- Negative quantities: Prevents invalid positions
