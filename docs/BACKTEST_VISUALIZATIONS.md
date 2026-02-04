# Backtest Visualizations

## Overview

The backtesting system generates comprehensive visualizations to help you analyze trade execution and strategy performance. All charts are saved as images and embedded in the HTML report.

## Chart Types

### 1. Price Chart with Trades

**Purpose**: Visualize entry/exit points on price action

**Elements**:
- **Candlestick/Bar Chart**: OHLC price data
- **Support/Resistance Levels**: Horizontal lines showing identified levels
- **Entry Points**: Green upward arrows (▲) for long entries
- **Exit Points**: Red downward arrows (▼) for exits
- **Stop Loss Lines**: Red horizontal dashed lines
- **Profit Target Lines**: Green horizontal dashed lines
- **Breakout Points**: Blue markers where price broke through levels
- **Retest Points**: Yellow markers where price retested broken levels
- **Volume Bars**: Volume histogram below price chart

**Example Layout**:
```
Price Chart
├── Support/Resistance Levels (horizontal lines)
├── Price Candles/Bars
├── Entry Markers (green ▲)
├── Exit Markers (red ▼)
├── Stop Loss Lines (red dashed)
├── Profit Target Lines (green dashed)
└── Breakout/Retest Markers (blue/yellow)

Volume Chart (below)
└── Volume Bars
```

**Information Displayed**:
- Entry price and time
- Exit price and time
- P&L for each trade
- Trade duration
- Entry confidence score

### 2. Equity Curve

**Purpose**: Show account value progression over time

**Elements**:
- **Equity Line**: Blue line showing account value
- **Initial Capital**: Horizontal reference line
- **Trade Markers**: Vertical lines marking trade entries (green) and exits (red)
- **Drawdown Areas**: Shaded red areas showing drawdown periods
- **Benchmark Line**: Optional comparison to buy-and-hold (dashed line)

**Metrics Displayed**:
- Final equity value
- Peak equity value
- Maximum drawdown
- Recovery periods

**Example**:
```
Equity Curve
│
│     ╱╲
│    ╱  ╲    ╱╲
│   ╱    ╲  ╱  ╲
│  ╱      ╲╱    ╲
│ ╱            ╲
│╱              ╲
└─────────────────> Time
```

### 3. Drawdown Chart

**Purpose**: Visualize drawdown periods and recovery

**Elements**:
- **Drawdown Line**: Red line showing drawdown percentage
- **Zero Line**: Reference line at 0% drawdown
- **Max Drawdown Marker**: Highlighted point showing maximum drawdown
- **Recovery Periods**: Shaded areas showing time to recover

**Information**:
- Maximum drawdown value and date
- Drawdown duration
- Recovery time
- Number of drawdown periods

### 4. Trade Distribution

**Purpose**: Analyze distribution of trade outcomes

**Chart Types**:

#### A. P&L Histogram
- **X-axis**: P&L amount (bins)
- **Y-axis**: Number of trades
- **Colors**: Green for profits, red for losses
- Shows distribution of winning vs losing trades

#### B. Win/Loss Comparison
- **Bar Chart**: Comparing total wins vs losses
- **Metrics**: Win rate, average win, average loss

#### C. Trade Duration Distribution
- **Histogram**: Distribution of trade durations
- Shows typical holding period

### 5. Monthly Returns

**Purpose**: Analyze performance by month

**Elements**:
- **Monthly Bars**: Green for positive months, red for negative
- **Cumulative Line**: Running total of returns
- **Average Line**: Average monthly return (dashed)
- **Best/Worst Months**: Highlighted

**Information**:
- Monthly return percentage
- Best performing month
- Worst performing month
- Consistency of returns

### 6. Strategy Analysis Charts

**Purpose**: Analyze strategy-specific performance factors

#### A. Confidence vs Performance
- **Scatter Plot**: Entry confidence vs trade P&L
- **Trend Line**: Shows correlation
- **Insight**: Does higher confidence lead to better trades?

#### B. Breakout Quality vs Performance
- **Scatter Plot**: Breakout strength vs trade outcome
- **Grouped by**: Breakout volume, break distance
- **Insight**: What breakout characteristics lead to wins?

#### C. Retest Quality vs Performance
- **Scatter Plot**: Retest strength vs trade outcome
- **Grouped by**: Bounce strength, volume on retest
- **Insight**: What retest characteristics lead to wins?

#### D. Level Strength vs Performance
- **Bar Chart**: Performance by level strength category
- **Categories**: Weak (0-0.3), Medium (0.3-0.6), Strong (0.6-1.0)
- **Insight**: Do stronger levels lead to better trades?

### 7. Performance Metrics Dashboard

**Purpose**: Quick overview of key metrics

