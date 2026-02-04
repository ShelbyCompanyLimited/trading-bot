# Strategy Framework

## Overview

The strategy framework provides a flexible interface for implementing trading strategies. All strategies inherit from `BaseStrategy` and implement required methods.

## BaseStrategy

### Abstract Interface

All strategies must implement:

#### `generate_signals(symbol, market_data) -> Dict`
Generates trading signals for a symbol:
```python
{
    'action': 'BUY', 'SELL', or 'HOLD',
    'confidence': float (0.0 to 1.0),
    'entry_price': float,
    'stop_loss': float or None,
    'metadata': dict
}
```

#### `should_buy(symbol, market_data) -> bool`
Determines if we should buy a symbol.

#### `should_sell(symbol, market_data, current_price) -> bool`
Determines if we should sell a symbol.

### Base Methods

#### `update_market_data(symbol, data)`
Updates market data for a symbol (pandas DataFrame).

#### `get_market_data(symbol) -> Optional[pd.DataFrame]`
Retrieves stored market data for a symbol.

## BreakAndRetestStrategy

### Overview
Break and retest strategy that identifies support/resistance levels and trades on breakouts with retest confirmation:
- Identifies key support/resistance levels from price history
- Waits for price to break through these levels
- Confirms entry on retest of the broken level
- Uses volume confirmation for stronger signals

### Configuration
```yaml
strategy:
  name: "BreakAndRetestStrategy"
  symbols:
    - "AAPL"
    - "MSFT"
    - "GOOGL"
  lookback_period: 50        # Bars to look back for support/resistance
  min_touch_count: 2         # Minimum touches to qualify as S/R level
  break_threshold: 0.005     # 0.5% break above/below level required
  retest_threshold: 0.01     # 1% retest zone around broken level
  volume_multiplier: 1.2     # Volume must be 1.2x average for confirmation
  min_confidence: 0.6        # Minimum confidence to enter trade
```

### Signal Generation Logic

1. **Identify Support/Resistance Levels**:
   - Scan price history for local highs and lows
   - Identify levels where price touched multiple times (min_touch_count)
   - Store these levels as potential support/resistance

2. **Detect Breakouts**:
   - Monitor when price breaks above resistance or below support
   - Break must exceed threshold (e.g., 0.5% above resistance)
   - Volume should be above average for confirmation

3. **Wait for Retest**:
   - After breakout, wait for price to retest the broken level
   - Retest zone is within threshold (e.g., 1% of broken level)
   - Price should bounce off the level (support becomes resistance, vice versa)

4. **Generate Entry Signal**:
   - BUY on retest of broken resistance (now support)
   - SELL on retest of broken support (now resistance)
   - Confidence based on:
     - Number of previous touches at level
     - Volume on breakout
     - Strength of retest bounce
     - Distance from level

5. **Exit Logic**:
   - Exit on opposite break (break of new support/resistance)
   - Exit on stop loss trigger
   - Exit on profit target (if configured)

### Key Concepts

**Support Level**: Price level where buying pressure prevents further decline
**Resistance Level**: Price level where selling pressure prevents further rise
**Breakout**: Price moves decisively through support/resistance
**Retest**: Price returns to test the broken level
**Confirmation**: Volume and price action validate the breakout

### Strategy Workflow

```
1. Identify Support/Resistance Levels
   └─> Scan price history for local highs/lows
   └─> Cluster nearby price levels
   └─> Validate minimum touch count

2. Monitor for Breakouts
   └─> Watch for price breaking above resistance or below support
   └─> Confirm with volume analysis
   └─> Validate breakout strength

3. Wait for Retest
   └─> Price returns to broken level
   └─> Price bounces off level (confirmation)
   └─> Volume analysis on retest

4. Enter Trade
   └─> Enter on bounce from retest
   └─> Set stop loss below/above retest level
   └─> Set profit target based on breakout distance

5. Manage Position
   └─> Monitor for stop loss trigger
   └─> Take profit at target
   └─> Exit on opposite breakout
```

