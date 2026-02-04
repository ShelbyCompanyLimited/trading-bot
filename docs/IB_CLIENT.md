# IB Client Implementation

## Overview

The `IBClient` class (`src/ib_client.py`) is a wrapper around the `ib_async` library that provides a clean interface to Interactive Brokers' Trader Workstation (TWS) and IB Gateway.

## Key Features

- Automatic connection and reconnection
- Market data subscription
- Order placement and tracking
- Account information retrieval
- Position updates
- Error handling and recovery

## Class Structure

```python
class IBClient:
    def __init__(self, host: str, port: int, client_id: int, account: str = "")
    
    async def connect(self) -> bool
    async def disconnect(self)
    async def is_connected(self) -> bool
    
    async def get_account_summary(self) -> Dict
    async def get_account_equity(self) -> float
    async def get_positions(self) -> List[Dict]
    
    async def subscribe_market_data(self, symbol: str) -> bool
    async def unsubscribe_market_data(self, symbol: str)
    async def get_current_price(self, symbol: str) -> Optional[float]
    async def get_historical_data(
        self, 
        symbol: str, 
        duration: str = "1 D", 
        bar_size: str = "1 min"
    ) -> pd.DataFrame
    
    async def place_market_order(
        self, 
        symbol: str, 
        quantity: int, 
        action: str = "BUY"
    ) -> Optional[Order]
    
    async def place_limit_order(
        self, 
        symbol: str, 
        quantity: int, 
        limit_price: float, 
        action: str = "BUY"
    ) -> Optional[Order]
    
    async def place_stop_loss_order(
        self, 
        symbol: str, 
        quantity: int, 
        stop_price: float
    ) -> Optional[Order]
    
    async def cancel_order(self, order_id: int) -> bool
    async def get_order_status(self, order_id: int) -> Optional[str]
```

## Connection Management

### Initialization
```python
client = IBClient(
    host="127.0.0.1",
    port=7497,  # Paper trading port
    client_id=1,
    account=""  # Empty for default account
)
```

### Connection
- Uses `ib_async.IB` class for connection
- Implements automatic reconnection on disconnect
- Handles connection errors gracefully
- Logs connection status changes

### Reconnection Logic
- Detects disconnections automatically
- Exponential backoff for reconnection attempts
- Maximum retry limits
- Notifies bot of connection status changes

## Market Data

### Subscription
- Subscribes to real-time market data for symbols
- Handles multiple concurrent subscriptions
- Stores latest prices in memory
- Provides callbacks for price updates

### Historical Data
- Retrieves historical bar data
- Returns pandas DataFrame with OHLCV data
- Supports various durations and bar sizes
- Used for strategy backtesting and signal generation

## Order Management

### Order Types
- **Market Orders**: Immediate execution at current market price
- **Limit Orders**: Execute only at specified price or better
- **Stop Loss Orders**: Triggered when price reaches stop level

### Order Placement
- Validates order parameters before submission
- Returns order object for tracking
- Handles order errors and rejections
- Logs all order activities

### Order Tracking
- Monitors order status (submitted, filled, cancelled, etc.)
- Tracks fill prices and quantities
- Updates portfolio on order fills
- Handles partial fills

## Account Information

### Account Summary
- Retrieves account equity, buying power, margin
- Updates account information periodically
- Used for position sizing calculations

### Positions
- Gets current open positions from IB
- Syncs with PortfolioManager
- Handles position updates from IB

## Error Handling

### Connection Errors
- Network failures
- TWS/Gateway not running
- Invalid credentials
- Port conflicts

### API Errors
- Invalid symbols
- Insufficient buying power
- Order rejections
- Market data subscription failures

### Recovery Strategies
- Automatic reconnection
- Order retry with backoff
- Graceful degradation
- Comprehensive error logging

## Usage Example

```python
# Initialize client
client = IBClient(host="127.0.0.1", port=7497, client_id=1)

# Connect
await client.connect()

# Get account equity
equity = await client.get_account_equity()

# Subscribe to market data
await client.subscribe_market_data("AAPL")

# Get current price
price = await client.get_current_price("AAPL")

# Place market order
order = await client.place_market_order("AAPL", quantity=100, action="BUY")

# Get historical data
historical = await client.get_historical_data("AAPL", duration="1 D", bar_size="1 min")

# Disconnect
await client.disconnect()
```

## Dependencies

- `ib_async`: Modern async-based IB API wrapper
- `pandas`: Data manipulation for historical data
- `asyncio`: Async/await support

## Configuration

IB connection settings are configured in `config/config.yaml`:
```yaml
ib:
  host: "127.0.0.1"
  port: 7497
  client_id: 1
  account: ""
```

## Port Numbers

- **7497**: TWS Paper Trading
- **4001**: IB Gateway Paper Trading
- **7496**: TWS Live Trading
- **4002**: IB Gateway Live Trading