**Layout**:
```
┌─────────────────────────────────────┐
│  Total Return: 25.3%                 │
│  Sharpe Ratio: 1.85                  │
│  Win Rate: 62.2%                     │
│  Max Drawdown: -12.1%                │
│  Profit Factor: 2.45                 │
│  Total Trades: 45                    │
└─────────────────────────────────────┘
```

### 8. Trade Timeline

**Purpose**: Visualize trade sequence and timing

**Elements**:
- **Timeline**: Horizontal timeline
- **Trade Bars**: Colored bars showing trade duration
- **Colors**: Green for wins, red for losses
- **Height**: Proportional to P&L amount

**Information**:
- Trade sequence
- Overlapping positions
- Time between trades
- Win/loss streaks

## Interactive Features (Plotly)

If using Plotly for charts:

1. **Zoom and Pan**: Interactive zooming and panning
2. **Hover Tooltips**: Detailed information on hover
3. **Legend Toggle**: Show/hide chart elements
4. **Data Points**: Click to see trade details
5. **Export**: Download charts as images

## Chart Customization

### Configuration Options

```yaml
charts:
  # Chart appearance
  style: "dark"              # "light" or "dark"
  figure_size: [12, 8]        # Width, height in inches
  dpi: 300                    # Resolution for PNG export
  
  # Price chart settings
  price_with_trades:
    show_volume: true
    show_levels: true
    show_indicators: false
    candle_style: "candlestick"  # "candlestick" or "line"
  
  # Equity curve settings
  equity_curve:
    show_drawdown: true
    show_benchmark: true
    benchmark_symbol: "SPY"
  
  # Colors
  colors:
    entry: "#00FF00"          # Green
    exit: "#FF0000"            # Red
    profit: "#00AA00"          # Dark green
    loss: "#AA0000"            # Dark red
    support: "#0000FF"         # Blue
    resistance: "#FF00FF"      # Magenta
```

## Report Layout

The HTML report organizes charts in sections:

1. **Executive Summary**
   - Key metrics dashboard
   - Overall performance rating

2. **Performance Analysis**
   - Equity curve
   - Drawdown chart
   - Monthly returns

3. **Trade Analysis**
   - Price chart with trades
   - Trade distribution
   - Trade timeline

4. **Strategy Analysis**
   - Confidence vs performance
   - Breakout quality analysis
   - Retest quality analysis
   - Level strength analysis

5. **Detailed Metrics**
   - Complete performance metrics table
   - Trade-by-trade breakdown

## Export Options

Charts can be exported as:
- **PNG**: High-resolution images
- **PDF**: Vector format for printing
- **HTML**: Interactive Plotly charts
- **CSV**: Underlying data for custom analysis

## Best Practices

1. **Review Price Charts First**: Understand trade execution context
2. **Check Equity Curve**: Verify consistent growth
3. **Analyze Drawdowns**: Understand worst-case scenarios
4. **Study Trade Distribution**: Identify patterns in wins/losses
5. **Review Strategy Analysis**: Understand what works/doesn't work
6. **Compare to Benchmark**: See if strategy beats buy-and-hold

## Example Analysis Workflow

1. **Start with Equity Curve**: Get overall performance picture
2. **Review Price Charts**: See how trades were executed
3. **Analyze Drawdowns**: Identify risk periods
4. **Study Trade Distribution**: Find win/loss patterns
5. **Deep Dive into Strategy Analysis**: Understand what drives performance
6. **Compare Metrics**: Evaluate against benchmarks
7. **Identify Improvements**: Use insights to refine strategy

## Technical Details

### Chart Generation

```python
class TradeVisualizer:
    def plot_price_with_trades(self, data, trades, levels):
        # Generate price chart with all trade markers
        fig = plt.figure(figsize=(14, 10))
        
        # Price subplot
        ax1 = plt.subplot(2, 1, 1)
        # Plot candlesticks
        # Plot support/resistance levels
        # Plot entry/exit markers
        # Plot stop loss/profit target lines
        
        # Volume subplot
        ax2 = plt.subplot(2, 1, 2)
        # Plot volume bars
        
        return fig
    
    def plot_equity_curve(self, equity_curve, trades):
        # Generate equity curve with trade markers
        fig = plt.figure(figsize=(12, 6))
        
        # Plot equity line
        # Shade drawdown areas
        # Mark trade entries/exits
        # Add benchmark if specified
        
        return fig
```

### Data Requirements

For accurate visualizations:
- Complete OHLCV data (no gaps)
- Accurate timestamps
- Trade execution details
- Portfolio equity history

## Troubleshooting

**Charts not generating**:
- Check data availability
- Verify matplotlib/plotly installation
- Check output directory permissions

**Charts look incorrect**:
- Verify data timestamps
- Check trade entry/exit times
- Validate price data

**Performance issues**:
- Reduce chart resolution
- Limit number of trades displayed
- Use simpler chart styles
