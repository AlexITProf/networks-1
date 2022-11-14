# v4 to v5

Persistence v5 gov proposal: [102](https://testnet.mintscan.io/persistence-testnet/proposals/102) \
Height: [8878000](https://testnet.mintscan.io/persistence-testnet/blocks/8878000) (Countdown) \
Release: [v5](https://github.com/persistenceOne/persistenceCore/releases/tag/v5.0.0)

## Install and setup Cosmovisor
We highly recommend validators use cosmovisor to run their nodes. This will make low-downtime
upgrades smoother, as validators don't have to manually upgrade binaries during the upgrade,
and instead can pre-install new binaries, and cosmovisor will automatically update them based
on on-chain SoftwareUpgrade proposals.

You should review the docs for cosmovisor located here: https://docs.cosmos.network/master/run-node/cosmovisor.html

If you choose to use cosmovisor, please continue with these instructions:

To install Cosmovisor (latest version):
```
go install github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor@v1.4.0
```
After this, you must make the necessary folders for cosmosvisor in your daemon home directory (`~/.persistenceCore`).
```
mkdir -p ~/.persistenceCore
mkdir -p ~/.persistenceCore/cosmovisor
mkdir -p ~/.persistenceCore/cosmovisor/genesis
mkdir -p ~/.persistenceCore/cosmovisor/genesis/bin
mkdir -p ~/.persistenceCore/cosmovisor/upgrades
```

Copy the current v4 persistenceCore binary into the cosmovisor/genesis folder and the v4 folder.
```
cp $GOPATH/bin/persistenceCore ~/.persistenceCore/cosmovisor/genesis/bin
mkdir -p ~/.persistenceCore/cosmovisor/upgrades/v4/bin
cp $GOPATH/bin/persistenceCore ~/.persistenceCore/cosmovisor/upgrades/v4/bin
```

Cosmovisor is now ready to be started. We will now set up Cosmovisor for the upgrade

Set these environment variables:
```
echo "# Setup Cosmovisor" >> ~/.profile
echo "export DAEMON_NAME=persistenceCore" >> ~/.profile
echo "export DAEMON_HOME=$HOME/.persistenceCore" >> ~/.profile
echo "export DAEMON_ALLOW_DOWNLOAD_BINARIES=false" >> ~/.profile
echo "export DAEMON_LOG_BUFFER_SIZE=512" >> ~/.profile
echo "export DAEMON_RESTART_AFTER_UPGRADE=true" >> ~/.profile
echo "export UNSAFE_SKIP_BACKUP=true" >> ~/.profile
source ~/.profile
```

Now you can start the cosmovisor for current v4.
```
cosmovisor start --minimum-gas-prices="0.0005uxprt" --home $HOME/.persistenceCore
```

Now, create the required upgrade folder, make the build, and copy the daemon over to that folder

**NOTE: Please use `go 1.19.3+` version**

```
mkdir -p ~/.persistenceCore/cosmovisor/upgrades/v5/bin
cd $HOME/persistenceCore
git pull
git checkout v5.0.0
make build
cp build/persistenceCore ~/.persistenceCore/cosmovisor/upgrades/v5/bin
~/.persistenceCore/cosmovisor/upgrades/v5/bin/persistenceCore version --long
# Check: v5.0.0
# name: persistenceCore
# server_name: persistenceCore
# version: v5.0.0
# commit: e36cab5394c56a2d6d781c9c31d143149c7449ca
# build_tags: netgo,ledger
# go: go version go1.19.3 linux/amd64
```
Now, at the upgrade height, Cosmovisor will upgrade swap the binaries.

## Manual Option
1. Wait for PersistenceCore to reach the upgrade height
2. Look for a panic message, followed by endless peer logs. Stop the daemon
3. Run the following commands:
```
cd $HOME/persistenceCore
git pull
git checkout v5.0.0
make build
cp build/persistenceCore <destination-binary>
```
4. Start the persistenceCore daemon again, watch the upgrade happen, and then continue to hit blocks

## Communications
Operators are encouraged to join the [#validators-discussion](https://discord.gg/hnvDDzRFrV)
channel of the Persistence Community Discord. This channel is the primary communication tool
for operators to ask questions, report upgrade status, report technical issues, and to build
social consensus should the need arise. If you don't have access, please reach out to someone
from the Persistence team directly.