### Example Scenario: Bullish Break and Retest

1. **Initial State**: 
   - Resistance level at $150 (touched 3 times)
   - Price trading below $150

2. **Breakout**:
   - Price breaks above $150 with high volume (1.5x average)
   - Breakout exceeds threshold ($150.75 = 0.5% above)
   - Price closes above resistance

3. **Retest**:
   - Price pulls back to $150.50 (within 1% retest zone)
   - Price bounces off $150 level (now acting as support)
   - Volume decreases on retest

4. **Entry**:
   - Enter long at $150.50 on bounce
   - Stop loss at $149.50 (below support)
   - Target at $152.50 (breakout distance)

5. **Exit**:
   - Price reaches target → Take profit
   - OR price breaks below $149.50 → Stop loss
   - OR price breaks below new support → Exit signal

### Market Data Requirements

DataFrame must have columns:
- `date` or `datetime`: Timestamp
- `open`: Opening price
- `high`: High price
- `low`: Low price
- `close`: Closing price
- `volume`: Trading volume

## Creating Custom Strategies

### Step 1: Inherit from BaseStrategy

```python
from src.strategy.base_strategy import BaseStrategy

class MyStrategy(BaseStrategy):
    def __init__(self, config: Dict):
        super().__init__(config)
        # Initialize your strategy parameters
```

### Step 2: Implement Required Methods

```python
def generate_signals(self, symbol: str, market_data: pd.DataFrame) -> Dict:
    # Your signal generation logic
    return {
        'action': 'BUY',
        'confidence': 0.8,
        'entry_price': current_price,
        'stop_loss': stop_price,
        'metadata': {}
    }

def should_buy(self, symbol: str, market_data: pd.DataFrame) -> bool:
    signal = self.generate_signals(symbol, market_data)
    return signal['action'] == 'BUY' and signal['confidence'] > 0.5

def should_sell(self, symbol: str, market_data: pd.DataFrame, current_price: float) -> bool:
    signal = self.generate_signals(symbol, market_data)
    return signal['action'] == 'SELL' and signal['confidence'] > 0.5
```

### Step 3: Add to Configuration

```yaml
strategy:
  name: "MyStrategy"
  symbols:
    - "AAPL"
  my_custom_param: 42
```

### Step 4: Register in Bot

The bot automatically loads strategies based on configuration. The architecture supports multiple strategies, though currently one strategy is used. Future extensions can support:
- Multiple strategies running simultaneously
- Strategy selection based on market conditions
- Strategy weighting and portfolio allocation

## Strategy Best Practices

### 1. Data Validation
Always check data availability:
```python
if len(market_data) < required_period:
    return {'action': 'HOLD', 'confidence': 0.0, ...}
```

### 2. Confidence Scores
Use confidence to filter weak signals:
- High confidence (>0.7): Strong signal
- Medium confidence (0.4-0.7): Moderate signal
- Low confidence (<0.4): Weak signal, consider filtering

### 3. Stop Loss Integration
Always provide stop loss prices:
```python
stop_loss = entry_price * (1 - stop_loss_percentage)
```

### 4. Metadata
Include useful information in metadata:
```python
'metadata': {
    'indicator_value': rsi_value,
    'trend': 'bullish',
    'volume': current_volume
}
```

### 5. Error Handling
Handle edge cases gracefully:
- Missing data
- Division by zero
- Invalid prices
- Insufficient history

## Break and Retest Strategy Implementation Details

### Support/Resistance Identification

The strategy identifies support and resistance levels by:

1. **Finding Local Extrema**:
   - Scan price history for local highs (resistance) and lows (support)
   - Use rolling windows to identify peaks and troughs
   - Filter by minimum touch count requirement

2. **Level Clustering**:
   - Group nearby price levels together
   - Use tolerance zone (e.g., 0.5% of price) to cluster touches
   - Only levels with minimum touch count are considered valid

3. **Level Strength**:
   - More touches = stronger level
   - Recent touches weighted more heavily
   - Volume at level affects strength

