# How to Run a Node in Cysic

*Credit: Rachit Yadav*

---

## Overview

This guide shows a practical, end-to-end way to run a Cysic blockchain node (full node / validator). It covers system requirements, installation, initialization, syncing, running as a service, validator setup, maintenance, and troubleshooting.

---

## 1. Prerequisites

### Hardware (recommended)

* CPU: 4–8 cores (8+ preferred for validators)
* RAM: 16 GB+
* Disk: 500 GB SSD (NVMe preferred)
* Network: Stable internet (>= 100 Mbps recommended)
* OS: Linux (Ubuntu 20.04+ recommended)

### Software

* `git`, `curl`, `jq`, `build-essential`
* Go (1.19+)
* `systemd` (for service)

---

## 2. System preparation

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git curl jq build-essential
```

Set locale and limits (optional but recommended):

```bash
sudo tee -a /etc/security/limits.conf <<'EOF'
* soft nofile 65536
* hard nofile 65536
EOF
```

---

## 3. Install Go

```bash
# change version if needed
GO_VER=1.19.5
wget https://golang.org/dl/go${GO_VER}.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go${GO_VER}.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> ~/.profile
source ~/.profile
go version
```

---

## 4. Clone and build Cysic

```bash
# choose a working dir
cd $HOME
git clone https://github.com/Cysic-project/cysic-chain.git
cd cysic-chain
make install
# or follow repo's build command if different (eg: go build ./...)
```

After `make install`, verify binaries like `cysicd` (daemon) and `cysiccli` exist in `$HOME/go/bin` or `/usr/local/bin`.

---

## 5. Initialize the node

```bash
cysicd config keyring-backend file
cysicd config chain-id cysic-mainnet
cysicd init "<your-moniker>"
```

This creates `~/.cysicd` (config and data).

---

## 6. Obtain genesis & peers

Place the correct `genesis.json` in the config folder. If you have an official genesis file, copy it to `~/.cysicd/config/genesis.json`.

Configure peers/persistent\_peers in `~/.cysicd/config/config.toml` and `~/.cysicd/config/addrbook.json` with known good peers (use trusted peer list from the project).

---

## 7. Fast sync options (statesync / snapshot)

To avoid months of syncing, use statesync or download a snapshot (if project provides):

**Statesync example:** edit `~/.cysicd/config/config.toml` and set `enable`/`rpc_servers`/`trust_hash`/`trust_height` per network instructions.

**Snapshot:** stop node, extract snapshot to `~/.cysicd/data`, then restart.

---

## 8. Run the node (manual)

```bash
cysicd start 2>&1 | tee ~/cysicd.log
# or run in background
nohup cysicd start &
```

Check logs:

```bash
tail -f ~/.cysicd/logs/node.log
journalctl -u cysicd.service -f
```

---

## 9. Create wallet / keys

```bash
cysicd keys add walletname
# backup the mnemonic securely
```

---

## 10. Run as systemd service

Create `/etc/systemd/system/cysicd.service` with:

```ini
[Unit]
Description=Cysic Daemon
After=network-online.target

[Service]
User=%u
ExecStart=/usr/local/bin/cysicd start
Restart=on-failure
RestartSec=5
LimitNOFILE=65536
Environment="HOME=/home/youruser"

[Install]
WantedBy=multi-user.target
```

Reload & enable:

```bash
sudo systemctl daemon-reload
sudo systemctl enable cysicd
sudo systemctl start cysicd
sudo systemctl status cysicd
```

Replace `/usr/local/bin/cysicd` and `youruser` with actual paths/user.

---

## 11. Become a validator (basic steps)

1. Ensure your node is fully synced and has enough bonded tokens.
2. Create a validator transaction:

```bash
cysicd tx staking create-validator \
  --amount 1000000cysic \
  --pubkey "$(cysicd tendermint show-validator)" \
  --moniker "<your-moniker>" \
  --chain-id cysic-mainnet \
  --from walletname
```

3. Broadcast and wait for inclusion.

---

## 12. Common commands

```bash
# status
cysicd status
# query sync
cysicd status 2>&1 | jq .SyncInfo
# show peers
cysicd tendermint show-peer
# show validator key
cysicd tendermint show-validator
# wallet balance
cysicd query bank balances <address>
```

---

## 13. Troubleshooting

* **Node not syncing**: check peers, RPC endpoints, time drift (use `ntp`), and disk IO.
* **High CPU / disk**: enable pruning, reduce cache in `config.toml`, or increase hardware.
* **State mismatch on restart**: try `unsafe_reset_all` **only** if you have a snapshot or state to restore.

```bash
cysicd unsafe-reset-all --keep-addr-book
```

---

## 14. Backups & upgrades

* Backup `~/.cysicd/config/` and `~/.cysicd/data/priv_validator_state.json` regularly.
* For upgrades: follow on-chain upgrade instructions; stop service, replace binary, restart.

---

## 15. Security best-practices

* Keep mnemonic and `priv_validator_key.json` offline/backed-up.
* Use firewall to allow only necessary ports (p2p, rpc if needed) and restrict admin access.
* Run node under a non-root user.

---

## 16. Monitoring & alerts

* Use Prometheus + Grafana if exporters are available.
* Simple alerts: monitor process uptime (`systemd`), disk usage, and peers.

---

## 17. Final tips

* Start with a testnet before mainnet.
* Join project channels to get trusted peers and snapshots.
* Keep binaries and Go up to date, and always read release notes before upgrading.

---
