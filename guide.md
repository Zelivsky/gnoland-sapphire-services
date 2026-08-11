# Gnoland Sapphire Validator Guide

Step-by-step guide for setting up a validator node on the Gnoland Sapphire testnet.

[![Sapphire](https://img.shields.io/badge/Chain-sapphire--1-blue)](https://sapphire.testnets.gno.land)
[![RPC](https://img.shields.io/badge/RPC-rpc.apollo--validator.eu-green)](https://rpc.apollo-validator.eu/gnoland/)

---

## Table of Contents

1. [Hardware Requirements](#1-hardware-requirements)
2. [Installation](#2-installation)
3. [Configuration](#3-configuration)
4. [Running the Node](#4-running-the-node)
5. [Sync](#5-sync)
6. [Validator Registration](#6-validator-registration)
7. [Troubleshooting](#7-troubleshooting)

---

## 1. Hardware Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| CPU | 4 cores | 8 cores |
| RAM | 8 GB | 16 GB |
| Storage | 200 GB SSD | 500 GB NVMe |
| Network | 100 Mbps | 1 Gbps |

**OS:** Ubuntu 22.04/24.04 (recommended)

**Software:**
- Go 1.22+ (for building from source)
- Docker (alternative installation method)
- Git
- jq (for API queries)

---

## 2. Installation

### Option A: Build from Source

```bash
# Install Go (if not installed)
GO_VERSION="1.22.4"
wget -q "https://go.dev/dl/go${GO_VERSION}.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go${GO_VERSION}.linux-amd64.tar.gz"
rm "go${GO_VERSION}.linux-amd64.tar.gz"
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> ~/.bashrc
source ~/.bashrc

# Clone and build
git clone https://github.com/gnolang/gno.git
cd gno && git checkout chain/sapphire
make -C gno.land install.gnoland install.gnokey

# Verify
gnoland version  # should show: chain/sapphire
gnokey --help
```

### Option B: Docker

```bash
# Build image
docker build --target gnoland -t gnoland:sapphire .

# Or pull from GHCR
docker pull ghcr.io/gnolang/gno/gnoland:chain-sapphire
```

### Option C: Prebuilt Binaries

Download from the [releases page](https://github.com/gnolang/gno/releases/tag/chain%2Fsapphire).

---

## 3. Configuration

### Initialize Config and Secrets

```bash
cd ~/gno
gnoland config init
gnoland secrets init
```

### Set Required Configuration

```bash
# Persistent peers (required — Sapphire uses persistent_peers, NOT seeds)
gnoland config set p2p.persistent_peers \
  "g10xll77gz6yzg43v9mdalj8360ng6sunt2vvvhf@seed-1.sapphire.testnets.gno.land:26656,g1gw2d7qsmrg06p204ty2qs8ygzd32t2c7p46te0@seed-2.sapphire.testnets.gno.land:26656"

# Chain-wide consensus settings (must match exactly)
gnoland config set application.prune_strategy syncable
gnoland config set consensus.timeout_commit 3s
gnoland config set consensus.peer_gossip_sleep_duration 10ms
gnoland config set p2p.flush_throttle_timeout 10ms

# Performance
gnoland config set mempool.size 10000
gnoland config set p2p.max_num_outbound_peers 40
```

### Set Node-Specific Configuration

```bash
# Your node name
gnoland config set moniker YOUR-MONIKER

# Your public IP for P2P
gnoland config set p2p.external_address "$(curl -s ifconfig.me):26656"

# Enable peer exchange
gnoland config set p2p.pex true
```

### Custom Ports (if 26xxx is in use)

```bash
gnoland config set p2p.laddr "tcp://0.0.0.0:17656"
gnoland config set rpc.laddr "tcp://127.0.0.1:17657"
gnoland config set telemetry.prometheus_listen_addr ":17660"
# Update external_address to match your P2P port
```

### Open Firewall

```bash
sudo ufw allow 26656/tcp comment "Sapphire P2P"
```

---

## 4. Running the Node

### Download Genesis

```bash
cd ~/gno
wget -O genesis.json https://github.com/gnolang/gno/releases/download/chain/sapphire/genesis.json

# Verify SHA256
shasum -a 256 genesis.json
# Expected: d511e0e5b767d4e53f5c1afeeea1bc61d2c7b2118146c820f1f3e4296f67498e
```

### Create systemd Service

```bash
sudo tee /etc/systemd/system/gnoland.service > /dev/null <<EOF
[Unit]
Description=Gnoland Sapphire Node
After=network-online.target
Wants=network-online.target

[Service]
User=$USER
WorkingDirectory=$HOME/gno
Environment=GNOROOT=$HOME/gno
Environment=HOME=$HOME
ExecStart=$(which gnoland) start \\
  --chainid sapphire-1 \\
  --genesis $HOME/gno/genesis.json \\
  --skip-genesis-sig-verification
Restart=on-failure
RestartSec=5s
LimitNOFILE=65535
StandardOutput=journal
StandardError=journal
SyslogIdentifier=gnoland

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now gnoland
```

> **IMPORTANT:** `--skip-genesis-sig-verification` is required. Some genesis transactions carry intentionally-invalidated signatures.

### Verify Node is Running

```bash
# Check service status
sudo systemctl status gnoland

# Check logs
journalctl -u gnoland -n 50 --no-pager

# Check sync status
curl -s http://localhost:26657/status | jq '.result.sync_info | {height: .latest_block_height, catching_up: .catching_up}'

# Check peers
curl -s http://localhost:26657/net_info | jq '.result.n_peers'
```

---

## 5. Sync

### From Genesis (slower)

The node will sync automatically from genesis. Monitor progress:

```bash
while true; do
  R=$(curl -s http://localhost:26657/status | jq -r '"\(.result.sync_info.latest_block_height) \(.result.sync_info.catching_up)"')
  echo "$(date +%H:%M:%S) $R"
  [ "${R#* }" = "false" ] && echo "SYNCED" && break
  sleep 30
done
```

### From Snapshot (fast, recommended)

```bash
# Install lz4
sudo apt install liblz4-tool -y

# Stop node
sudo systemctl stop gnoland

# Backup validator state (validators only)
cp ~/gno/gnoland-data/secrets/priv_validator_state.json ~/priv_validator_state.json.bak

# Download snapshot
wget -O snapshot.tar.lz4 https://snapshots.apollo-validator.eu/gnoland/snapshots/latest.tar.lz4

# Restore
rm -rf ~/gno/gnoland-data/db ~/gno/gnoland-data/wal
lz4 -d snapshot.tar.lz4 | tar -xf - -C ~/gno/gnoland-data/

# Restore validator state (validators only)
cp ~/priv_validator_state.json.bak ~/gno/gnoland-data/secrets/priv_validator_state.json

# Start node
sudo systemctl start gnoland

# Clean up
rm -v snapshot.tar.lz4
```

---

## 6. Validator Registration

### Prerequisites

- Node fully synced (`catching_up: false`)
- GNOT in your operator account (request from [faucet](https://sapphire.testnets.gno.land/faucet))

### Get Your Keys

```bash
# List your accounts
gnokey list

# Get consensus public key
# IMPORTANT: GNOROOT and --data-dir are required
GNOROOT=$HOME/gno gnoland secrets get validator_key --data-dir ~/gno/gnoland-data/secrets
```

### Register as Validator

```bash
gnokey maketx call \
  --pkgpath gno.land/r/gnops/valopers \
  --func Register \
  --args "YOUR-MONIKER" \
  --args "Your description (max 2048 chars)" \
  --args "data-center" \
  --args "YOUR_G1_OPERATOR_ADDRESS" \
  --args "YOUR_GPUB1_CONSENSUS_PUBKEY" \
  --gas-fee 1000000ugnot \
  --gas-wanted 50000000 \
  --chainid sapphire-1 \
  --remote https://rpc.apollo-validator.eu/gnoland/ \
  --broadcast \
  wallet
```

### Verify Registration

```bash
gnokey query vm/qrender -data "gno.land/r/gnops/valopers:YOUR_G1_ADDRESS" \
  -remote https://rpc.apollo-validator.eu/gnoland/
```

Or check on [GnoScan](https://gnoscan.io) or the [Valopers page](https://sapphire.testnets.gno.land/r/gnops/valopers).

### Join Active Set

After registration, you are a **candidate**. A GovDAO member must create and pass a proposal to add you to the active validator set.

---

## 7. Troubleshooting

### 0 Peers / Stuck at Height 0

**Cause:** `p2p.persistent_peers` not set.

**Fix:**
```bash
gnoland config set p2p.persistent_peers "g10xll77gz6yzg43v9mdalj8360ng6sunt2vvvhf@seed-1.sapphire.testnets.gno.land:26656,g1gw2d7qsmrg06p204ty2qs8ygzd32t2c7p46te0@seed-2.sapphire.testnets.gno.land:26656"
sudo systemctl restart gnoland
```

### AppHash Mismatch Crash Loop

**Cause:** Local database corruption.

**Fix:**
```bash
sudo systemctl stop gnoland
rm -rf ~/gno/gnoland-data/db ~/gno/gnoland-data/wal
# Restore from snapshot or resync from genesis
sudo systemctl start gnoland
```

### `go: command not found`

**Fix:**
```bash
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> ~/.bashrc
```

### Validator Not Signing Blocks

**Cause:** `priv_validator_state.json` has stale height from previous testnet.

**Fix:**
```bash
sudo systemctl stop gnoland
echo '{"height":"0","round":"0","step":0}' > ~/gno/gnoland-data/secrets/priv_validator_state.json
sudo systemctl start gnoland
```

> **IMPORTANT:** Format must be `{"height":"0","round":"0","step":0}` — height and round as strings, step as integer.

### GNOROOT Error

**Cause:** `GNOROOT` environment variable not set.

**Fix:**
```bash
export GNOROOT=$HOME/gno
# Or add to systemd service Environment=GNOROOT=$HOME/gno
```

---

## Quick Reference

| Item | Value |
|------|-------|
| Chain ID | `sapphire-1` |
| RPC | `https://rpc.apollo-validator.eu/gnoland/` |
| Genesis SHA256 | `d511e0e5b767d4e53f5c1afeeea1bc61d2c7b2118146c820f1f3e4296f67498e` |
| Snapshot | `https://snapshots.apollo-validator.eu/gnoland/snapshots/latest.tar.lz4` |
| Faucet | `https://sapphire.testnets.gno.land/faucet` |
| Valopers | `https://sapphire.testnets.gno.land/r/gnops/valopers` |
| GnoScan | `https://gnoscan.io` |
| Discord | `https://discord.gg/gnoland` |

---

## Links

- [Official Validator Docs](https://github.com/gnolang/gno/blob/chain/sapphire/misc/deployments/sapphire.gno.land/VALIDATOR.md)
- [Gnoland Website](https://gno.land)
- [GitHub](https://github.com/gnolang/gno)
- [Apollo Validator](https://apollo-validator.eu)

---

*Created by [Apollo Validator](https://apollo-validator.eu). Contributions welcome!*
