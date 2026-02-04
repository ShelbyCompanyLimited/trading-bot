# Confidence Score Calculation

## Overview

The confidence score is a value between 0.0 and 1.0 that represents how strong a trading signal is. The strategy calculates this score for each potential trade, and only enters trades where the confidence score meets or exceeds the `min_confidence` threshold.

## Key Concepts

### `min_confidence` (Configuration Parameter)
- **What it is**: A threshold value you set in the configuration (e.g., 0.6 = 60%)
- **Purpose**: Filters out weak signals - only trades with confidence >= min_confidence are executed
- **Range**: 0.0 to 1.0 (0% to 100%)
- **Typical values**: 0.5-0.8 (50%-80%)

### Confidence Score (Calculated Value)
- **What it is**: A calculated score based on multiple factors
- **Purpose**: Measures the quality and strength of a trading signal
- **Range**: 0.0 to 1.0 (0% to 100%)
- **Calculation**: Combines multiple weighted factors

## Confidence Score Calculation Formula

The confidence score is calculated as the sum of four weighted components:

```
Confidence Score = Level Strength + Breakout Quality + Retest Quality + Market Context
```

Where each component has a maximum value:
- Level Strength: 0.0 to 0.3 (30% of total)
- Breakout Quality: 0.0 to 0.3 (30% of total)
- Retest Quality: 0.0 to 0.2 (20% of total)
- Market Context: 0.0 to 0.2 (20% of total)

**Maximum possible confidence**: 1.0 (100%)

## Component Calculations

### 1. Level Strength (0.0 - 0.3)

Measures how strong the support/resistance level is:

```python
def calculate_level_strength(level):
    # Factor 1: Number of touches (0.0 - 0.15)
    touch_score = min(level.touch_count / 5.0, 1.0) * 0.15
    # 5+ touches = maximum score
    
    # Factor 2: Recency of touches (0.0 - 0.10)
    # More recent touches weighted higher
    recent_touches = count_recent_touches(level, last_10_bars)
    recency_score = min(recent_touches / 3.0, 1.0) * 0.10
    
    # Factor 3: Volume at level (0.0 - 0.05)
    avg_volume_at_level = calculate_avg_volume_at_level(level)
    volume_score = min(avg_volume_at_level / (avg_volume * 1.5), 1.0) * 0.05
    
    return touch_score + recency_score + volume_score
```

**Example**:
- Level touched 4 times: 0.12 (4/5 * 0.15)
- 2 recent touches: 0.067 (2/3 * 0.10)
- High volume at level: 0.05
- **Total Level Strength**: 0.237 (23.7%)

### 2. Breakout Quality (0.0 - 0.3)

Measures how strong the breakout is:

```python
def calculate_breakout_quality(breakout):
    # Factor 1: Break distance (0.0 - 0.12)
    break_distance = (breakout.price - breakout.level) / breakout.level
    distance_score = min(break_distance / (break_threshold * 2), 1.0) * 0.12
    # Break 2x threshold = maximum score
    
    # Factor 2: Volume confirmation (0.0 - 0.12)
    volume_ratio = breakout.volume / avg_volume
    if volume_ratio >= volume_multiplier:
        volume_score = min((volume_ratio - volume_multiplier) / volume_multiplier, 1.0) * 0.12
    else:
        volume_score = 0.0
    
    # Factor 3: Price action quality (0.0 - 0.06)
    # Clean break (closes beyond level) vs wick only
    if breakout.is_clean_break:
        price_action_score = 0.06
    else:
        price_action_score = 0.03  # Partial break
    
    return distance_score + volume_score + price_action_score
```

**Example**:
- Break 1.0% above level (2x threshold): 0.12
- Volume 1.8x average (1.5x required): 0.12
- Clean break: 0.06
- **Total Breakout Quality**: 0.30 (30%)

### 3. Retest Quality (0.0 - 0.2)

Measures how strong the retest confirmation is:

```python
def calculate_retest_quality(retest):
    # Factor 1: Bounce strength (0.0 - 0.12)
    bounce_distance = abs(retest.bounce_price - retest.level) / retest.level
    bounce_score = min(bounce_distance / retest_bounce_threshold, 1.0) * 0.12
    # Strong bounce = higher score
    
    # Factor 2: Volume on retest (0.0 - 0.05)
    # Decreasing volume on retest is good
    retest_volume_ratio = retest.volume / avg_volume
    if retest_volume_ratio < 1.0:
        volume_score = (1.0 - retest_volume_ratio) * 0.05
    else:
        volume_score = 0.0
    
    # Factor 3: Price rejection (0.0 - 0.03)
    # Clear rejection of level
    if retest.shows_rejection:
        rejection_score = 0.03
    else:
        rejection_score = 0.0
    
    return bounce_score + volume_score + rejection_score
```

**Example**:
- Strong bounce (0.5%): 0.12
- Low volume on retest (0.7x average): 0.015
- Clear rejection: 0.03
- **Total Retest Quality**: 0.165 (16.5%)

### 4. Market Context (0.0 - 0.2)

Measures alignment with overall market conditions:

