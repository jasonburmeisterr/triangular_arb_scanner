# Triangular Arbitrage Scanner

A read-only Python scanner that monitors crypto exchange order books for **triangular arbitrage** opportunities in real time — built as a learning project to explore exchange APIs, fee-aware trade math, and market microstructure.

> ⚠️ **Educational project.** This scanner does not place trades. Real triangular arbitrage windows on liquid pairs typically close in milliseconds, faster than a REST-polling script can act. This is a tool for understanding the mechanics, not a production trading bot.

## What it does

The scanner watches a loop of three trading pairs on a single exchange (e.g. `USD → BTC → ETH → USD`) and continuously checks whether converting through all three legs returns more money than you started with, after fees. If it does, that's a triangular arbitrage opportunity.

- Pulls live order book data via [ccxt](https://github.com/ccxt/ccxt), a unified API wrapper for 100+ crypto exchanges
- Checks both directions of the triangle (forward and reverse)
- Accounts for taker fees on all three legs
- Logs every scan to a CSV file for later analysis in Excel/Google Sheets
- Flags any result that clears a configurable profit threshold

## Setup

```bash
pip install ccxt
```

No API key is required — the scanner only reads public market data.

## Usage

```bash
python triangular_arb_scanner.py
```

The script runs continuously, checking prices every 5 seconds until you stop it (`Ctrl+C`). Every scan — not just flagged opportunities — is appended to `arb_scan_log.csv` in the same folder, so you can leave it running and review results later.

### Sample output

```
Scanning kraken for triangular arbitrage: USDT <-> BTC/USD <-> ETH/BTC <-> ETH/USD
Fee assumption: 0.260% per trade (0.780% total for 3 legs)
Reporting threshold: 0.05% net profit
Logging every scan to: /path/to/arb_scan_log.csv

[14:32:05] forward: -0.1123%  (below threshold)
[14:32:05] reverse: -0.0847%  (below threshold)
[14:32:10] forward: -0.0932%  (below threshold)
```

### CSV log columns

| Column | Description |
|---|---|
| `date`, `time` | When the scan ran |
| `direction` | `forward` or `reverse` |
| `profit_pct` | Net profit percentage after fees |
| `is_opportunity` | `True` if it cleared the configured threshold |
| `final_usdt` | Ending balance from a hypothetical starting amount |
| `leg1_symbol` / `leg1_price` ... | Prices used for each leg of the triangle |

## Configuration

All the tunable settings live at the top of `triangular_arb_scanner.py`:

```python
EXCHANGE_ID = "kraken"           # any ccxt-supported exchange
TAKER_FEE = 0.0026               # match your exchange's actual fee tier
MIN_PROFIT_PCT = 0.05            # % threshold before an opportunity is flagged
SCAN_INTERVAL_SECONDS = 5        # how often to poll
TRIANGLE = {
    "leg1": "BTC/USD",
    "leg2": "ETH/BTC",
    "leg3": "ETH/USD",
}
```

Swap in any three pairs your exchange supports, as long as they form a valid loop back to the starting currency.

**Note on exchange availability:** `binance.com` blocks connections from the US (HTTP 451). US-based users should use an exchange like Kraken or Coinbase instead — this is set as the default.

## How it works

1. Fetch the top-of-book bid/ask for each pair in the triangle
2. Walk the loop: buy the first asset, use it to buy the second, sell the second back to the starting currency
3. Multiply through each leg, subtracting the taker fee at every step
4. Compare the final amount to the starting amount — anything above the fee cost is a potential opportunity
5. Repeat for the reverse direction, since mispricing can appear either way
6. Log the result and repeat on a timer

## Roadmap / ideas for extending this

- [ ] Move from REST polling to WebSocket streams for lower latency
- [ ] Add a 4-leg or cross-exchange variant
- [ ] Visualize the CSV log (profit % over time) with matplotlib or a spreadsheet chart
- [ ] Backtest against historical order book data instead of scanning live

## Disclaimer

This project is for educational purposes only. It does not constitute financial advice, and it is not intended for live trading without significant further development (execution logic, latency optimization, risk management, and testing). Cryptocurrency trading carries substantial risk.
