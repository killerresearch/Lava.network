# Lava.network

=Setup and Install Node=
Minimum Specifications
6 Core
100 GB NVMe
Go

=Install Build Essentials=
Preparations
```pyhton
sudo apt update && sudo apt upgrade -y
```
​```python
MONIKER="YOUR_MONIKER”
​```

//Replace YOUR_MONIKER with your validator name/initial//

```python
sudo apt -qy install curl git jq lz4 build-essential
```
​
=Install Go=

```python
sudo rm -rf /usr/local/gocurl -Ls https://go.dev/dl/go1.20.12.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
​```
```python
eval $(echo 'export PATH=$PATH:/usr/local/go/bin' | sudo tee /etc/profile.d/golang.sh)
​```
```python
eval $(echo 'export PATH=$PATH:$HOME/go/bin' | tee -a $HOME/.profile)
```

## Setup and Install Node

- Clone Node Repository

```python
cd $HOME
```

```python
rm -rf lava
```

```python
git clone https://github.com/lavanet/lava.git
```

```python
cd lava
```

```python
git checkout v0.35.0
```

- Build Binaries

```python
export LAVA_BINARY=lavad
```

```python
make build
```

- Prepare Cosmovisor Binaries

```python
mkdir -p $HOME/.lava/cosmovisor/genesis/bin
```

```python
mv build/lavad $HOME/.lava/cosmovisor/genesis/bin/
```

```python
rm -rf build
```

- Create Symlinks

```python
sudo ln -s $HOME/.lava/cosmovisor/genesis $HOME/.lava/cosmovisor/current -f
```

```python
sudo ln -s $HOME/.lava/cosmovisor/current/bin/lavad /usr/local/bin/lavad -f
```


## Setup and Install Cosmovisor
Download and Install Cosmovisor
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@latest
​
Create Service
```python
sudo tee /etc/systemd/system/lava.service > /dev/null << EOF
[Unit]
Description=lava node service
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.lava"
Environment="DAEMON_NAME=lavad"
Environment="UNSAFE_SKIP_BACKUP=true"

[Install]
WantedBy=multi-user.target
EOF

```

​```python
sudo systemctl daemon-reload
​```

```python
sudo systemctl enable lava.service
```

## Initialize Node=
Setup Node Configurations:

lavad config chain-id lava-testnet-2
​
lavad config keyring-backend test
​
lavad config node tcp://localhost:14457

##initialize The Node

lavad init $MONIKER --chain-id lava-testnet-2


## Download Node Essentials
curl -Ls https://snapshots.kjnodes.com/lava-testnet/genesis.json > $HOME/.lava/config/genesis.json
​
curl -Ls https://snapshots.kjnodes.com/lava-testnet/addrbook.json > $HOME/.lava/config/addrbook.json

#### Configure Node

- Add Seeds

```python

sed -i -e "s|^seeds *=.*|seeds = \"3f472746f46493309650e5a033076689996c8881@lava-testnet.rpc.kjnodes.com:14459\"|" $HOME/.lava/config/config.toml

```

- Set Minimum Gas Price

```python

sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0ulava\"|" $HOME/.lava/config/app.toml

```

- Set Pruning

```python

sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.lava/config/app.toml

```

- Set Custom Ports

```python

sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:14458\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:14457\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:14460\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:14456\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":14466\"%" $HOME/.lava/config/config.toml

```

```python

sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:14417\"%; s%^address = \":8080\"%address = \":14480\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:14490\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:14491\"%; s%:8545%:14445%; s%:8546%:14446%; s%:6065%:14465%" $HOME/.lava/config/app.toml

```

- Update Chain Specifications

```python

sed -i \
  -e 's/timeout_commit = ".*"/timeout_commit = "30s"/g' \
  -e 's/timeout_propose = ".*"/timeout_propose = "1s"/g' \
  -e 's/timeout_precommit = ".*"/timeout_precommit = "1s"/g' \
  -e 's/timeout_precommit_delta = ".*"/timeout_precommit_delta = "500ms"/g' \
  -e 's/timeout_prevote = ".*"/timeout_prevote = "1s"/g' \
  -e 's/timeout_prevote_delta = ".*"/timeout_prevote_delta = "500ms"/g' \
  -e 's/timeout_propose_delta = ".*"/timeout_propose_delta = "500ms"/g' \
  -e 's/skip_timeout_commit = ".*"/skip_timeout_commit = false/g' \
  $HOME/.lava/config/config.toml

```

- Download Latest Chain Snapshot

```python

curl -L https://snapshots.kjnodes.com/lava-testnet/snapshot_latest.tar.lz4 | tar -Ilz4 -xf - -C $HOME/.lava[[ -f $HOME/.lava/data/upgrade-info.json ]] && cp $HOME/.lava/data/upgrade-info.json $HOME/.lava/cosmovisor/genesis/upgrade-info.json

```

## ## Running The Node

```python

sudo systemctl start lava.service && sudo journalctl -u lava.service -f --no-hostname -o cat

```

> *CTRL+C for Exit Node Logs*

## Create Validator /// tanpa tanda petik
Setup Wallet
Make new Wallet

"lavad keys add wallet"
​
Make sure you're already saved the phrase
Restore/import wallet

"lavad keys add wallet --restore"
​
Get some test token on faucet channel at lava discord server
Check Wallet Balance

"lavad q bank balances $(lavad keys show wallet -a)"

## Create Validator

- Backup Validator Key

```python

cat $HOME/.lava/config/priv_validator_key.json

```

<aside>
📌 *Validator key is crucial, make sure you're already back up this file*

</aside>

- Check Network Synchronization Status

```python

lavad status 2>&1 | jq .SyncInfo.catching_up

```

<aside>
📌 Make sure your node is already synchronized with latest blocks, [check here for the latest nettwork blocks](https://lava.explorers.guru/). Or just wait the “catching_up” Become false

</aside>

- Creating Validator

```python

lavad tx staking create-validator \
--amount=200000ulava \
--pubkey=$(lavad tendermint show-validator) \
--moniker "CRYPTOSIAST" \
--details "JOIN CRYPTOSIAST KALO MAU TUTOR" \
--website "HTTPS://T.ME/CRYPTOSIASTDROPS" \
--chain-id=lava-testnet-2 \
--commission-rate=0.1 \
--commission-max-rate=0.2 \
--commission-max-change-rate=0.01 \
--min-self-delegation=10000 \
--fees=20000ulava \
--from=wallet \
-y

```

<aside>
📌 *Don't forget to change moniker, details, and website configuration with your own*

</aside>

> *You must wait for some minutes untill your validator created*
> 
- For check your validator information

```python

lavad q staking validator $(lavad keys show wallet --bech val -a)

```

#
