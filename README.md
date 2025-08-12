# Cysic Network Node Setup Guide

**Authored by Rachit Yadav**

> This guide explains how to run Prover, Verifier, and Validator nodes on the Cysic Network (Testnet Phase 3). Follow each section carefully for a smooth deployment.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Prover Node Setup](#prover-node-setup)
3. [Verifier Node Setup](#verifier-node-setup)
4. [Validator Node Setup](#validator-node-setup)
5. [Troubleshooting & Maintenance](#troubleshooting--maintenance)


---

## 1. Prerequisites

Before you begin, ensure you have:

* Cysic Phase3 invite code and a wallet address
* Access to a Linux / macOS / Windows system (with sudo/admin privileges where required)
* Stable internet connection
* Basic familiarity with the terminal (bash / PowerShell)

---

## 2. Prover Node Setup

Prover nodes generate zero-knowledge proofs and earn credits. They require high hardware specs and some DevOps experience.

### Hardware Requirements

* **Scroll Proof (highly demanding):** Ubuntu, **≥256 GB RAM**, 32-core CPU, GPU ≥20 GB VRAM
* **ETH Proof (less than Scroll):** Ubuntu, **≥32 GB RAM**, 8-core CPU, GPU ≥16 GB VRAM

> Prover nodes are resource intensive — ensure proper cooling, UPS, and monitoring.

### Step-by-step

1. **Connect Wallet**

   * Visit the Cysic Phase3 website, join Testnet Phase 3, connect your wallet, sign the message, and enter your invite code.
   * Copy your wallet address (used for rewards).

2. **Get RPC Endpoint (for ETH proof)**

   * Register at a provider (e.g., Alchemy), create a new app, and copy the RPC URL for later use.

3. **Download & run setup script (Linux example)**

```bash
# Replace 0x-Fill-in-your-reward-address-here with your wallet address
curl -L https://github.com/cysic-labs/cysic-phase3/releases/download/v1.0.0/setup_linux.sh \
  > ~/setup_linux.sh && bash ~/setup_linux.sh 0x-Fill-in-your-reward-address-here
```

* During setup, if prompted about `eth_proof` dependencies, choose `y` to install.

4. **Start the Prover**

```bash
cd ~/cysic-prover/ && bash start.sh
```

* Wait several minutes. A log line like `send heartbeat to server` indicates a successful launch.

5. **Backup Mnemonic**

* Your mnemonic is stored at `~/.cysic/assets/`. Back it up securely (offline preferred).

---

## 3. Verifier Node Setup

Verifier nodes validate proofs produced by Prover nodes. Hardware needs are modest compared to Provers.

### Hardware Requirements

* Minimal — a modern Linux, macOS, or Windows machine is sufficient.

### Step-by-step

1. **Connect Wallet**

   * Same as Prover (connect, sign, enter invite code, copy wallet address).

2. **Download & run setup script**

**Linux**

```bash
# Replace 0x-Fill-in-your-reward-address-here with your wallet address
curl -L https://github.com/cysic-labs/cysic-phase3/releases/download/v1.0.0/setup_linux.sh \
  > ~/setup_linux.sh && bash ~/setup_linux.sh 0x-Fill-in-your-reward-address-here
cd ~/cysic-verifier/ && bash start.sh
```

**macOS**

```bash
curl -L https://github.com/cysic-labs/cysic-phase3/releases/download/v1.0.0/setup_mac.sh \
  > ~/setup_mac.sh && bash ~/setup_mac.sh 0x-Fill-in-your-reward-address-here
cd ~/cysic-verifier/ && bash start.sh
```

**Windows (PowerShell)**

```powershell
cd $env:USERPROFILE
Invoke-WebRequest -Uri "https://github.com/cysic-labs/cysic-phase3/releases/download/v1.0.0/setup.ps1" -OutFile "setup_win.ps1"
.\setup_win.ps1 -CLAIM_REWARD_ADDRESS "0x-Fill-in-your-reward-address-here"
cd $env:USERPROFILE\cysic-verifier
.\start.ps1
```

* After a few minutes, look for `send heartbeat to server` to confirm the verifier is running.

3. **Backup Mnemonic**

* Verifier mnemonic typically at `~/.cysic/keys/` — back it up securely.

---

## 4. Validator Node Setup

Validators secure the network and participate in consensus. Testnet Phase III is permissioned, so follow the whitelisting steps.

### Hardware & Software Requirements

| Resource       | Minimum       | Recommended |
| -------------- | ------------- | ----------- |
| CPU            | 4 cores       | 8 cores     |
| Memory         | 8 GB RAM      | 16 GB RAM   |
| Storage        | 200 GB SSD    | 500 GB SSD  |
| OS             | Ubuntu 20.04+ | —           |
| Docker         | >= 20.10.0    | —           |
| Docker Compose | >= 1.29.0     | —           |
| Git            | Latest        | —           |

**Required Ports:** `26657`, `26656`, `8545`, `8546`, `1317`, `9090`, `6065` (metrics)

> Ensure these ports are open and reachable from the network as required.

### Step-by-step

1. **Download & extract configuration files**

```bash
mkdir -p /data && cd /data
wget https://statics.prover.xyz/testnet_node.tar.gz
tar -zxvf testnet_node.tar.gz
```

2. **Download data snapshot**

```bash
wget https://statics.prover.xyz/data.tar.gz
rm -rf node/cysicmintd/data
tar -zxvf data.tar.gz -C node/cysicmintd
```

3. **Pull Docker images**

```bash
docker-compose pull
```

4. **Configure & start node**

* Modify `docker-compose.yml` as needed (volumes, ports, env vars).

**IMPORTANT:** Share your node exit IP with the Cysic team for whitelisting.

Start the node:

```bash
docker-compose up -d node
docker-compose logs -f node
```

5. **Verify node status**

```bash
curl http://localhost:26657/status
curl http://localhost:26657/abci_info
```

6. **Generate & backup validator key**

```bash
docker exec -it node bash
./cysicmintd keys add validator-xxx --home ./cysicmint --keyring-backend test
# Write down your mnemonic securely!
# Obtain node ID and public key:
./cysicmintd tendermint show-node-id --home ./cysicmint --keyring-backend test
./cysicmintd tendermint show-validator --home ./cysicmint --keyring-backend test
# Export private key:
./cysicmintd keys export validator-xxx --home ./cysicmint --keyring-backend test
```

7. **Stake & create validator**

* Acquire testnet staking tokens from the Cysic team.

```bash
./cysicmintd tx staking create-validator \
  --amount=10000000000000000000CGT \
  --moniker="your-validator-name" \
  --details="your-validator-description" \
  --chain-id cysicmint_9001-1 \
  --commission-rate="0.05" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation=1000000000000000000 \
  --pubkey=$(./cysicmintd tendermint show-validator --home ./cysicmint) \
  --from validator-xxx \
  --home=./cysicmint \
  --keyring-backend test \
  --gas auto \
  --gas-adjustment 1.2 \
  --gas-prices=300000CYS \
  --yes
```

8. **Delegate & monitor**

* Delegate additional tokens if required and monitor health endpoints, logs, and sync status.

---

## 5. Troubleshooting & Maintenance

* **Sync issues:** Check internet connectivity, firewall/ports, and configuration. Resync using snapshots if necessary.
* **Not earning/staking rewards:** Confirm active set membership, token balance, and that you are not jailed.
* **Docker/container failures:** Review `docker-compose logs` and container logs; verify ports and resource limits.
* **Emergency recovery:** Keep regular backups of keys and data; if needed, restore from snapshot and re-register with the Cysic team.

---

##

Written by **Rachit Yadav**.

---
