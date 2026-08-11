# Gnoland Sapphire State Sync Guide

Fast synchronization methods for Gnoland Sapphire testnet nodes.

[![Sapphire](https://img.shields.io/badge/Chain-sapphire--1-blue)](https://sapphire.testnets.gno.land)
[![RPC](https://img.shields.io/badge/RPC-rpc.apollo--validator.eu-green)](https://rpc.apollo-validator.eu/gnoland/)

---

## Overview

Gnoland Sapphire supports fast-sync via snapshots:

| Method | Speed | Data Size | Use Case |
|--------|-------|-----------|----------|
| **Snapshot** | ~2-5 min | ~150 MB compressed | Recommended for most users |
| **Genesis Sync** | ~20-40 min | Full chain | For archival nodes |

> **Note:** Traditional Tendermint state sync (`statesync.enable`) is not available in Gnoland TM2. Use snapshots instead.

---

## Method 1: Snapshot (Recommended)

### Quick Start

```bash
# 1. Stop the node
sudo systemctl stop gnoland

# 2. Download snapshot
wget -O snapshot.tar.lz4 https://snapshots.apollo-validator.eu/gnoland/snapshots/latest.tar.lz4

# 3. Restore
rm -rf ~/gno/gnoland-data/db ~/gno/gnoland-data/wal
lz4 -d snapshot.tar.lz4 | tar -xf - -C ~/gno/gnoland-data/

# 4. Start the node
sudo systemctl start gnoland
```

### Detailed Steps

#### Step 1: Install lz4

```bash
sudo apt install liblz4-tool -y
```

#### Step 2: Stop the Node

```bash
sudo systemctl stop gnoland
```

#### Step 3: Backup Validator State (Validators Only)

```bash
# Only if you're running a validator
cp ~/gno/gnoland-data/secrets/priv_validator_state.json ~/priv_validator_state.json.bak
```

#### Step 4: Download Snapshot

```bash
# Get latest snapshot info
curl -s https://snapshots.apollo-validator.eu/api/gnoland/snapshots/latest | jq

# Download
wget -O snapshot.tar.lz4 https://snapshots.apollo-validator.eu/gnoland/snapshots/latest.tar.lz4
```

#### Step 5: Restore from Snapshot

```bash
# Remove old data
rm -rf ~/gno/gnoland-data/db ~/gno/gnoland-data/wal

# Extract snapshot (note: lz4, NOT zstd)
lz4 -d snapshot.tar.lz4 | tar -xf - -C ~/gno/gnoland-data/

# Verify extraction
ls ~/gno/gnoland-data/db/
```

#### Step 6: Restore Validator State (Validators Only)

```bash
# Only if you're running a validator
cp ~/priv_validator_state.json.bak ~/gno/gnoland-data/secrets/priv_validator_state.json
```

#### Step 7: Start the Node

```bash
sudo systemctl start gnoland

# Monitor logs
journalctl -u gnoland -f
```

#### Step 8: Verify Sync

```bash
# Check if synced
curl -s http://localhost:26657/status | jq '.result.sync_info | {height: .latest_block_height, catching_up: .catching_up}'

# Check peers
curl -s http://localhost:26657/net_info | jq '.result.n_peers'
```

#### Step 9: Clean Up

```bash
rm -v snapshot.tar.lz4
```

---

## Method 2: Genesis Sync

If you prefer to sync from genesis (slower but more thorough):

### Step 1: Start the Node

```bash
gnoland start \
  --chainid sapphire-1 \
  --genesis ~/gno/genesis.json \
  --skip-genesis-sig-verification
```

### Step 2: Monitor Progress

```bash
while true; do
  R=$(curl -s http://localhost:26657/status | jq -r '"\(.result.sync_info.latest_block_height) \(.result.sync_info.catching_up)"')
  echo "$(date +%H:%M:%S) $R"
  [ "${R#* }" = "false" ] && echo "SYNCED" && break
  sleep 30
done
```

### Step 3: Optimize with Persistent Peers

Add peers to config for faster sync:

```bash
gnoland config set p2p.persistent_peers "g10xll77gz6yzg43v9mdalj8360ng6sunt2vvvhf@seed-1.sapphire.testnets.gno.land:26656,g1gw2d7qsmrg06p204ty2qs8ygzd32t2c7p46te0@seed-2.sapphire.testnets.gno.land:26656"
```

---

## Troubleshooting

### Snapshot restore fails

**Cause:** Snapshot is corrupted or incomplete download.

**Fix:** Re-download the snapshot.

### Node crashes after restore with "AppHash mismatch"

**Cause:** Database corruption during snapshot extraction.

**Fix:**
```bash
rm -rf ~/gno/gnoland-data/db ~/gno/gnoland-data/wal
# Re-download and restore snapshot
```

### 0 peers after restore

**Cause:** Seeds not configured or not reachable.

**Fix:**
```bash
gnoland config set p2p.persistent_peers "g10xll77gz6yzg43v9mdalj8360ng6sunt2vvvhf@seed-1.sapphire.testnets.gno.land:26656,g1gw2d7qsmrg06p204ty2qs8ygzd32t2c7p46te0@seed-2.sapphire.testnets.gno.land:26656"
sudo systemctl restart gnoland
```

### Node stuck at height 0

**Cause:** Genesis verification failed.

**Fix:** Ensure you're using `--skip-genesis-sig-verification`:
```bash
gnoland start --chainid sapphire-1 --genesis ~/gno/genesis.json --skip-genesis-sig-verification
```

### Validator not signing after restore

**Cause:** `priv_validator_state.json` has stale height.

**Fix:**
```bash
sudo systemctl stop gnoland
echo '{"height":"0","round":"0","step":0}' > ~/gno/gnoland-data/secrets/priv_validator_state.json
sudo systemctl start gnoland
```

---

## API Reference

### Get Latest Snapshot

```bash
curl -s https://snapshots.apollo-validator.eu/api/gnoland/snapshots/latest | jq
```

Response:
```json
{
  "height": 86000,
  "size": "150M",
  "created": "2026-08-11T14:00:00Z",
  "file": "gnoland-sapphire-86000.tar.lz4",
  "checksum": "sha256:...",
  "network": "sapphire-1"
}
```

### Download Snapshot

```bash
wget https://snapshots.apollo-validator.eu/gnoland/snapshots/latest.tar.lz4
```

---

## Links

- [Snapshot Service](https://snapshots.apollo-validator.eu/gnoland/)
- [Public RPC](https://rpc.apollo-validator.eu/gnoland/)
- [Peers List](peers.md)
- [Validator Guide](guide.md)
- [Apollo Validator](https://apollo-validator.eu)

---

*Created by [Apollo Validator](https://apollo-validator.eu)*
