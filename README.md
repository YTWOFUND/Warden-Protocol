# Warden-Protocol

### Warden-Protocol node Installation Instructions.

[Official documentation](https://docs.wardenprotocol.org/)

System requirements:</br>
CPU: Quad Core or larger AMD or Intel (amd64) CPU
RAM:32GB RAM
SSD:1TB NVMe Storage
100MBps bidirectional internet connection
OS: Ubuntu 20.04 or 22.04</br>

You can take a weaker server

### Network configurations: </br>
Outbound - allow all traffic </br>
Inbound - open the following ports :</br>
1317 - REST </br>
26657 - TENDERMINT_RP </br>
26656 - Cosmos </br>
Running as a Provider? Add these specific ports: </br>
22221 - provider port </br>
22231 - provider port </br>
22241 - provider port </br>

### Installing the Babylon Node

1. Preparing the server/Required packages installation</br>
```
sudo apt update
sudo apt upgrade -y
sudo apt-get install libclang-dev
sudo apt install -y unzip logrotate git jq sed wget curl coreutils systemd
```
### Go installation.
```
sudo rm -rf /usr/local/go
curl -L https://go.dev/dl/go1.21.6.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
source .bash_profile
go version
```

### Download and build binaries
```
cd $HOME && rm -rf wardenprotocol
git clone https://github.com/warden-protocol/wardenprotocol
cd  wardenprotocol
git checkout v0.3.0
make install-wardend
```

# Config and init app
```
wardend config set client chain-id buenavista-1
wardend config set client keyring-backend test
wardend config set client node tcp://localhost:26657
wardend init "your moniker" --chain-id buenavista-1
```

# Download genesis and addrbook
```
curl -L https://snapshots-testnet.nodejumper.io/wardenprotocol-testnet/genesis.json > $HOME/.warden/config/genesis.json
curl -L https://snapshots-testnet.nodejumper.io/wardenprotocol-testnet/addrbook.json > $HOME/.warden/config/addrbook.json
```

# Set seeds and peers
```
sed -i -e 's|^seeds *=.*|seeds = "ddb4d92ab6eba8363bab2f3a0d7fa7a970ae437f@sentry-1.buenavista.wardenprotocol.org:26656,c717995fd56dcf0056ed835e489788af4ffd8fe8@sentry-2.buenavista.wardenprotocol.org:26656,e1c61de5d437f35a715ac94b88ec62c482edc166@sentry-3.buenavista.wardenprotocol.org:26656"|' $HOME/.warden/config/config.toml
sed -i -e 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.01uward"|' $HOME/.warden/config/app.toml
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "17"|' \
  $HOME/.warden/config/app.toml
```

# Create service file
```
sudo tee /etc/systemd/system/wardend.service > /dev/null << EOF
[Unit]
Description=Warden Protocol node service
After=network-online.target
[Service]
User=$USER
ExecStart=$(which wardend) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable wardend.service
```

# Reset and download snapshot
```
curl "https://snapshots-testnet.nodejumper.io/wardenprotocol-testnet/wardenprotocol-testnet_latest.tar.lz4" | lz4 -dc - | tar -xf - -C "$HOME/.warden"
```

# enable and start service
```
sudo systemctl start wardend.service
sudo journalctl -u wardend.service -f --no-hostname -o cat
```

### Becoming a Validator

# Create wallet key new
```
wardend keys add wallet
```

(OPTIONAL) RECOVER EXISTING KEY
```
wardend keys add wallet --recover
```

# check sync status, once your node is fully synced, the output from above will print "false"
```
wardend status 2>&1 | jq -r '.SyncInfo.catching_up // .sync_info.catching_up'
```

### We receive tokens the [faucet](https://spaceward.buenavista.wardenprotocol.org/)

# before creating a validator, you need to fund your wallet and check balance
```
wardend q bank balances $(wardend keys show wallet -a) 
```
# Create validator
```
wardend tx staking create-validator \
--amount=1000000uward \
--pubkey=$(wardend tendermint show-validator) \
--moniker="Moniker" \
--identity=FFB0AA51A2DF5955 \
--details="I love ytwo❤️" \
--chain-id=buenavista-1 \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=wallet \
--gas-prices=0.01uward \
--gas-adjustment=1.5 \
--gas=auto \
-y 
```

### Update
```
No update

Current network:buenavista-1
Current version:v0.3.0
```

### Useful commands

Check balance
```
wardend q bank balances $(wardend keys show wallet -a) 
```

CHECK SERVICE LOGS
```
sudo journalctl -u wardend -f --no-hostname -o cat
```

RESTART SERVICE
```
sudo systemctl restart wardend
```

GET VALIDATOR INFO
```
wardend status 2>&1 | jq -r '.ValidatorInfo // .validator_info'
```

DELEGATE TOKENS TO YOURSELF
```
wardend tx staking delegate $(wardend keys show wallet --bech val -a) 1000000uward --from wallet --chain-id buenavista-1 --gas-prices 0.01uward --gas-adjustment 1.5 --gas auto -y 
```

REMOVE NODE
Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your priv_validator_key.json!
```
sudo systemctl stop wardend && sudo systemctl disable wardend && sudo rm /etc/systemd/system/wardend.service && sudo systemctl daemon-reload && rm -rf $HOME/.warden && rm -rf wardenprotocol && sudo rm -rf $(which wardend) 
```
