# Citrea Devnet Node Rehberi

<img src="https://cryptoslate.com/wp-content/themes/cryptoslate-2020/imgresize/timthumb.php?src=https://cryptoslate.com/wp-content/uploads/2024/02/citrea-pr.jpg&w=700&h=367&q=75" width="700"/>

## Citrea Hakkında
Citrea, Bitcoin blockspace'inin özelliklerini zero-knowledge teknolojisi ile geliştiren ve Bitcoin üzerine her şeyi inşa etmeyi mümkün kılan ilk rollup'tır.
* [Twitter](https://x.com/citrea_xyz)
* [Website](https://citrea.xyz/)
* [Discord](https://discord.gg/citrea)
* [Docs](https://docs.citrea.xyz/)
* [Github](https://github.com/chainwayxyz/citrea)

Bu rehber, `Citrea Devnet` Node'unun nasıl çalıştırılacağını adım adım açıklamaktadır.

Not: Bu node 2 şekilde çalıştırılabilir. Bugün `Configurable Setup` ile çalıştıracağız. Diğer yolu da önümüzdeki günlerde rehbere ekleyeceğim.

## Sistem Gereksinimleri

- CPU: 4 core
- RAM: 8 GB
- Depolama: 256 GB SSD (NVMe önerilir)
- Ağ: 25+ Mbps ağ bağlantısı

## Adım 1: Sistem Güncellemeleri ve Gerekli Araçların Kurulumu

### Sistem Paketlerini Güncelleme
```bash
sudo apt update
sudo apt upgrade -y
```
### Docker Kurulumu
```bash
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt install docker-ce -y

# Docker'ı başlatın ve Autostart olarak ayarlayın:
sudo systemctl start docker
sudo systemctl enable docker
  ```
#### Docker'ın başarılı bir şekilde kurulduğunu doğrulamak için:
```bash
docker --version
  ```

### Docker Compose Kurulumu
```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
  ```
#### Docker Compose'un başarılı bir şekilde kurulduğunu doğrulamak için:
```bash
docker-compose --version
  ```
### Geliştirme Araçlarını ve Rust'ı Kurma
```bash
sudo apt install build-essential -y
sudo apt install clang -y

# Rust için:
# # Not: Rust'ı kurarken, 1. seçeneği seçin.
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
  ```

#### Rust'ın başarılı bir şekilde kurulduğunu doğrulamak için:
```bash
rustc --version
  ```

## Adım 2: Bitcoin Signet Kurulumu

### Bitcoin Signet Konteynerini Klonlama
```bash
git clone https://github.com/chainwayxyz/bitcoin_signet && cd bitcoin_signet
```
### Signet Konteynerini Oluşturma ve Çalıştırma
```bash
docker build -t bitcoin-signet .

docker run -d --name bitcoin-signet-client-instance \
  --env MINERENABLED=0 \
  --env SIGNETCHALLENGE=512102653734c749d5f7227d9576b3305574fd3b0efdeaa64f3d500f121bf235f0a43151ae \
  --env BITCOIN_DATA=/mnt/task/btc-data \
  --env ADDNODE=signet.citrea.xyz:38333 -p 38332:38332 \
  bitcoin-signet
```
#### Bitcoin Signet docker'ın loglarını görmek için:
```bash
docker logs bitcoin-signet-client-instance
```
![signet log](https://github.com/kocality/citrea-node-guide/assets/69348404/c54bc983-1d17-47e7-be1f-541bdf28dcec)

## Adım 3: Citrea Client Kurulumu 

### Citrea Client'ı Klonlama
```bash
cd
git clone https://github.com/chainwayxyz/citrea --branch=v0.4.1 && cd citrea
```
### Rollup Configuration Dosyasını Düzenleme
```bash
nano configs/devnet/rollup_config.toml
```
Bu kısımda "node_url", "node_username" ve "node_password "ü değiştireceğiz.

Bitcoin Signet'i, Citrea client'ımızla aynı sunucuya kurduğumuz için "node_url" kısmını `http://0.0.0.0:38332` olarak ayarlıyoruz. 

Resimdeki gibi "node_username" ve "node_password" kısımlarını `bitcoin` olarak değiştiriyoruz. 

![nano](https://github.com/kocality/citrea-node-guide/assets/69348404/887401d3-d961-48d3-a8be-e48bb2dcae9f)

```bash
mkdir -p ~/citrea/configs/devnet/genesis-files
```

### Projeyi Buildleme
```bash
# this part takes a little longer
screen -S citrea
SKIP_GUEST_BUILD=1 make build-release
```

### Citrea Client'ı Başlatma
```bash
./target/release/citrea --genesis-paths configs/devnet/genesis-files --da-layer bitcoin --rollup-config-path configs/devnet/rollup_config.toml
```

Not: Eğer görselde gördüğünüz gibi loglar alıyorsanız node'umuz başarılı bir şekilde çalışıyor demektir. `Finalise_l2{l2_block_height=xxxx}` çıktısındaki xxxx, mevcut block'a ulaştığında node'umuz sync olmuş demektir.

Blockları [bu bağlantıdaki](https://explorer.devnet.citrea.xyz/blocks) Explorer'dan takip edebilirsiniz. Bu adım biraz daha uzun sürer.

![synced](https://github.com/kocality/citrea-node-guide/assets/69348404/900f1300-043e-4943-b6f0-8a88a01cc641)

### Bitcoin Core Kurulumu
```bash
wget https://bitcoincore.org/bin/bitcoin-core-25.0/bitcoin-25.0-x86_64-linux-gnu.tar.gz
tar -xzf bitcoin-25.0-x86_64-linux-gnu.tar.gz
sudo install -m 0755 -o root -g root -t /usr/local/bin bitcoin-25.0/bin/*
```
### Signet Konteynerinin Block Sayısını bitcoin-cli ile Kontrol Etme
```bash
# Rollup Configuration Dosyası'nda user ve password'ü "bitcoin" olarak ayarladık. Eğer buradaki rehberde olduğu gibi yaptıysanız o bölümleri değiştirmeyin.
bitcoin-cli -rpcconnect=0.0.0.0 -rpcport=38332 -rpcpassword=bitcoin -rpcuser=bitcoin -signet getblockcount
```

[Bu bağlantıdaki](https://mempool.signet.citrea.xyz/) Citrea Bitcoin Signet Explorer'dan kontrol edebilirsiniz.

![koca1](https://github.com/kocality/citrea-node-guide/assets/69348404/f3f4bd63-75c0-4c36-a29c-8e86328bc00e)

## Not: Node ile ilgili sorularınızı [Discord](https://discord.gg/citrea)'da sorabilirsiniz.

## Node Çalıştırma Deneyimim
Citrea Devnet Node'u çalıştırma süreci genel olarak sorunsuzdu. Rehber doküman ayrıntılı ve anlaşılırdı, Docker ve Rust kurulumu gibi önemli adımları kapsıyordu. Bununla birlikte, ilk kez bir node kuran biri için süreç biraz karmaşık olabilir ve bazı adımlarda daha net açıklamalardan yararlanılabilir. Ayrıca, bazı configuration dosyalarının doğru çalışması için ekstra kontroller ve ayarlamalar gerekiyordu. Genel olarak, performans ve stabilite oldukça iyiydi ve node sorunsuz bir şekilde senkronize oldu.

