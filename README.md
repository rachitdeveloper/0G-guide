
<div align="center">

# 🚀 **0G Storage Node Complete Setup Guide** 🚀

### *by Satyam Jha*

</div>

---

## 🧩 **Introduction**

Storage Nodes are the foundation of 0G's decentralized storage—while validator nodes secure the chain, storage nodes keep critical data persistent, available, and distributed. Running a storage node means you’re actively supporting the network’s long-term AI dataset and model hosting capabilities, contributing to resilience and decentralization.

---

## 🖥️ **System & Hardware Requirements**

> ![System Requirements](https://github.com/user-attachments/assets/6f06b201-c4b1-4671-b3e1-bf1e49cb5182)

---

## ⚡ **Initial Setup Steps**

**Before You Begin:**

* Add the 0G Galileo Testnet: [Instructions here](https://docs.0g.ai/run-a-node/testnet-information)
* Request testnet tokens: [0G Faucet](https://faucet.0g.ai/)

---

## 🔧 **Install Required Dependencies**

Update & upgrade:

```bash
sudo apt-get update && sudo apt-get upgrade -y
```

Essential packages:

```bash
sudo apt install curl iptables build-essential git wget lz4 jq make protobuf-compiler cmake gcc nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev screen ufw -y
```

---

### **Install Rust**

```bash
curl https://sh.rustup.rs -sSf | sh
```

```bash
source $HOME/.cargo/env
```

Check Rust version:

```bash
rustc --version
```

---

### **Install Go**

```bash
wget https://go.dev/dl/go1.24.3.linux-amd64.tar.gz && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf go1.24.3.linux-amd64.tar.gz && \
rm go1.24.3.linux-amd64.tar.gz && \
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc && \
source ~/.bashrc
```

Check Go version:

```bash
go version
```

---

## 📂 **Clone & Build Storage Node**

Clone the repo:

```bash
git clone https://github.com/0glabs/0g-storage-node.git
```

Switch to the latest release & update submodules:

```bash
cd 0g-storage-node && git checkout v1.1.0 && git submodule update --init
```

Compile in release mode:

```bash
cargo build --release
```

---

## ⚙️ **Configuration**

Clear previous config (if any):

```bash
rm -rf $HOME/0g-storage-node/run/config.toml
```

Download the config file:

```bash
curl -o $HOME/0g-storage-node/run/config.toml https://raw.githubusercontent.com/Mayankgg01/0G-Storage-Node-Guide/main/config.toml
```

**Add your wallet's private key:**
*Important: Exclude the "0x" prefix!*

Open the config for editing and enter your key under `miner_key`:

```bash
nano $HOME/0g-storage-node/run/config.toml
```

> ![Config Example](https://github.com/user-attachments/assets/a513812f-177e-4a74-83a9-1548c98f4556)

---

### **Custom RPC (Optional)**

1. Find available RPC endpoints: [Astrostake RPC List](https://www.astrostake.xyz/0g-status)
2. Update the RPC in your config:

> ![RPC Example](https://github.com/user-attachments/assets/44b682a5-45ce-4fc8-8c3a-7f2355f3b9ac)

---

## 🛡️ **Systemd Service Setup**

Create a service file:

```bash
sudo tee /etc/systemd/system/zgs.service > /dev/null <<EOF
[Unit]
Description=ZGS Node
After=network.target

[Service]
User=$USER
WorkingDirectory=$HOME/0g-storage-node/run
ExecStart=$HOME/0g-storage-node/target/release/zgs_node --config $HOME/0g-storage-node/run/config.toml
Restart=on-failure
RestartSec=10
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

Reload & enable:

```bash
sudo systemctl daemon-reload
sudo systemctl enable zgs
sudo systemctl start zgs
```

---

## 🔎 **Monitoring & Logs**

Check service status:

```bash
sudo systemctl status zgs
```

> ![Service Status](https://github.com/user-attachments/assets/3b01ab3f-8d43-43b3-9bf1-b2a8e870e1fe)

Follow logs live:

```bash
tail -f ~/0g-storage-node/run/log/zgs.log.$(TZ=UTC date +%Y-%m-%d)
```

Check sync and block status:

```bash
while true; do response=$(curl -s -X POST http://localhost:5678 -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","method":"zgs_getStatus","params":[],"id":1}'); logSyncHeight=$(echo $response | jq '.result.logSyncHeight'); connectedPeers=$(echo $response | jq '.result.connectedPeers'); echo -e "logSyncHeight: \033[32m$logSyncHeight\033[0m, connectedPeers: \033[34m$connectedPeers\033[0m"; sleep 5; done
```

> ![Sync Example](https://github.com/user-attachments/assets/ab97078b-2c2a-4328-aace-bc94982ab802)

---

## 🛑 **Stop & Remove Node**

Stop & disable:

```bash
sudo systemctl stop zgs
sudo systemctl disable zgs
sudo rm /etc/systemd/system/zgs.service
rm -rf $HOME/0g-storage-node
```

---

## 🔗 **Useful Links**

* Block explorer: [Explorer 1](https://chainscan-galileo.bangcode.id/) | [Explorer 2](https://chainscan-galileo.0g.ai/)
* Miner details (append wallet address): [Miner Details](https://storagescan-galileo.0g.ai/miner/)

---

## ❓ **FAQ: Running on a Local Device & Restarting**

* The process is **identical** for local PC and VPS—just follow all steps above.
* To restart your node the next day, simply run:

```bash
sudo systemctl restart zgs
```

---

<div align="center">

# ⚡ **Fast Sync: Download Snapshot** ⚡

</div>

To sync from block `3562700`:

1. **Stop your node & delete the flow DB:**

   ```bash
   sudo systemctl stop zgs
   ```
   ```
   rm -rf $HOME/0g-storage-node/run/db/flow_db
   ```

2. **Download and extract:**

   ```bash
   wget https://github.com/Mayankgg01/0G-Storage-Node-Guide/releases/download/v1.0/flow_db.tar.gz \
     -O $HOME/0g-storage-node/run/db/flow_db.tar.gz && \
     tar -xzvf $HOME/0g-storage-node/run/db/flow_db.tar.gz -C $HOME/0g-storage-node/run/db/
   ```

3. **Restart the node:**

   ```bash
   sudo systemctl restart zgs
   ```

That’s it—your node will sync quickly from the latest snapshot!

---

<div align="center">

# 🧹 **How to Clear Data & Start Fresh**

</div>

If your VPS storage is full or you want a clean reset:

```bash
sudo systemctl stop zgs
sudo systemctl disable zgs
sudo rm /etc/systemd/system/zgs.service
rm -rf $HOME/0g-storage-node
```

Then **restart from the repository clone step** and don’t forget the latest snapshot for fast sync!

---

<div align="center">

# 🚀 **Upgrading to v1.1.0 Release**

</div>

1. **Stop service:**

   ```bash
   sudo systemctl stop zgs
   ```

2. **Reset, fetch, and switch repo:**

   ```bash
   cd ~/0g-storage-node
   git reset --hard
   git clean -fd
   git fetch --all
   git checkout v1.1.0
   git submodule update --init --recursive
   ```

3. **Build new version:**

   ```bash
   sudo apt-get install protobuf-compiler
   cargo build --release
   ```

4. **Reapply config:**
   [Configuration instructions](https://github.com/Mayankgg01/0G-Storage-Node-Guide?tab=readme-ov-file#set-configrations)

5. **Restart node:**

   ```bash
   sudo systemctl start zgs
   ```

6. **Re-sync from snapshot for speed:**
   [Snapshot guide](https://github.com/Mayankgg01/0G-Storage-Node-Guide?tab=readme-ov-file#-download-snapshot-for-faster-sync-)

---

## 💬 **Need Help?**

* Telegram: [Join the 0G Community](https://telegram.me/cryptogg)
* Issues? [Open an issue on GitHub](https://github.com/Mayankgg01/0G-Storage-Node-Guide/issues)
* DM me (Satyam Jha) on Telegram for direct help!

---

<div align="center">

**Happy coding! You’re now powering the future of decentralized storage.**
*— Guide by Satyam Jha*

</div>

---

Let me know if you want this in a different format (PDF, HTML, etc.), or want images hosted somewhere else!
