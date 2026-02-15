# Republic-Node-Guide
RepublicAI Testnet Node Setup Guide
This guide provides step-by-step instructions to deploy a full node and participate as a validator in the RepublicAI (raitestnet) environment.
1. Hardware Requirements
Component	Minimum	Recommended
CPU	4 Cores	8 Cores+
RAM	16 GB	32 GB+
Storage	500 GB SSD	1 TB NVMe SSD
OS	Ubuntu 22.04 LTS	Ubuntu 24.04 LTS
2. Server Preparation
sudo apt update -y && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
🐹 Install Go (Golang)

cd $HOME
wget [https://go.dev/dl/go1.22.4.linux-amd64.tar.gz](https://go.dev/dl/go1.22.4.linux-amd64.tar.gz)
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.22.4.linux-amd64.tar.gz
rm go1.22.4.linux-amd64.tar.gz

echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
🏗 3. Installation & Compilation

cd $HOME
git clone [https://github.com/RepublicAI/republic-chain.git](https://github.com/RepublicAI/republic-chain.git)
cd republic-chain
git fetch --tags
git checkout $(git describe --tags $(git rev-list --tags --max-count=1))
make install
⚙️ 4. Node Initialization & Port Settings Replace YOUR_NODE_NAME with your preferred validator name (e.g., Günahkar_Casper).

MONIKER="YOUR_NODE_NAME"
republicd init $MONIKER --chain-id raitestnet_77701-1
Custom Port Configuration (43)
We define the port as a variable (e.g., 43)
echo "export REPUBLIC_PORT=43" >> $HOME/.bash_profile
source $HOME/.bash_profile
📥 5. Download Genesis File

rm $HOME/.republic/config/genesis.json
wget -O $HOME/.republic/config/genesis.json [https://raw.githubusercontent.com/RepublicAI/networks/main/testnet/genesis.json](https://raw.githubusercontent.com/RepublicAI/networks/main/testnet/genesis.json)
🌐 6. Network Configuration & Peers

URL="[https://rpc.republicai.io/net_info](https://rpc.republicai.io/net_info)"
response=$(curl -s $URL)
PEERS=$(echo $response | jq -r '.result.peers[] | select(.remote_ip | test("^[0-9]{1,3}(\\.[0-9]{1,3}){3}")) | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr | split(":")[-1])"' | paste -sd "," -)
sed -i -e "s|^persistent_peers *=.*|persistent_peers = \"$PEERS\"|" $HOME/.republic/config/config.toml
⚙️ 7. System Service Setup

sudo tee /etc/systemd/system/republicd.service > /dev/null <<EOF
[Unit]
Description=RepublicAI Node
After=network-online.target

[Service]
User=$USER
ExecStart=$(which republicd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
🌐Start

sudo systemctl daemon-reload
sudo systemctl enable republicd
sudo systemctl start republicd
💰 8. Wallet & Validator Creation Note: Ensure your node is fully synced before proceeding. Create Wallet (You'll be asked for a password for the budget; make sure it's something you won't forget, so you can ask for it again later. It's invisible, so make sure you write it down.)

republicd keys add wallet_name
Create Validator (Dont forget to change your moniker)

republicd tx staking create-validator \
  --amount=1000000urai \
  --pubkey=$(republicd tendermint show-validator) \
  --moniker="YOUR_NODE_NAME" \
  --chain-id="raitestnet_77701-1" \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --from=wallet_name \
  --gas="auto" \
  --gas-adjustment="1.5" \
  -y
🚀 9. Delegation & Monitoring Check your validator status on the official explorer.

Delegate to Validator
republicd tx staking delegate YOUR_VALIDATOR_VALOPER_ADDRESS 1000000000000000000arai --from wallet --chain-id raitestnet_77701-1 --gas auto --gas-adjustment 1.5 --node tcp://localhost:${REPUBLIC_PORT}657 -y
Check Logs
journalctl -u republicd -f -o cat
🏁 Congratulations! Your RepublicAI validator is now live and fully configured. 🚀
