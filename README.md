# Zenrock-Installation

#### **1. Prerequisites**

Before starting the installation, make sure you have the following:
- A Linux server or machine.
- Root or sudo privileges.
- Go installed (version 1.19+ recommended).
- At least 4 GB of RAM and 100 GB of free disk space.

##### **Install required dependencies:**

```bash
sudo apt update
sudo apt install -y build-essential git curl jq
```

##### **Install Go (if not already installed):**

Download and install Go by following these commands:

```bash
wget https://go.dev/dl/go1.19.8.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.19.8.linux-amd64.tar.gz
```

Now, add Go to your path:

```bash
echo "export PATH=$PATH:/usr/local/go/bin" >> ~/.bashrc
source ~/.bashrc
```

Verify the Go installation:

```bash
go version
```

You should see the Go version printed out.

#### **2. Clone the Zenrock Repository**

Next, we need to clone the Zenrock repository from GitHub:

```bash
cd $HOME
git clone https://github.com/zenrocknetwork/zenrock.git
cd zenrock
```

#### **3. Build the Zenrock Binary**

To build the node binary, run the following commands:

```bash
make install
```

This command will compile the node software and install it.

#### **4. Initialize the Node**

Before starting the node, you need to initialize it. Replace `YOUR_MONIKER` with a custom name for your node:

```bash
zenrockd init YOUR_MONIKER --chain-id zenrock-testnet
```

#### **5. Configure the Node**

Edit the node configuration. Replace `seeds` with the list of seed nodes for the testnet.

```bash
seeds="SEED_NODE_ADDRESSES"
peers="PEER_NODE_ADDRESSES"
```

Run the following to update your configuration:

```bash
sed -i -e "s|^seeds *=.*|seeds = \"$seeds\"|" ~/.zenrock/config/config.toml
sed -i -e "s|^persistent_peers *=.*|persistent_peers = \"$peers\"|" ~/.zenrock/config/config.toml
```

#### **6. Download the Genesis File**

To join the testnet, download the genesis file:

```bash
curl -Ls https://snapshots.kjnodes.com/zenrock-testnet/genesis.json > $HOME/.zenrock/config/genesis.json
```

Validate the genesis file:

```bash
sha256sum $HOME/.zenrock/config/genesis.json
```

#### **7. Set Up State Sync**

If you want to enable state sync (optional but recommended), edit the `config.toml` file:

```bash
sed -i -e "s|^enable *=.*|enable = true|" $HOME/.zenrock/config/config.toml
```

#### **8. Start the Node**

Finally, start your node by running:

```bash
zenrockd start
```

You can monitor your node's logs to ensure it's running correctly:

```bash
journalctl -u zenrockd -f -o cat
```

#### **9. Enable Service (Optional)**

To ensure your node starts automatically after reboots, you can create a systemd service:

```bash
sudo tee /etc/systemd/system/zenrockd.service > /dev/null <<EOF
[Unit]
Description=Zenrock Node
After=network.target

[Service]
User=$USER
ExecStart=$(which zenrockd) start
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

Enable and start the service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable zenrockd
sudo systemctl start zenrockd
```

#### **10. Check Node Sync Status**

You can check the sync status of your node by running:

```bash
zenrockd status 2>&1 | jq .SyncInfo
```

Your node should start syncing with the network.

#### **11. Create a Validator (Optional)**

If you plan to become a validator, once your node is synced, you can create a validator using the following command:

```bash
zenrockd tx staking create-validator \
  --amount 1000000urock \
  --from YOUR_WALLET_NAME \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.20" \
  --commission-rate "0.10" \
  --min-self-delegation "1" \
  --pubkey $(zenrockd tendermint show-validator) \
  --moniker YOUR_MONIKER \
  --chain-id zenrock-testnet \
  --fees 5000urock
```
