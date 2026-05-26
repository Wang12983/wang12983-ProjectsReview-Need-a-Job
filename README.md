# Polymarket BTC Up/Down Research Pipeline

A research-grade data pipeline for Polymarket Bitcoin Up/Down binary markets. Uses only public APIs. Goals: auditable market discovery, order book collection, feature generation, complete-set edge detection, and local reporting.

No live trading. No wallet credentials, order signing, order placement, CTF split/merge/redeem, or real-money execution logic.

## Current Status

Working pipeline:

```text
discover current BTC 15m market
-> read Up / Down token ids
-> fetch public CLOB order books
-> append books / features / complete-set edges
-> report data quality and arbitrage edges
```

Acceptance command:

```powershell
$env:PYTHONPATH = "$PWD\src"
python -m polymarket_updown.cli run-once --iterations 1 --interval-sec 1
```

Acceptance output:

```text
markets: 1
tokens: 2
iterations: 1
token_errors: 0
books_appended: 2
features_appended: 2
edges_appended: 1
feature_error_rows: 0
missing_input_rows: 0
has_complete_set_opportunity: false
```

Test status:

```powershell
$env:PYTHONPATH = "$PWD\src"
python -m unittest discover -s tests
python -m compileall src tests
```

```text
Ran 39 tests, OK
compileall OK
```

## Features

- Gamma API market discovery
- Auto slug derivation for current BTC 15m Up/Down recurring market
- CLOB `/book` public order book reads
- Raw JSON archiving
- Local Parquet storage
- L2 order book feature generation
- Complete-set buy / split-sell edge calculation
- Local reporting via `report-edges`
- One-shot acceptance via `run-once`
- 429 / 5xx retry with `Retry-After` support
- Gamma events paginated reads
- Atomic Parquet / JSON writes
- Graceful degradation on short-cycle token expiry or single-token failures

## Security Boundaries

Default mode: research, replay, and paper-data collection only.

Not implemented:

- Live trading
- Credential loading
- Wallet / private key handling
- Order signing
- Live order placement
- CTF split / merge / redeem
- Chain reconciliation

Any live execution capability requires separate design with full legal, account permissions, API compatibility, fee, heartbeat, kill switch, and on-chain reconciliation review.

## Quick Start

Run in PowerShell:

```powershell
$env:PYTHONPATH = "$PWD\src"
python -m polymarket_updown.cli plan
```

Discover current BTC 15m Up/Down market:

```powershell
python -m polymarket_updown.cli discover-markets --current-btc-15m --out data/parquet/markets.parquet
```

Paginated discovery across active events:

```powershell
python -m polymarket_updown.cli discover-markets --limit 100 --max-pages 20 --out data/parquet/markets.parquet
```

Discover by known slug:

```powershell
python -m polymarket_updown.cli discover-markets --slug btc-updown-15m-1779759000 --out data/parquet/markets.parquet
```

Single order book snapshot:

```powershell
python -m polymarket_updown.cli snapshot-book --markets data/parquet/markets.parquet
```

Continuous order book collection:

```powershell
python -m polymarket_updown.cli collect-books --markets data/parquet/markets.parquet --interval-sec 2 --duration-sec 60
```

Local edge report:

```powershell
python -m polymarket_updown.cli report-edges --edges data/parquet/complete_set_edges.parquet --features data/parquet/book_features.parquet
```

Full pipeline acceptance run:

```powershell
python -m polymarket_updown.cli run-once --iterations 2 --interval-sec 1
```

## Output Files

Default output paths:

```text
data/raw/
data/parquet/markets.parquet
data/parquet/books.parquet
data/parquet/book_features.parquet
data/parquet/complete_set_edges.parquet
```

`data/` contains local collection artifacts and is excluded from version control via `.gitignore`.

## Project Structure

```text
configs/                 Default configuration
docs/                    Implementation plans, data contracts, risk boundaries
src/polymarket_updown/   Python package
tests/                   Unit tests
```

Core modules:

```text
clients.py      Public API client with retry/backoff
discovery.py    Market discovery and BTC Up/Down filtering
normalize.py    Gamma payload normalization
features.py     Order book features
fees.py         Taker fee and complete-set edge formulas
snapshots.py    Order book snapshot normalization and edge rows
collection.py   Single-round order book collection
reports.py      Local reporting
storage.py      Raw JSON / Parquet atomic writes
cli.py          CLI entrypoint
```
