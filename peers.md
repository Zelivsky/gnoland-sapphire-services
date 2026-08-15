# Gnoland Sapphire Peers

Last updated: 2026-08-15 15:41 UTC | Height: 183,123 | Verified: 51/64

## How we collect peers

Every 6 hours, peer addresses are gathered from our node and public RPCs.
Each peer is then TCP-probed to confirm it is actually reachable.
Only peers that pass this check are included in the list below.

## Adding peers to your node

```bash
PEERS="g18z69nce5maf7zhrcxcahlr80e3q8ynqez8023v@43.202.214.151:26656,g1nazs9uqecsxszksmsjjjy5rc8l849hgvg9a03q@135.181.17.54:26656,g188678w86qyu4garey40eslcuw0m6n4knqxpcdt@109.123.243.186:26676,g18psgsr5f26l57qwpq2an07q97ws6l5243z986q@0.0.0.0:26656,g148rf73cvu3456wgtpvadcvjeyxnu77hj77rmxr@135.181.78.21:17656,g1ehlkj5cmghkcthg5sc65cnqmf8rdds20eunaef@37.27.63.150:21956,g1wx5cm8633h4txwrc9tc2fvfszdz2xlnhpwnm0r@65.108.65.23:56656,g1x38lw7g2p6l5ne2vpjwm3x8cjwwjqd2hd2dck4@185.16.39.172:56656,g1203vcgvnvw4xq33a82qda9576u85n8c3xejh7p@5.9.8.148:30804,g152gwf835peylsntycry3gfgyzd6d0s4d9pyndv@88.198.46.55:58656,g1pnxlnwyxnthyrh8f8akmlreup0jxfff4r0s5r0@0.0.0.0:26656,g1g7ud4knkm58hzl3hpuh7wac2tvdywhx4hah5vz@163.172.33.181:26656,g1d7pksd7luqhk5sm5zmxwkl8w2cge6n7wz9llnh@135.181.227.236:37656,g1qwawm46wy4e76fusd056yataqxxr0t4hhkndmx@14.225.215.118:26656,g1uh7esgx5dl7rax9d04unwek6zk9ym0h0uhsfm4@0.0.0.0:26656,g1ff2mz7ywqpct7nngjljg722pz2wmve3j2h657h@85.195.116.219:26680,g1h0xy387s7zxfmnw07hfs3a8vxndmtlynvqjak7@95.216.242.118:60656,g1gvsc39264a6mq4v56qpjpte32klvqrh93swd9a@162.250.190.214:26656,g1wfu406mz57cv2cq68agrkefd4g6pw0ew4kyvlu@65.109.22.211:54656,g1xeu7vvyh9e9tpqtnyn3gn84wzj4yd5k9zdltk9@176.9.111.199:26656,g1kres5ar5mcqrannxnxu02q8q5wydt0aezqspe2@100.62.129.64:26656,g1gw2d7qsmrg06p204ty2qs8ygzd32t2c7p46te0@34.246.18.165:26656,g1cw4ajr026320c3a6rfm2d77tr94yglh5namap9@65.108.198.182:22656,g1s76trwsy82drua7d886gzaw9y7nca0fqn6rsem@149.50.96.58:41676,g1kxe342hr7fm2687fecsrvfc0mnkynca8h4scxr@37.27.235.144:54656,g1we3jtgpknz5sxdgkmsgxwk6gytsvzl983crppg@95.217.43.94:26656,g1xgsjdqrumr5cjutusf54635qg4p86zaa059wf6@0.0.0.0:26656,g1zt50ekaaqvsh0e7gvy2x2g3x955ust965wgdhc@135.181.21.38:22656,g17j4rcqumac8vr0yxtysk8cydhlyt032n09fssj@0.0.0.0:26656,g17t7vlg7hjvsldqj06zenkxpktzntkp62thg9a7@65.109.106.214:55666,g1nhj7jwvlrukcl5uc4ptt4ls66jq3ge8j6afy6k@0.0.0.0:26656,g1d7r34aznev47puvdgkr9dump3dhl3mdl7n7me4@208.76.222.122:26656,g1tsd2mxdarwk8q59jgghu3ey947648tgc6mwuvp@152.53.48.29:27656,g1k2qaq62j4ddnfseq8t2txhd26wc0zux86gpvxt@65.108.237.96:26656,g1277y5mdwmg68cx7ryx3txdwdmjl6z5p72xfqcd@44.199.26.207:26656,g1hwujpr37atlx5k63v9mms7e2p66txs5mz3knda@192.155.100.132:26656,g14x8wxfxv7p2qs2j6tnkvdauamwr30glrr602x0@198.244.200.116:54656,g17mkvewp957pvcu2n5m7y2cpmpl7fdzlajhqm53@149.202.68.156:26676,g1s9xs348chr0dfpugzykwjupwxduz6e02mz55xx@0.0.0.0:26656,g1zrjayky7rz2e3g469ue2cj8uu5qffak7sthql9@217.76.53.126:54656,g10xll77gz6yzg43v9mdalj8360ng6sunt2vvvhf@54.224.10.49:26656,g1z700tjus883ku3y282pndyluvjavxh5zqe9xya@54.155.249.122:26656,g1nrmvrryw6dqkwjynw6lm75532p5keh2uwmpqmh@0.0.0.0:26656,g1yr6l8qz095yz7dlvfs6cjfw3s026ksxwx7wgx5@1.36.239.11:28756,g1r6jgch2ax4kvqrtju8v3pgk5pmwe9myjle9uws@65.108.111.225:36656,g1tuuuxxn8rjlr860hm4qxm4tjv77fl2vffayga3@100.59.119.139:26656,g1q0uufaj5ug8dfkf85x5640rthh25j8l7jelfqc@135.181.232.179:26656,g1mwh2ll0pp5dpjlzfnywr5cjs6ws885gnxvyczv@148.251.2.253:56656,g1edj9r6sfus7n680s4kyq4wdn746wz7qmnchhy4@0.0.0.0:26656,g1e3pftctakyx58mzapqs4syc4h0jcxwtnraq00c@185.248.24.16:26656,g1w44kfhv9a4ruqx70jxqmykychre3ffpnfje8h7@159.195.47.190:60656"

cd ~/gno
gnoland config set p2p.persistent_peers "$PEERS"
```

Sapphire seeds:

```bash
gnoland config set p2p.persistent_peers "g10xll77gz6yzg43v9mdalj8360ng6sunt2vvvhf@seed-1.sapphire.testnets.gno.land:26656,g1gw2d7qsmrg06p204ty2qs8ygzd32t2c7p46te0@seed-2.sapphire.testnets.gno.land:26656"
```