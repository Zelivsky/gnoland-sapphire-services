# Gnoland Sapphire Peers

Last updated: (auto-updated every 6 hours)

## How we collect peers

Every 6 hours, peer addresses are gathered from our node and public RPCs.
Each peer is then TCP-probed to confirm it is actually reachable.
Only peers that pass this check are included in the list below.

## Adding peers to your node

```bash
PEERS="(auto-generated list)"

cd ~/gno
gnoland config set p2p.persistent_peers "$PEERS"
```

Sapphire seeds:

```bash
gnoland config set p2p.persistent_peers "g10xll77gz6yzg43v9mdalj8360ng6sunt2vvvhf@seed-1.sapphire.testnets.gno.land:26656,g1gw2d7qsmrg06p204ty2qs8ygzd32t2c7p46te0@seed-2.sapphire.testnets.gno.land:26656"
```
