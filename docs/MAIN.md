# Main Entry Point

## Overview

The `main.py` file is the entry point for running the trading bot. It handles initialization, startup, and graceful shutdown.

## File Structure

```python
import asyncio
import signal
import sys
from src.bot import TradingBot
from src.utils.config import load_config
from src.utils.logger import setup_logger

async def main():
    # Load configuration
    # Initialize logger
    # Create and start bot
    # Handle shutdown

if __name__ == "__main__":
    asyncio.run(main())
```

## Execution Flow

### 1. Load Configuration

```python
config = load_config("config/config.yaml")
```

Loads configuration from YAML file and environment variables.

### 2. Initialize Logging

```python
logger = setup_logger(
    name="trading_bot",
    log_file=config["logging"]["file"],
    level=config["logging"]["level"],
    console=config["logging"]["console"]
)
```

Sets up logging with file and console handlers.

### 3. Create Trading Bot

```python
bot = TradingBot(config)
```

Initializes the trading bot with configuration.

### 4. Initialize Bot

```python
if not await bot.initialize():
    logger.error("Failed to initialize bot")
    sys.exit(1)
```

Initializes all components and connects to IB.

### 5. Start Bot

```python
await bot.start()
```

Starts the main event loop and begins trading.

### 6. Handle Shutdown

```python
# Signal handlers for graceful shutdown
signal.signal(signal.SIGINT, handle_shutdown)
signal.signal(signal.SIGTERM, handle_shutdown)
```

Handles Ctrl+C and termination signals gracefully.

## Signal Handling

### SIGINT (Ctrl+C)

- Stops the bot gracefully
- Closes positions (optional)
- Disconnects from IB
- Saves state (optional)

### SIGTERM

- Same as SIGINT
- Used by process managers
- Allows clean shutdown

## Error Handling

### Configuration Errors

- Log error and exit
- Provide helpful error messages
- Validate configuration on startup

### Connection Errors

- Retry with backoff
- Log connection attempts
- Exit if unable to connect after retries

### Runtime Errors

- Log errors with context
- Continue running if possible
- Graceful degradation
- Alert on critical errors

## Command Line Arguments

### Future Enhancements

Could support:
- `--config`: Specify config file path
- `--dry-run`: Run without trading
- `--symbols`: Override symbols to trade
- `--log-level`: Override log level

## Example Execution

```bash
# Run bot
python main.py

# Output:
# 2024-01-01 10:00:00 - trading_bot - INFO - Loading configuration
# 2024-01-01 10:00:00 - trading_bot - INFO - Initializing logger
# 2024-01-01 10:00:00 - trading_bot - INFO - Creating TradingBot
# 2024-01-01 10:00:01 - trading_bot - INFO - Connecting to IB...
# 2024-01-01 10:00:02 - trading_bot - INFO - Connected to IB
# 2024-01-01 10:00:02 - trading_bot - INFO - Starting bot...
# 2024-01-01 10:00:02 - trading_bot - INFO - Bot started
# ...
```

## Graceful Shutdown

### Process

1. **Receive Signal**: SIGINT or SIGTERM
2. **Set Flag**: Set shutdown flag
3. **Stop Bot**: Call `bot.stop()`
4. **Wait for Completion**: Wait for current operations
5. **Cleanup**: Close connections, save state
6. **Exit**: Exit with appropriate code

### Cleanup Tasks

- Cancel pending orders (optional)
- Close positions (optional)
- Disconnect from IB
- Flush log files
- Save portfolio state (optional)

## Logging

### Startup Logs

- Configuration loaded
- Components initialized
- Connection established
- Bot started

### Runtime Logs

- Strategy evaluations
- Signal generation
- Order execution
- Position updates

### Shutdown Logs

- Shutdown initiated
- Cleanup tasks
- Final state
- Exit confirmation

## Exit Codes

- `0`: Successful execution
- `1`: Configuration error
- `2`: Connection error
- `3`: Initialization error
- `4`: Runtime error

## Running as a Service

### Systemd Service

```ini
[Unit]
Description=Trading Bot
After=network.target

[Service]
Type=simple
User=trading
WorkingDirectory=/path/to/trading-bot
ExecStart=/usr/bin/python3 main.py
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

### Docker

```dockerfile
FROM python:3.10
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
CMD ["python", "main.py"]
```

## Monitoring

### Health Checks

- Connection status
- Bot running status
- Recent log activity
- Position count

### Metrics

- Uptime
- Trades executed
- P&L
- Error rate

## Best Practices

1. **Always test in paper trading first**
2. **Monitor logs closely during initial runs**
3. **Use process managers for production**
4. **Implement health checks**
5. **Set up alerts for errors**
6. **Regular backups of configuration**
7. **Document any custom modifications**
