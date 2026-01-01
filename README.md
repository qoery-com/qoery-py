<a href="https://www.producthunt.com/products/qoery-python-sdk?embed=true&amp;utm_source=badge-featured&amp;utm_medium=badge&amp;utm_campaign=badge-qoery-python-sdk" target="_blank" rel="noopener noreferrer"><img alt="Qoery Python SDK - 50% cheaper crypto data. Now in Python. | Product Hunt" width="250" height="54" src="https://api.producthunt.com/widgets/embed-image/v1/featured.svg?post_id=1056631&amp;theme=light&amp;t=1767263265752"></a>

# qoery-py

**The Official Python SDK for Qoery - The Cheapest DeFi Data API**

Access real-time and historical DeFi data from Uniswap and other DEXs with the most cost-effective API on the market.

[![Python Version](https://img.shields.io/badge/python-3.9+-blue.svg)](https://www.python.org/downloads/)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

## [Docs](https://qoery.com/docs/sdks/python/overview)

## Features

- **Simple & Intuitive**: Clean, Pythonic API that feels natural
- **OHLCV Candles**: Get aggregated price data with flexible intervals
- **Raw Tick Data**: Access individual swap transactions for detailed analysis
- **Pool Discovery**: Find the best liquidity pools across multiple networks
- **Multi-Network Support**: Ethereum, Arbitrum, Polygon, Base, and Optimism
- **Usage Tracking**: Monitor your API usage and rate limits
- **Type-Safe**: Full type hints and dataclasses for better IDE support

## Installation

```bash
pip install qoery
```

## Authentication

Get your free API key at [qoery.com](https://qoery.com)

Set your API key as an environment variable:

```bash
export QOERY_API_KEY="your-api-key-here"
```

## Quick Start

```python
import qoery

# Initialize - it's that simple!
client = qoery.Client()

# Get analyzed data instantly as Pandas DataFrame
df = client.candles.get(symbol="ETH-USDC", interval="15m").df
print(df.head())
```

## Why This SDK?

This SDK provides two layers:

1. **`qoery`** (recommended) - Clean, developer-friendly wrapper
2. **`qoery._internal`** - Auto-generated OpenAPI client (lower-level)

**The clean wrapper provides:**
- Cleaner, more intuitive API
- Better error handling
- Pythonic naming conventions
- Type-safe dataclasses instead of dictionaries
- Context manager support
- Sensible defaults

## Examples

### Get OHLCV Candles

```python
# Simple - using symbol
candles = client.candles.get(
    symbol="ETH-USDC",
    interval="15m",
    limit=10
)

# Advanced - using pool address (faster, cheaper)
candles = client.candles.get(
    pool="0x88e6a0c2ddd26feeb64f039a2c41296fcb3f5640",
    interval="1h",
    limit=100,
    networks="ethereum"
)
```

### Get Raw Tick Data

```python
ticks = client.ticks.get(
    symbol="WBTC-USDT",
    limit=100
)

for tick in ticks:
    print(f"{tick.timestamp}: ${tick.price} ({tick.side})")
```

### Find Liquidity Pools

```python
pools = client.pools.find(symbol="ETH-USDC")

for pool in pools:
    print(f"{pool.network}: TVL ${pool.tvl_usd}")
```

### Check API Usage

```python
usage = client.account.usage()

print(f"Plan: {usage.subscription_plan}")
print(f"Credits: {usage.credits_month.used}/{usage.credits_month.limit}")
print(f"Remaining: {usage.credits_month.remaining}")
```

### Using Context Manager

```python
with qoery.Client() as client:
    candles = client.candles.get(symbol="ETH-USDC", interval="15m")
    # Client automatically closes when exiting context
```

### Error Handling

```python
try:
    candles = client.candles.get(symbol="ETH-USDC", interval="15m")
except qoery.AuthenticationError:
    print("Invalid API key")
except qoery.InvalidRequestError as e:
    print(f"Bad request: {e}")
except qoery.RateLimitError:
    print("Rate limit exceeded")
except qoery.APIError as e:
    print(f"API error: {e}")
```

## Full Examples

Check out the [`examples/`](examples/) directory:

- **[simple_usage.py](examples/simple_usage.py)**: Clean, modern examples using `qoery`
- **[basic_usage.py](examples/basic_usage.py)**: Examples using the auto-generated client directly

Run the examples:

```bash
python examples/simple_usage.py
```

## Testing

Run the test suite:

```bash
pytest
```

## Documentation

### API Resources

The client provides the following resources:

- **`client.candles`** - OHLCV candle data
  - `get()` / `list()` - Get candles
- **`client.ticks`** - Raw tick/swap data
  - `get()` / `list()` - Get ticks
- **`client.pools`** - Pool discovery
  - `find()` / `search()` / `list()` - Find pools
- **`client.account`** - Account management
  - `usage()` / `get_usage()` - Get usage stats

### Data Models

All responses use clean dataclasses:

- `Candle` - OHLCV data
- `Tick` - Raw swap data
- `Pool` - Pool information
- `Token` - Token details
- `Usage` - Usage statistics
- `Response` - Generic response wrapper with `data` and `credits_used`

### Configuration

```python
client = qoery.Client(
    api_key="your-key",           # Or use QOERY_API_KEY env var
    base_url="https://api.qoery.com/v0",  # Custom base URL
    timeout=30                     # Request timeout in seconds
)
```

## Advanced Usage

### Using Pool Addresses (Recommended)

Using pool addresses is faster and uses fewer credits:

```python
# Step 1: Find the best pool
pools = client.pools.find(symbol="ETH-USDC")
best_pool = pools.data[0]  # Sorted by liquidity

# Step 2: Use the pool address
candles = client.candles.get(
    pool=best_pool.id,
    interval="15m",
    limit=100
)
```

### Time Range Queries

```python
from datetime import datetime, timedelta

# Get candles from last 24 hours
candles = client.candles.get(
    symbol="ETH-USDC",
    interval="1h",
    from_time=datetime.now() - timedelta(days=1),
    to_time=datetime.now()
)
```

### Multi-Network Queries

```python
# Query specific networks
candles = client.candles.get(
    symbol="ETH-USDC",
    interval="15m",
    networks="ethereum,arbitrum,polygon"
)
```

## Development

### Project Structure

```
qoery-py/
├── qoery/               # Main package (clean wrapper - use this!)
│   ├── client.py        # Main client
│   ├── models.py        # Clean dataclass models
│   ├── exceptions.py    # Custom exceptions
│   ├── resources/       # Resource endpoints
│   │   ├── candles.py
│   │   ├── ticks.py
│   │   ├── pools.py
│   │   └── account.py
│   └── _internal/       # Auto-generated OpenAPI client (private)
│       ├── api/
│       ├── models/
│       └── ...
├── examples/            # Usage examples
└── docs/                # Documentation
```

## License

MIT License - see [LICENSE](LICENSE) file for details

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Support

- Website: [qoery.com](https://qoery.com)
- Documentation: [qoery.com/docs](https://qoery.com/docs/sdks/python/overview)
- Issues: [GitHub Issues](https://github.com/realfishsam/qoery-py/issues)

---

Made by the Qoery team
