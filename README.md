# Zetachain_guide
How to install zetachain node


### web3validator
```
web3validator provides much more than security! We are actively participating in the development of the Network and Community by providing informational, technical and humanitarian support!
```


### Testnet
[RPC_API](http://zetachain.web3validator.info:23657/)

## Install binary

```
mkdir -p $HOME/go/bin
curl -L https://zetachain-external-files.s3.amazonaws.com/binaries/athens3/v6.2.0/zetacored-ubuntu-20-amd64 > $HOME/go/bin/zetacored
chmod +x $HOME/go/bin/zetacored

```

## init 
```
zetacored config chain-id athens_7001-1
zetacored init "web34ever" --chain-id athens_7001-1
```

### Genesis
```
curl -L https://raw.githubusercontent.com/zeta-chain/network-athens3/main/network_files/config/genesis.json > $HOME/.zetacored/config/genesis.json

```
### Adderbook
```
curl -L https://snapshots1-testnet.nodejumper.io/zetachain-testnet/addrbook.json > $HOME/.zetacored/config/addrbook.json

```
### You need to create `zetacored.service`
```
sudo tee /etc/systemd/system/zetacored.service > /dev/null << EOF
[Unit]
Description=ZetaChain Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which zetacored) start
Restart=on-failure
RestartSec=10
LimitNOFILE=10000
WorkingDirectory=$HOME
[Install]
WantedBy=multi-user.target
EOF

```
# State Sync 
If you want to quickly catch up with the network, use this 
```
sudo systemctl stop zetacored

cp $HOME/.zetacored/data/priv_validator_state.json $HOME/.zetacored/priv_validator_state.json.backup
zetacored tendermint unsafe-reset-all --home $HOME/.zetacored --keep-addr-book

SNAP_RPC="http://zetachain.web3validator.info:23657/"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i 's|^enable *=.*|enable = true|' $HOME/.zetacored/config/config.toml
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $HOME/.zetacored/config/config.toml
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $HOME/.zetacored/config/config.toml
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $HOME/.zetacored/config/config.toml

mv $HOME/.zetacored/priv_validator_state.json.backup $HOME/.zetacored/data/priv_validator_state.json

sudo systemctl restart zetacored
sudo journalctl -u zetacored -f --no-hostname -o cat
```