### Breakout Detection

1. **Price Break**:
   - Price must break above resistance or below support
   - Break must exceed threshold (e.g., 0.5%)
   - Break should be decisive (not just a wick)

2. **Volume Confirmation**:
   - Volume on breakout should exceed average
   - Volume multiplier (e.g., 1.2x) confirms strength
   - Low volume breakouts are less reliable

3. **Breakout Validation**:
   - Price should close above/below level
   - Multiple bars confirming direction
   - No immediate reversal

### Retest Logic

1. **Retest Zone**:
   - After breakout, price often retests the broken level
   - Retest zone is within threshold (e.g., 1% of level)
   - Price should approach but not break back through

2. **Retest Confirmation**:
   - Price bounces off the level (support becomes resistance, vice versa)
   - Volume may decrease on retest
   - Price action shows rejection of the level

3. **Entry Signal**:
   - Enter on bounce from retest
   - Stop loss below/above the retest level
   - Target based on breakout distance or next level

### Confidence Calculation

Confidence score (0.0-1.0) based on:
- **Level Strength**: Number of previous touches (0.0-0.3)
- **Breakout Quality**: Volume and price action (0.0-0.3)
- **Retest Quality**: Bounce strength and volume (0.0-0.2)
- **Market Context**: Trend alignment, volatility (0.0-0.2)

### Example Implementation Structure

```python
class BreakAndRetestStrategy(BaseStrategy):
    def __init__(self, config: Dict):
        super().__init__(config)
        self.lookback_period = config.get("lookback_period", 50)
        self.min_touch_count = config.get("min_touch_count", 2)
        self.break_threshold = config.get("break_threshold", 0.005)
        self.retest_threshold = config.get("retest_threshold", 0.01)
        self.volume_multiplier = config.get("volume_multiplier", 1.2)
        self.min_confidence = config.get("min_confidence", 0.6)
        
        # Track identified levels and breakouts
        self.support_levels = {}
        self.resistance_levels = {}
        self.active_breakouts = {}
    
    def _identify_levels(self, data: pd.DataFrame) -> tuple:
        # Identify support and resistance levels
        # Return (support_levels, resistance_levels)
        pass
    
    def _detect_breakout(self, symbol: str, data: pd.DataFrame) -> Optional[Dict]:
        # Detect if price has broken a level
        # Return breakout information
        pass
    
    def _check_retest(self, symbol: str, data: pd.DataFrame, breakout: Dict) -> bool:
        # Check if price is retesting broken level
        # Return True if retest confirmed
        pass
    
    def generate_signals(self, symbol: str, market_data: pd.DataFrame) -> Dict:
        # Main signal generation logic
        # 1. Identify levels
        # 2. Detect breakouts
        # 3. Check for retests
        # 4. Generate entry signals
        pass
```

## Future Strategy Extensions

The architecture is designed to support multiple strategies in the future:

### Multi-Strategy Support
- Run multiple strategies simultaneously
- Allocate capital across strategies
- Strategy performance tracking

### Strategy Selection
- Market condition-based strategy selection
- Dynamic strategy weighting
- Strategy rotation based on performance

### Strategy Combination
- Combine signals from multiple strategies
- Weighted signal aggregation
- Conflict resolution logic

## Strategy Testing

### Backtesting
- Use historical data to test strategies
- Validate signal generation
- Check P&L performance
- Optimize parameters

### Paper Trading
- Test with live market data
- No real money at risk
- Validate execution logic
- Monitor performance

### Production
- Start with small position sizes
- Monitor closely
- Adjust parameters as needed
- Keep detailed logs

## Common Patterns

### Trend Following
- Moving averages
- MACD
- ADX

### Mean Reversion
- RSI
- Bollinger Bands
- Stochastic

### Breakout
- Support/Resistance
- Volume analysis
- Price patterns

### Multi-Timeframe
- Combine signals from different timeframes
- Higher timeframe for trend
- Lower timeframe for entry
