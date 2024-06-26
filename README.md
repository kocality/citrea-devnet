# Citrea Devnet Node Guide

<img src="https://cryptoslate.com/wp-content/themes/cryptoslate-2020/imgresize/timthumb.php?src=https://cryptoslate.com/wp-content/uploads/2024/02/citrea-pr.jpg&w=700&h=367&q=75" width="700"/>

## About Citrea
Citrea is the first rollup that enhances the capabilities of Bitcoin blockspace with zero-knowledge technology, making it possible to build everything on Bitcoin.
* [Twitter](https://x.com/citrea_xyz)
* [Website](https://citrea.xyz/)
* [Discord](https://discord.gg/citrea)
* [Docs](https://docs.citrea.xyz/)
* [Github](https://github.com/chainwayxyz/citrea)

This guide explains step-by-step how to run a `Citrea Devnet` Node.

Note: This node can be run in 2 ways. Today we will run it with `Configurable Setup`. I will add the other way to the guide in the coming days. 

## System Requirements

- CPU: 4 core
- RAM: 8 GB
- Storage: 256 GB SSD (NVMe recommended)
- Network: 25+ Mbps network connection

## Step 1: System Updates and Installation of Required Tools

### Update System Packages
```bash
sudo apt update
sudo apt upgrade -y
```
### Docker Installation
```bash
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt install docker-ce -y

# Launch Docker and Set to Autostart:
sudo systemctl start docker
sudo systemctl enable docker
  ```
#### To confirm that Docker is installed successfully:
```bash
docker --version
  ```

### Docker Compose Installation
```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
  ```
#### To confirm that Docker Compose is installed successfully:
```bash
docker-compose --version
  ```
### Install Development Tools and Rust
```bash
sudo apt install build-essential -y
sudo apt install clang -y

# For Rust:
# Note: When installing Rust, we select the 1st option "Proceed with standard installation".
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
  ```

#### To confirm that Rust is installed successfully:
```bash
rustc --version
  ```

## Step 2: Setup Bitcoin Signet

### Clone the Bitcoin Signet Container
```bash
git clone https://github.com/chainwayxyz/bitcoin_signet && cd bitcoin_signet
```
### Create and Run the Signet Container
```bash
docker build -t bitcoin-signet .

docker run -d --name bitcoin-signet-client-instance \
  --env MINERENABLED=0 \
  --env SIGNETCHALLENGE=512102653734c749d5f7227d9576b3305574fd3b0efdeaa64f3d500f121bf235f0a43151ae \
  --env BITCOIN_DATA=/mnt/task/btc-data \
  --env ADDNODE=signet.citrea.xyz:38333 -p 38332:38332 \
  bitcoin-signet
```
#### To see the logs of Bitcoin Signet docker:
```bash
docker logs bitcoin-signet-client-instance
```
![signet log](https://github.com/kocality/citrea-node-guide/assets/69348404/c54bc983-1d17-47e7-be1f-541bdf28dcec)

## Step 3: Setup Citrea Client 

### Clone the Citrea Client 
```bash
cd
git clone https://github.com/chainwayxyz/citrea --branch=v0.4.1 && cd citrea
```
### Edit the Rollup Configuration File
```bash
nano configs/devnet/rollup_config.toml
```
In this section, we will change "node_url", "node_username" and "node_password".

Since we setup Bitcoin Signet on the same server as our Citrea client, we make the "node_url" part as `http://0.0.0.0:38332`. 

As in the picture, we change the "node_username" and "node_password" parts to `bitcoin`. 

![nano](https://github.com/kocality/citrea-node-guide/assets/69348404/887401d3-d961-48d3-a8be-e48bb2dcae9f)

```bash
mkdir -p ~/citrea/configs/devnet/genesis-files
```

### Build the Project
```bash
# this part takes a little longer
screen -S citrea
SKIP_GUEST_BUILD=1 make build-release
```
### Start Citrea Client
```bash
./target/release/citrea --genesis-paths configs/devnet/genesis-files --da-layer bitcoin --rollup-config-path configs/devnet/rollup_config.toml
```

Note: If you get logs as you see in the image, our node is running successfully. When xxxx in the `finalise_l2{l2_block_height=xxxx}` output reaches the current block, it means that our node has synced.

You can follow the blocks from the explorer [in this link](https://explorer.devnet.citrea.xyz/blocks). This step takes a little longer.

![synced](https://github.com/kocality/citrea-node-guide/assets/69348404/900f1300-043e-4943-b6f0-8a88a01cc641)

### Install Bitcoin Core
```bash
wget https://bitcoincore.org/bin/bitcoin-core-25.0/bitcoin-25.0-x86_64-linux-gnu.tar.gz
tar -xzf bitcoin-25.0-x86_64-linux-gnu.tar.gz
sudo install -m 0755 -o root -g root -t /usr/local/bin bitcoin-25.0/bin/*
```
### Check Block Count of Signet Container via bitcoin-cli
```bash
# In the Rollup Configuration File we set user and password to "bitcoin". so don't change it if you did it as in the manual here.
bitcoin-cli -rpcconnect=0.0.0.0 -rpcport=38332 -rpcpassword=bitcoin -rpcuser=bitcoin -signet getblockcount
```

You can check it from Citrea Bitcoin Signet Explorer at [this link](https://mempool.signet.citrea.xyz/)

![koca1](https://github.com/kocality/citrea-node-guide/assets/69348404/f3f4bd63-75c0-4c36-a29c-8e86328bc00e)

## Note: You can ask your questions about Node on the [Discord](https://discord.gg/citrea).

## My Node Running Experience
The process of running the Citrea Devnet Node was generally smooth. The guide doc was detailed and clear, covering important steps like Docker and Rust installation. However, the process might be a bit complex for someone setting up a node for the first time, and some steps could benefit from clearer explanations. Additionally, some configuration files required extra checks and adjustments to work correctly. Overall, the performance and stability were quite good, and the node synchronized without issues.
