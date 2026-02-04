# Break and Retest Strategy - Detailed Guide

## Overview

The Break and Retest strategy is a technical analysis approach that trades on the principle that broken support/resistance levels often act as the opposite level after a breakout. This strategy is particularly effective in trending markets and can be applied to various timeframes.

## Core Principles

### 1. Support and Resistance Levels

Support and resistance levels are price zones where:
- **Support**: Buying pressure is strong enough to prevent further price decline
- **Resistance**: Selling pressure is strong enough to prevent further price advance

These levels are identified by:
- Multiple price touches at similar levels
- Price reversals at these levels
- Volume concentration at these levels

### 2. Breakout Mechanics

A breakout occurs when:
- Price moves decisively through a support or resistance level
- Volume confirms the move (typically 1.2x+ average volume)
- Price closes beyond the level (not just a wick)
- The move exceeds a minimum threshold (e.g., 0.5%)

### 3. Retest and Role Reversal

After a breakout:
- The broken level often becomes the opposite level
- Broken resistance becomes new support
- Broken support becomes new resistance
- Price frequently retests the broken level
- Successful retest confirms the breakout

## Strategy Parameters

### Lookback Period
- **Purpose**: How far back to scan for support/resistance levels
- **Typical Range**: 20-100 bars
- **Trade-off**: Longer = more levels but slower adaptation

### Minimum Touch Count
- **Purpose**: Minimum number of touches to qualify as valid level
- **Typical Range**: 2-4 touches
- **Trade-off**: Higher = stronger levels but fewer opportunities

### Break Threshold
- **Purpose**: Minimum break distance to confirm breakout
- **Typical Range**: 0.3%-1.0% of price
- **Trade-off**: Higher = fewer false breakouts but fewer signals

### Retest Threshold
- **Purpose**: Zone around broken level for retest confirmation
- **Typical Range**: 0.5%-2.0% of level
- **Trade-off**: Wider = more retest opportunities but less precise

### Volume Multiplier
- **Purpose**: Volume requirement for breakout confirmation
- **Typical Range**: 1.2x-2.0x average volume
- **Trade-off**: Higher = stronger signals but fewer opportunities

### Minimum Confidence
- **Purpose**: Minimum confidence score to enter trade
- **Typical Range**: 0.5-0.8 (50%-80%)
- **Trade-off**: Higher = better quality but fewer trades

## Signal Generation Process

### Step 1: Level Identification

```python
def identify_levels(data, lookback_period, min_touch_count):
    # Find local highs (resistance) and lows (support)
    # Cluster nearby levels within tolerance
    # Filter by minimum touch count
    # Calculate level strength
    return support_levels, resistance_levels
```

**Algorithm**:
1. Scan price history for local extrema
2. Group nearby levels (within 0.5% tolerance)
3. Count touches per level
4. Filter by minimum touch count
5. Rank by strength (touches, recency, volume)

### Step 2: Breakout Detection

```python
def detect_breakout(price, level, break_threshold, volume, avg_volume):
    # Check if price broke through level
    # Verify break exceeds threshold
    # Confirm with volume
    return breakout_info or None
```

**Conditions**:
- Price breaks above resistance or below support
- Break distance > break_threshold
- Volume > avg_volume * volume_multiplier
- Price closes beyond level (not just intraday break)

### Step 3: Retest Confirmation

```python
def check_retest(price, broken_level, retest_threshold):
    # Check if price is in retest zone
    # Verify bounce off level
    # Confirm rejection of level
    return retest_confirmed
```

**Conditions**:
- Price returns to within retest_threshold of broken level
- Price bounces off level (rejection)
- Volume may decrease on retest
- Price action shows level holding

### Step 4: Entry Signal

```python
def generate_entry_signal(breakout, retest, confidence):
    if confidence >= min_confidence:
        return {
            'action': 'BUY' or 'SELL',
            'entry_price': current_price,
            'stop_loss': calculate_stop_loss(),
            'target': calculate_target(),
            'confidence': confidence
        }
```

**Entry Conditions**:
- Breakout confirmed
- Retest confirmed
- Confidence >= minimum
- Risk management checks pass

## Confidence Score Calculation

The confidence score combines multiple factors:

### Level Strength (0.0-0.3)
- Number of touches: More touches = higher score
- Recency: Recent touches weighted more
- Volume: Higher volume at level = stronger

