install dependence  

cd $HOME && source <(curl -s https://raw.githubusercontent.com/vnbnode/binaries/main/update-binary.sh)

1. Clone GitHub Repositories

git clone https://github.com/airchains-network/evm-station.git
git clone https://github.com/airchains-network/tracks.git

2. Setting Up and Running the EVM Station

cd evm-station
go mod tidy

/bin/bash ./scripts/local-setup.sh
/bin/bash ./scripts/local-start.sh 

3. Setting Up DA Keys and Running DA Client  (eigen)

wget https://github.com/airchains-network/tracks/releases/download/v0.0.2/eigenlayer
chmod +x ./eigenlayer

./eigenlayer operator keys create --key-type ecdsa [your-keyname-wallet]

give it password

4. Setting Up and Running Tracks
sudo rm -rf ~/.tracks
cd tracks
go mod tidy

go run cmd/main.go init --daRpc "disperser-holesky.eigenda.xyz" --daKey "daKey" --daType "eigen" --moniker "your-monikey" --stationRpc "http://127.0.0.1:8545" --stationAPI "http://127.0.0.1:8545" --stationType "evm"

go run cmd/main.go keys junction --accountName your-walletname --accountPath $HOME/.tracks/junction-accounts/keys

faucet from airchains discord  then  

go run cmd/main.go prover v1EVM

nano ~/.tracks/config/sequencer.toml

get node id : 

go run cmd/main.go create-station --accountName your-walletname --accountPath $HOME/.tracks/junction-accounts/keys --jsonRPC " https://junction-testnet-rpc.synergynodes.com/"  --info "EVM Track" --tracks your-wallet-address --bootstrapNode "/ip4/your-ip/tcp/2300/nodeid"

sudo tee /etc/systemd/system/stationd.service > /dev/null << EOF
[Unit]
Description=station track service
After=network-online.target
[Service]
User=root
WorkingDirectory=/home/root/tracks/
ExecStart=/usr/local/go/bin/go run cmd/main.go start
Restart=always
RestartSec=3
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF

sudo systemctl enable stationd
sudo systemctl restart stationd
sudo journalctl -u stationd -f --no-hostname -o cat

creat tx for your-evm

install dependence :

curl -sL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs
sudo apt install nodejs npm 
npm install -g npm@10.8.1
npm install web3@1.5.3
nano send.js

const readline = require('readline');
const Web3 = require('web3');

const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout
});

rl.question('metamask private key: ', (privateKey) => {
  rl.question('adrees send to: ', (toAddress) => {
    rl.question('amount: ', (amount) => {
      rl.question('time delays tx : ', (interval) => {

        const rpcURL = "http://localhost:8545"; // veya sunucunuzun IP'si: "http://your-server-ip:8545"
        const web3 = new Web3(new Web3.providers.HttpProvider(rpcURL));

        function sendTransaction() {
          const account = web3.eth.accounts.privateKeyToAccount(privateKey);
          web3.eth.accounts.wallet.add(account);
          web3.eth.defaultAccount = account.address;

          const tx = {
            from: web3.eth.defaultAccount,
            to: toAddress,
            value: web3.utils.toWei(amount, 'ether'),
            gas: 21000,
            gasPrice: web3.utils.toWei('1', 'gwei')
          };

          web3.eth.sendTransaction(tx)
            .then(receipt => {
              console.log('Transaction successful with hash:', receipt.transactionHash);
            })
            .catch(err => {
              console.error('Error sending transaction:', err);
            });
        }

        setInterval(sendTransaction, interval * 1000);

        rl.close();
      });
    });
  });
});


get your private-key evm-station wallet 
/home/root/evm-station/build/station-evm keys export mykey --unarmored-hex --unsafe --keyring-backend test

node send.js

thats done . you can check your point with wallet creat above

for some error with  tracks 

nano rollback.sh

#!/bin/bash

# Define service name and log search string
service_name="stationd"
gas_string="with gas used"
vrf_error_string="Failed to Init VRF"
vrf_error2_string="Failed to Validate VRF"
vrf_error3_string="Failed to Transact Verify pod"
vrf_error4_string="VRF record is nil"
vrf_error5_string="Failed to submit pod"
restart_delay=120  # Restart delay in seconds (3 minutes)

echo "Script started and it will rollback $service_name if needed..."
while true; do
  # Get the last 10 lines of service logs
  logs=$(systemctl status "$service_name" --no-pager | tail -n 10)

  # Check for both error and gas used strings
  if [[ "$logs" =~ ($gas_string| $vrf_error_string| $vrf_error2_string| $vrf_error3_string| $vrf_error4_string| $vrf_error5_string) ]]; then
    echo "Found error and gas used in logs, stopping $service_name..."
    sudo systemctl stop "$service_name"
    cd ~/tracks

    echo "Service $service_name stopped, starting rollback..."
    go run cmd/main.go rollback
    go run cmd/main.go rollback
    go run cmd/main.go rollback
    echo "Rollback completed, starting $service_name..."
    sudo systemctl start "$service_name"
    echo "Service $service_name started"
  fi

  # Sleep for the restart delay
  sleep "$restart_delay"
done

bash ./rollback.sh


//*that all it will work automatic for u  
