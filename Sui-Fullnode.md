Update and Install RUST
curl https://sh.rustup.rs -sSf | sh
source $HOME/.cargo/env
rustup update stable
install these extra dependencies
    apt-get update \
    && DEBIAN_FRONTEND=noninteractive TZ=Etc/UTC apt-get install -y --no-install-recommends \
    tzdata \
    git \
    ca-certificates \
    curl \
    build-essential \
    libssl-dev \
    pkg-config \
    libclang-dev \
    cmake
Set up your fork of the Sui repository
Go to the Sui repository on GitHub and click the Fork button in the top right-hand corner of the screen. Clone your personal fork of the Sui repository to your local machine (ensure that you insert your GitHub username into the URL):

 git clone https://github.com/thonguyen/sui.git
Set up the Sui repository as a git remote:
 cd sui
 git remote add upstream https://github.com/MystenLabs/sui
Sync your fork:
   git fetch upstream
Check out the devnet branch:
   git checkout --track upstream/devnet
Make a copy of the fullnode configuration template:
   cp crates/sui-config/data/fullnode-template.yaml fullnode.yaml
Download the latest genesis
 curl -fLJO https://github.com/MystenLabs/sui-genesis/raw/main/devnet/genesis.blob
9. Optionally, edit your fullnode.yaml file to reflect any custom paths you employ
Update the db-path field with the path to where the fullnode's database will be located. By default this will create the database in a directory ./suidb relative to your current directory.

Update the genesis-file-location with the path to the genesis file. By default, the config looks for the file genesis.blob in your current directory:

mkdir /root/sui_fullnode
mkdir /root/sui_fullnode/suidb /root/sui_fullnode/genesisdb
cp /root/sui/fullnode.yaml /root/sui/fullnode.yaml-bak
cd /root/sui
sed -i '2,2 s/suidb/\/root\/sui_fullnode\/suidb/g' fullnode.yaml
sed -i '10,10 s/genesis\.blob/\/root\/sui_fullnode\/genesisdb\/genesis\.blob/g' fullnode.yaml
mv /root/sui/genesis.blob /root/sui_fullnode/genesisdb/
After update, checking content of the file /root/sui/fullnode.yaml : $cat /root/sui/fullnode.yaml --->
Update this value to the location you want Sui to store its database
db-path: "/root/sui_fullnode/suidb"

network-address: "/dns/localhost/tcp/8080/http"
metrics-address: "127.0.0.1:9184"
json-rpc-address: "127.0.0.1:9000"
genesis:
Update this to the location of where the genesis file is stored
  genesis-file-location: "/root/sui_fullnode/genesisdb/genesis.blob"
NẾU BỎ QUA BƯỚC 9 THÌ CHẠY LỆNH NÀY
 cargo run --release --bin sui-node -- --config-path fullnode.yaml
xong chạy bưới 10 và Run

10. create systemd
sudo tee /etc/systemd/system/sui-fullnode.service > /dev/null <<EOF
[Unit]
Description=Sui Fullnode
After=network.target
[Service]
Type=simple
User=$USER
WorkingDirectory=/root/sui
ExecStart=/bin/bash -c '/root/.cargo/bin/cargo run --release --bin sui-node -- --config-path fullnode.yaml'
Restart=on-failure
RestartSec=10
Environment=RUST_BACKTRACE=1
[Install]
WantedBy=multi-user.target
EOF
Run
systemctl daemon-reload && systemctl enable sui-fullnode.service

systemctl start sui-fullnode.service
Check node
log
journalctl -u sui-fullnode.service -f -o cat 
status
 service sui-fullnode status
stop
sudo systemctl stop sui-fullnode
restar
sudo systemctl restar sui-fullnode
