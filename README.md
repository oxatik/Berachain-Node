# Berachain-Node
Berachain Public Testnet "Artio"

![image](https://github.com/user-attachments/assets/1d5804b9-5e87-4d21-a85e-1116edd2bfa4)


# Berachain Node
Berachain is a Layer 1 EVM-compatible chain, supporting any EVM execution client paired with a consensus client and framework called BeaconKit, Berachain nodes consist of a consensus client and an execution client. The consensus client is called beacon-kit and the execution client can be any of the ethereum clients go-ethereum, reth, nethermind,Proof-of-Liquidity is a novel consensus mechanism that aims to align network incentives, creating a strong synergy between Berachain validators and the ecosystem of projects.Berachain’s technology is built on Polaris, a high-performance blockchain framework for building EVM-compatible chains on top of the CometBFT consensus engine. You can find additional info here
 https://medium.com/berachain-foundation/the-bera-era-has-begun-49a18c6d77c0

 # Berachain: An Overview
 Berachain stands out as a major advancement in the blockchain universe, thanks to its revolutionary consensus mechanism, Proof of Liquidity (PoL). This high-performance blockchain, compatible with the Ethereum Virtual Machine (EVM), is based on Polaris and the CometBFT consensus engine, offering a robust and versatile alternative to existing infrastructures. The PoL does not just secure the network; it also aims to strengthen cooperation between validators and applications, promoting a fair distribution of rewards and stimulating liquidity. With an impressive fundraising of $42 million, supported by leading investors, Berachain positions itself as a key player in blockchain evolution, promising a more inclusive and efficient ecosystem.

# Requirements for Deploying a Berachain Node
*System: Linux Ubuntu 20.04 or newer

*CPU: Quad-core

*RAM: 16 GB

*Storage: 500 GB SSD

To host a Berachain node, opt for a VPS 2 server. I chose Contabo for its reliability and performance, ideal for meeting the technical requirements of Berachain.

# Preparing to Configure a Berachain Node Locally

Berachain is currently in the testnet phase. The Bera Node, an evolution of the Polaris project, brings several significant improvements. This guide will walk you through configuring a local node in anticipation of the Genesis file publication necessary to connect to the network. It’s important to note that the binaries needed to run a full validator node are not yet publicly accessible. This guide will help you prepare your testing environment and be ready for the official launch.

## Installation
#  Run a Berachain 🐻 ⛓️ node for the Artio testnet
we run both go-ethereum and reth nodes, and for this guide we'll use reth. You can run a node that syncs from genesis if you want, or you can use a snapshot that we created that'll help you sync much faster.

# Install dependencies
Make sure you your machine has go and rust installed because we'll be compiling beacon-kit and reth

* Install go (minimum version v1.22.4)

* Install rust (minimum version v1.79.0)

# Download snapshot
i have created a snapshot you can use. The snapshot has both the consensus layer state (for beacon-kit) and the execution layer state (for reth)




```bash
 sudo apt-get install lz4

mkdir bera
cd bera

wget 'https://public-snapshots.ghostgraph.xyz/bera-testnet-20240815.tar.lz4'

tar --use-compress-program=lz4 -xvf bera-testnet-20240815.tar.lz4

ls bera-snapshot
## you should see:
## beacond-data reth-data 
```
# Build the clients
Now lets build the consensus client (beacon-kit) and the execution client (we'll use reth)
```bash
## still in bera dir

git clone https://github.com/berachain/beacon-kit/
git clone https://github.com/paradigmxyz/reth

## build beacon-kit which will serve as the consensus client
cd beacon-kit
git checkout v0.2.0-alpha.2
make

## build reth which will serve as the execution client
cd ../reth
git checkout v1.0.1
cargo build --release  
```
# Run Reth
```bash
cd ~/bera/reth/
./target/release/reth node \
	--datadir ../bera-snapshot/reth-data \
	--chain ../bera-snapshot/beacond-data/config/eth-genesis.json \
	--instance 2 \
	--http \
	--ws  
```
If you have cast installed you can confirm the node is using the snapshot data

```bash
$ cast bn -r 'http://localhost:8544'
1517942  
```
# Run Beacon-kit
```bash
cd ~/bera/beacon-kit

## generate priv_validator_key.json and priv_validator_state.json files
## we'll use the beacon-kit init command
## in a temp location and extract just these files

./build/bin/beacond init --home tmp-bera --chain-id 80084 tmp-bartio
cp tmp-bera/config/priv_validator_key.json ../bera-snapshot/beacond-data/config/
cp tmp-bera/data/priv_validator_state.json ../bera-snapshot/beacond-data/data/

## now let's run the consensus client

export P2P_SEEDS=2f8ce8462cddc9ae865ab8ec1f05cc286f07c671@34.152.0.40:26656,3037b09eaa2eed5cd1b1d3d733ab8468bf4910ee@35.203.36.128:26656,add35d414bee9c0be3b10bcf8fbc12a059eb9a3b@35.246.180.53:26656,925221ce669017eb2fd386bc134f13c03c5471d4@34.159.151.132:26656,ae50b817fcb2f35da803aa0190a5e37f4f8bcdb5@34.64.62.166:26656,773b940b33dab98963486f0e5cbfc5ca8fc688b0@34.47.91.211:26656,977edf20575a0fc1d70fca035e5e53a02be80d9a@35.240.177.67:26656,5956d13b5285896a5c703ef6a6b28bf815f7bb22@34.124.148.177:26656,c28827cb96c14c905b127b92065a3fb4cd77d7f6@testnet-seeds.whispernode.com:25456,8a0fbd4a06050519b6bce88c03932bd0a57060bd@beacond-testnet.blacknodes.net:26656,d9903c2f9902243c88f2758fe2e81e305e737fb3@bera-testnet-seeds.nodeinfra.com:26656,9c50cc419880131ea02b6e2b92027cefe17941b9@139.59.151.125:26656,cf44af098820f50a1e96d51b7af6861bc961e706@berav2-seeds.staketab.org:5001,6b5040a0e1b29a2cca224b64829d8f3d8796a3e3@berachain-testnet-v2-2.seed.l0vd.com:21656,4f93da5553f0dfaafb620532901e082255ec3ad3@berachain-testnet-v2-1.seed.l0vd.com:61656,a62eefaa284eaede7460315d2f1d1f92988e01f1@135.125.188.10:26656

./build/bin/beacond start \
  --home ../bera-snapshot/beacond-data \
  --beacon-kit.engine.jwt-secret-path ../bera-snapshot/reth-data/jwt.hex \
  --beacon-kit.engine.rpc-dial-url http://127.0.0.1:8651 \
  --p2p.seeds $P2P_SEEDS \
  --p2p.seed_mode  
```
# To learn more, check out Berachain’s foundation page and socials below.
🌐 Website: https://www.berachain.com

🐦 Twitter: https://twitter.com/berachain

👾 Discord: https://discord.gg/berachain


# Disclaimer
This guide is being provided as is. The codebases (beacon-kit, reth) that it references are open source but are not written by this author. They may have bugs. The guide also may have bugs and may not work on your system. No guarantee, representation or warranty is being made, express or implied, as to the safety or correctness. It has not been audited and as such there can be no assurance it will work as intended, and users may experience delays, failures, errors, omissions or loss of transmitted information. Nothing in these docs should be construed as investment advice or legal advice for any particular facts or circumstances and is not meant to replace competent counsel. It is strongly advised for you to contact a reputable attorney in your jurisdiction for any questions or concerns with respect thereto. Author is not liable for any use of the foregoing, and users should proceed with caution and use at their own risk.
    
