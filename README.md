# Gnoland Sapphire Services

Public infrastructure and tools for Gnoland Sapphire testnet operators by [Apollo Validator](https://apollo-validator.eu).

[![Sapphire](https://img.shields.io/badge/Chain-sapphire--1-blue)](https://sapphire.testnets.gno.land)
[![RPC](https://img.shields.io/badge/RPC-rpc.apollo--validator.eu-green)](https://rpc.apollo-validator.eu/gnoland/)

## Services

| Service | Endpoint | Status |
|---|---|---|
| Public RPC | `https://rpc.apollo-validator.eu/gnoland/` | Active |
| Snapshots | `https://snapshots.apollo-validator.eu/gnoland/` | Active |
| State Sync Guide | [statesync.md](statesync.md) | Active |
| Peers List | [peers.md](peers.md) (auto-updated) | Active |
| Validator Guide | [guide.md](guide.md) | Active |
| Validator Monitor | @gnoland_monitor_apollo_bot | Active |

## Snapshots

Page: `https://snapshots.apollo-validator.eu/gnoland/`

Snapshots are created every 6 hours and stored for fast node synchronization.

### Download Latest

```bash
# Get snapshot info
curl -s https://snapshots.apollo-validator.eu/api/gnoland/snapshots/latest | jq

# Download latest snapshot
wget -O gnoland-snapshot.tar.lz4 https://snapshots.apollo-validator.eu/gnoland/snapshots/latest.tar.lz4
```

### Restore from Snapshot

```bash
# Install lz4
sudo apt install liblz4-tool -y

# Stop node
sudo systemctl stop gnoland

# Backup validator state (validators only)
cp ~/gno/gnoland-data/secrets/priv_validator_state.json ~/priv_validator_state.json.bak

# Download latest snapshot
wget -O gnoland-snapshot.tar.lz4 https://snapshots.apollo-validator.eu/gnoland/snapshots/latest.tar.lz4

# Restore
rm -rf ~/gno/gnoland-data/db ~/gno/gnoland-data/wal
lz4 -d gnoland-snapshot.tar.lz4 | tar -xf - -C ~/gno/gnoland-data/

# Restore validator state (validators only)
cp ~/priv_validator_state.json.bak ~/gno/gnoland-data/secrets/priv_validator_state.json

# Start node
sudo systemctl start gnoland

# Clean up
rm -v gnoland-snapshot.tar.lz4
```

### API

```bash
curl -s https://snapshots.apollo-validator.eu/api/gnoland/snapshots/latest | jq
```

## Public RPC

Base URL: `https://rpc.apollo-validator.eu/gnoland/`

### Available Endpoints

| Endpoint | Description |
|---|---|
| `/status` | Node sync status, block height |
| `/abci_info` | ABCI application info |
| `/net_info` | Network peers info |
| `/health` | Node health check |
| `/genesis` | Genesis file |
| `/block?height=N` | Block by height |
| `/validators?height=N` | Validator set |
| `/commit?height=N` | Block commit |
| `/tx?hash=H` | Transaction by hash |
| `/broadcast_tx_commit?tx=TX` | Broadcast transaction |
| `/consensus_state` | Consensus state |

### Usage Examples

**Check node status:**
```bash
curl -s https://rpc.apollo-validator.eu/gnoland/status | jq
```

**Get current block height:**
```bash
curl -s https://rpc.apollo-validator.eu/gnoland/status | jq -r '.result.sync_info.latest_block_height'
```

**Get network peers:**
```bash
curl -s https://rpc.apollo-validator.eu/gnoland/net_info | jq -r '.result.n_peers'
```

### Configuration for Wallets/Tools

```
RPC URL: https://rpc.apollo-validator.eu/gnoland/
Chain ID: sapphire-1
```

## Validator Monitor Bot

Public Telegram bot for monitoring Gnoland Sapphire validators. Each user has their own monitoring list.

**Bot:** [@gnoland_monitor_apollo_bot](https://t.me/gnoland_monitor_apollo_bot)

### Features

- **Network status** — height, peers, validators, voting power
- **Add by signing address** — most reliable method
- **Add by moniker** — supported for validators (auto-updated from sapphire)
- **Auto alerts** — RPC issues, validator leaves active set, zero voting power, low peers
- **Per-user monitoring** — each user tracks their own validators

### Commands

| Command | Description |
|---------|-------------|
| `/add <address or moniker>` | Add validator to monitoring |
| `/remove <address or moniker>` | Remove validator |
| `/list` | List your monitored validators |
| `/status` | Show network info |
| `/help` | Show help |

### Adding a Validator

**By signing address (most reliable):**
```
/add g1sgu52u6hfffg9tyck7v3zgd27hhv2paf9rgamr
```

**By moniker (if supported):**
```
/add Apollo
/add UTSA
/add CoreNode
```

Find your signing address at:
- [Valopers page](https://sapphire.testnets.gno.land/r/gnops/valopers)
- [GnoScan](https://gnoscan.io)

### Auto Alerts

The bot sends alerts for:
- **RPC unreachable** — cannot reach Sapphire node
- **Not in active set** — validator dropped from active set
- **Zero voting power** — validator has no delegations
- **Low peers** — fewer than 5 connected peers

## Validator Guide

Full installation and setup guide: [guide.md](guide.md)

### Quick Start

```bash
# Install Go and build
git clone https://github.com/gnolang/gno.git && cd gno && git checkout chain/sapphire
make -C gno.land install.gnoland install.gnokey

# Configure
gnoland config init
gnoland secrets init

# Set required config
gnoland config set p2p.persistent_peers "g10xll77gz6yzg43v9mdalj8360ng6sunt2vvvhf@seed-1.sapphire.testnets.gno.land:26656,g1gw2d7qsmrg06p204ty2qs8ygzd32t2c7p46te0@seed-2.sapphire.testnets.gno.land:26656"
gnoland config set application.prune_strategy syncable
gnoland config set consensus.timeout_commit 3s
gnoland config set moniker YOUR-MONIKER

# Download genesis
wget -O ~/gno/genesis.json https://github.com/gnolang/gno/releases/download/chain/sapphire/genesis.json

# Start
gnoland start --chainid sapphire-1 --genesis ~/gno/genesis.json --skip-genesis-sig-verification
```

## Validator

| Field | Value |
|---|---|
| Operator | Apollo Validator |
| Signing Address | `g1sgu52u6hfffg9tyck7v3zgd27hhv2paf9rgamr` |
| Operator Address | `g1z360harzpshhdnlrdgj5ljkx2aeckzavkyl9g0` |
| Chain | sapphire-1 |
| Website | [apollo-validator.eu](https://apollo-validator.eu) |

## Links

- [Gnoland Website](https://gno.land)
- [Sapphire Testnet](https://sapphire.testnets.gno.land)
- [GnoScan Explorer](https://gnoscan.io)
- [Official Validator Docs](https://github.com/gnolang/gno/blob/chain/sapphire/misc/deployments/sapphire.gno.land/VALIDATOR.md)
- [Discord](https://discord.gg/gnoland)

## License

MIT