### Breakout Quality (0.0-0.3)
- Break distance: Larger break = higher score
- Volume: Higher volume = stronger breakout
- Price action: Clean break = better score

### Retest Quality (0.0-0.2)
- Bounce strength: Strong bounce = higher score
- Volume: Decreasing volume on retest = good
- Price action: Clear rejection = better

### Market Context (0.0-0.2)
- Trend alignment: Breakout with trend = better
- Volatility: Moderate volatility = optimal
- Timeframe: Higher timeframe = more reliable

## Trade Management

### Entry
- Enter on bounce from retest level
- Use limit order near retest level for better fill
- Or market order if retest already occurred

### Stop Loss
- Place below broken support (for long) or above broken resistance (for short)
- Add buffer (0.2-0.5%) to avoid stop hunting
- Adjust based on volatility (ATR)

### Profit Target
- Option 1: Breakout distance (1:1 risk/reward minimum)
- Option 2: Next support/resistance level
- Option 3: Fibonacci extension levels
- Option 4: Trailing stop after initial target

### Exit Conditions
1. **Profit Target**: Take profit at target
2. **Stop Loss**: Exit on stop loss trigger
3. **Opposite Breakout**: Exit if price breaks opposite level
4. **Time Stop**: Exit after X bars if no progress

## Risk Management Integration

### Position Sizing
- Use risk-based position sizing
- Risk 1-2% of account per trade
- Adjust for confidence score (higher confidence = larger position)

### Stop Loss Integration
- Strategy provides stop loss price
- RiskManager validates stop loss
- Stop loss placed immediately on entry

### Portfolio Limits
- Maximum positions limit applies
- Daily loss limit applies
- Strategy respects all risk limits

## Advantages

1. **Clear Entry Points**: Retest provides clear entry signal
2. **Defined Risk**: Stop loss clearly defined at broken level
3. **Trend Following**: Works well in trending markets
4. **High Probability**: Retest confirmation increases win rate
5. **Scalable**: Works on multiple timeframes

## Disadvantages

1. **False Breakouts**: Not all breakouts succeed
2. **Whipsaws**: Price may break and immediately reverse
3. **Missed Entries**: Retest may not occur or may be too fast
4. **Range Markets**: Less effective in choppy markets
5. **Level Identification**: Requires accurate level identification

## Optimization Tips

### Parameter Tuning
- Start with default parameters
- Backtest on historical data
- Optimize one parameter at a time
- Use walk-forward analysis
- Avoid over-optimization

### Market Conditions
- Best in trending markets
- Less effective in ranging markets
- Works on multiple timeframes
- Higher timeframes = more reliable

### Level Quality
- Focus on levels with 3+ touches
- Prefer recent levels
- Consider volume at level
- Avoid levels too close together

### Entry Timing
- Wait for retest confirmation
- Don't chase breakouts
- Use limit orders for better fills
- Consider multiple timeframe confirmation

## Example Trades

### Example 1: Bullish Break and Retest

**Setup**:
- Resistance at $100 (touched 4 times)
- Price breaks above $100.50 with 1.5x volume
- Price retests $100.20 and bounces

**Entry**: $100.25
**Stop Loss**: $99.50 (below support)
**Target**: $101.50 (breakout distance)
**Risk/Reward**: 1:1.67

**Result**: Price reaches target, +1.25% gain

### Example 2: Bearish Break and Retest

**Setup**:
- Support at $50 (touched 3 times)
- Price breaks below $49.75 with 1.8x volume
- Price retests $50.10 and rejects

**Entry**: $50.05
**Stop Loss**: $50.50 (above resistance)
**Target**: $49.25 (breakout distance)
**Risk/Reward**: 1:1.78

**Result**: Stop loss triggered, -0.9% loss

## Backtesting Considerations

### Data Requirements
- Sufficient historical data (100+ bars minimum)
- Accurate OHLCV data
- No gaps in data
- Appropriate timeframe for strategy

### Metrics to Track
- Win rate
- Average win/loss
- Risk/reward ratio
- Maximum drawdown
- Sharpe ratio
- Number of trades

### Common Issues
- Overfitting to historical data
- Survivorship bias
- Slippage and commissions not accounted for
- Data quality issues
- Look-ahead bias

## Future Enhancements

Potential improvements to the strategy:
- Multi-timeframe confirmation
- Volume profile integration
- Order flow analysis
- Machine learning for level identification
- Dynamic parameter adjustment
- Market regime detection