```python
def calculate_market_context(symbol, market_data):
    # Factor 1: Trend alignment (0.0 - 0.10)
    # Breakout in direction of trend = better
    trend = calculate_trend(market_data, timeframe="daily")
    if breakout.direction == trend.direction:
        trend_score = 0.10
    elif trend.is_neutral:
        trend_score = 0.05
    else:
        trend_score = 0.0  # Counter-trend
    
    # Factor 2: Volatility (0.0 - 0.05)
    # Moderate volatility is optimal
    atr_percent = calculate_atr_percent(market_data)
    if 0.01 <= atr_percent <= 0.03:  # 1-3% ATR
        volatility_score = 0.05
    elif 0.005 <= atr_percent < 0.01 or 0.03 < atr_percent <= 0.05:
        volatility_score = 0.025
    else:
        volatility_score = 0.0  # Too low or too high
    
    # Factor 3: Timeframe reliability (0.0 - 0.05)
    # Higher timeframes = more reliable
    if timeframe == "daily":
        timeframe_score = 0.05
    elif timeframe == "4 hour":
        timeframe_score = 0.03
    elif timeframe == "1 hour":
        timeframe_score = 0.02
    else:
        timeframe_score = 0.01
    
    return trend_score + volatility_score + timeframe_score
```

**Example**:
- Breakout with trend: 0.10
- Moderate volatility: 0.05
- Daily timeframe: 0.05
- **Total Market Context**: 0.20 (20%)

## Complete Example Calculation

Let's calculate confidence for a real trade:

**Setup**:
- Resistance level at $100, touched 4 times
- Price breaks to $100.75 (0.75% break) with 1.8x volume
- Price retests $100.20 and bounces to $100.50
- Breakout aligns with uptrend on daily chart

**Calculation**:

1. **Level Strength**: 0.237
   - 4 touches: 0.12
   - 2 recent touches: 0.067
   - High volume: 0.05

2. **Breakout Quality**: 0.30
   - Break distance (0.75% vs 0.5% threshold): 0.12
   - Volume 1.8x (1.2x required): 0.12
   - Clean break: 0.06

3. **Retest Quality**: 0.165
   - Bounce 0.3%: 0.12
   - Low retest volume: 0.015
   - Clear rejection: 0.03

4. **Market Context**: 0.20
   - With trend: 0.10
   - Moderate volatility: 0.05
   - Daily timeframe: 0.05

**Total Confidence Score**: 0.237 + 0.30 + 0.165 + 0.20 = **0.902 (90.2%)**

**Decision**: If `min_confidence = 0.6`, this trade would be executed (0.902 >= 0.6) ✅

## Using min_confidence

### Setting min_confidence

The `min_confidence` parameter acts as a filter:

```yaml
strategy:
  min_confidence: 0.6  # Only trade signals with 60%+ confidence
```

### Impact of min_confidence

- **Low min_confidence (0.4-0.5)**: More trades, but lower quality signals
- **Medium min_confidence (0.6-0.7)**: Balanced - good quality, reasonable quantity
- **High min_confidence (0.8-0.9)**: Fewer trades, but very high quality signals

### Stock-Specific Overrides

You can set different `min_confidence` for different stocks:

```yaml
overrides:
  "TSLA":
    min_confidence: 0.7  # Higher threshold for volatile TSLA
  
  "JNJ":
    min_confidence: 0.5  # Lower threshold for stable JNJ
```

## Confidence Score Interpretation

| Confidence Range | Interpretation | Action |
|-----------------|---------------|--------|
| 0.9 - 1.0 | Excellent signal | Strong buy/sell |
| 0.7 - 0.9 | Good signal | Buy/sell |
| 0.6 - 0.7 | Decent signal | Consider trade |
| 0.5 - 0.6 | Weak signal | Avoid (unless min_confidence < 0.6) |
| 0.0 - 0.5 | Poor signal | Do not trade |

## Adjusting Confidence Calculation

### Increasing Confidence Requirements

To get higher confidence scores, you can:

1. **Require stronger levels**: Increase `min_touch_count`
2. **Require larger breakouts**: Increase `break_threshold`
3. **Require more volume**: Increase `volume_multiplier`
4. **Require stronger retests**: Increase `retest_bounce_threshold`

### Decreasing Confidence Requirements

To get more trading opportunities:

1. **Accept weaker levels**: Decrease `min_touch_count`
2. **Accept smaller breakouts**: Decrease `break_threshold`
3. **Accept lower volume**: Decrease `volume_multiplier`
4. **Lower min_confidence**: Decrease `min_confidence` threshold

## Best Practices

1. **Start with default values**: Use min_confidence = 0.6 initially
2. **Backtest different values**: Test 0.5, 0.6, 0.7, 0.8
3. **Monitor performance**: Track win rate by confidence level
4. **Adjust gradually**: Change one parameter at a time
5. **Use stock-specific overrides**: Volatile stocks may need higher thresholds

## Debugging Confidence Scores

To understand why a trade was or wasn't taken:

1. **Log confidence components**: Log each component separately
2. **Check level strength**: Verify level has enough touches
3. **Check breakout quality**: Verify volume and break distance
4. **Check retest quality**: Verify bounce and rejection
5. **Check market context**: Verify trend alignment

## Example Log Output

```
Signal for AAPL:
  Level Strength: 0.237 (4 touches, 2 recent, high volume)
  Breakout Quality: 0.300 (0.75% break, 1.8x volume, clean break)
  Retest Quality: 0.165 (0.3% bounce, low volume, clear rejection)
  Market Context: 0.200 (with trend, moderate vol, daily TF)
  Total Confidence: 0.902
  Min Confidence: 0.600
  Decision: EXECUTE (0.902 >= 0.600)
```
