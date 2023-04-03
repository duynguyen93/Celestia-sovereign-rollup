# Celestia-sovereign-rollup
GM world rollup

## ☀️ Introduction
In this tutorial, we will build a sovereign gm-world rollup using Rollkit and Celestia’s data availability and consensus layer to submit Rollkit blocks.

This tutorial will cover setting up Ignite CLI, building a Cosmos-SDK application-specific rollup blockchain, and posting data to Celestia. First, we will test on a local DA network and then we will deploy to a live testnet.

### Part one
This part of the tutorial will teach developers how to easily run a local data availability (DA) devnet on their own machine (or in the cloud). Running a local devnet for DA to test your rollup is the recommended first step before deploying to a testnet. This eliminates the need for testnet tokens and deploying to a testnet until you are ready.

NOTE
Part one of the tutorial has only been tested on an AMD machine running Ubuntu 22.10 x64.

Whether you're a developer simply testing things on your laptop or using a virtual machine in the cloud, this process can be done on any machine of your choosing. We tested it out on a machine with the following specs:

Memory: 1 GB RAM
CPU: Single Core AMD
Disk: 25 GB SSD Storage
OS: Ubuntu 22.10 x64

💻 Prerequisites
Docker installed on your machine
🏠 Running local devnet with a Rollkit rollup
First, run the local-celestia-devnet by running the following command:
```sh
docker run --platform linux/amd64 -p 26650:26657 -p 26659:26659 ghcr.io/celestiaorg/local-celestia-devnet:main
```

TIP
The above command is different than the command in the Running a Local Celestia Devnet tutorial by Celestia Labs. Port 26657 on the Docker container in this example will be mapped to the local port 26650. This is to avoid clashing ports with the Rollkit node, as we're running the devnet and node on one machine.

🔎 Query your balance
Open a new terminal instance. Check the balance on your account that you'll be using to post blocks to the local network, this will make sure you can post rollup blocks to your Celestia Devnet for DA & consensus:
```sh
curl -X GET http://0.0.0.0:26659/balance
```
You will see something like this, denoting your balance in TIA x 10-6:

{"denom":"utia","amount":"999995000000000"}


🟢 Start, stop, or remove your container
Find the Container ID that is running by using the command:
```sh
docker ps
```
Then stop the container:
```sh
docker stop CONTAINER_ID_or_NAME
```
You can obtain the container ID or name of a stopped container using the docker ps -a command, which will list all containers (running and stopped) and their details. For example:
```sh
docker ps -a
```
This will give you an output similar to this:
```sh
CONTAINER ID   IMAGE                                            COMMAND            CREATED         STATUS         PORTS                                                                                                                         NAMES
d9af68de54e4   ghcr.io/celestiaorg/local-celestia-devnet:main   "/entrypoint.sh"   5 minutes ago   Up 2 minutes   1317/tcp, 9090/tcp, 0.0.0.0:26657->26657/tcp, :::26657->26657/tcp, 26656/tcp, 0.0.0.0:26659->26659/tcp, :::26659->26659/tcp   musing_matsumoto
```

In this example, you can restart the container using either its container ID (d9af68de54e4) or name (musing_matsumoto). To restart the container, run:
```sh
docker start d9af68de54e4
```
or
```sh
docker start musing_matsumoto
```
If you ever would like to remove the container, you can use the docker rm command followed by the container ID or name.

Here is an example:
```sh
docker rm CONTAINER_ID_or_NAME
```
🏗️ Building your sovereign rollup
Now that you have a Celestia devnet running, you are ready to install Golang. We will use Golang to build and run our Cosmos-SDK blockchain.

The Ignite CLI comes with scaffolding commands to make development of blockchains quicker by creating everything that is needed to start a new Cosmos SDK blockchain.

Install Golang (these commands are for amd64/linux):
```sh
cd $HOME
ver="1.19.1"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```
Now, use the following command to install Ignite CLI:
```sh
curl https://get.ignite.com/cli! | bash
```
TIP
If you have issues with installation, the full guide can be found here or on docs.ignite.com. The above command was tested on amd64/linux.

Check your version:
```sh
ignite version
```
Open a new tab or window in your terminal and run this command to scaffold your rollup. Scaffold the chain:
```sh
cd $HOME
ignite scaffold chain gm --address-prefix gm

TIP
The --address-prefix gm flag will change the address prefix from cosmos to gm. Read more on the Cosmos docs.

The response will look similar to below:

jcs @ ~ % ignite scaffold chain gm

⭐️ Successfully created a new blockchain 'gm'.
👉 Get started with the following commands:

 % cd gm
 % ignite chain serve

Documentation: https://docs.ignite.com

This command has created a Cosmos SDK blockchain in the gm directory. The gm directory contains a fully functional blockchain. The following standard Cosmos SDK modules have been imported:

staking - for delegated Proof-of-Stake (PoS) consensus mechanism
bank - for fungible token transfers between accounts
gov - for on-chain governance
mint - for minting new units of staking token
nft - for creating, transferring, and updating NFTs
and more
Change to the gm directory:

cd gm

You can learn more about the gm directory’s file structure here. Most of our work in this tutorial will happen in the x directory.

🗞️ Install Rollkit
To swap out Tendermint for Rollkit, run the following command from inside the gm directory:

go mod edit -replace github.com/cosmos/cosmos-sdk=github.com/rollkit/cosmos-sdk@v0.46.7-rollkit-v0.7.2-no-fraud-proofs
go mod edit -replace github.com/tendermint/tendermint=github.com/celestiaorg/tendermint@v0.34.22-0.20221202214355-3605c597500d
go mod tidy
go mod download
```

▶️ Start your rollup
Download the init.sh script to start the chain:
```sh
# From inside the `gm` directory
wget https://raw.githubusercontent.com/rollkit/docs/main/docs/scripts/gm/init-local.sh
```
Run the init-local.sh script:
```sh
bash init-local.sh
```
This will start your rollup, connected to the local Celestia devnet you have running.

Now let's explore a bit.

🔑 Keys
List your keys:
```sh
gmd keys list --keyring-backend test
```
You should see an output like the following

- address: gm1sa3xvrkvwhktjppxzaayst7s7z4ar06rk37jq7
  name: gm-key-2
  pubkey: '{"@type":"/cosmos.crypto.secp256k1.PubKey","key":"AlXXb6Op8DdwCejeYkGWbF4G3pDLDO+rYiVWKPKuvYaz"}'
  type: local
- address: gm13nf52x452c527nycahthqq4y9phcmvat9nejl2
  name: gm-key
  pubkey: '{"@type":"/cosmos.crypto.secp256k1.PubKey","key":"AwigPerY+eeC2WAabA6iW1AipAQora5Dwmo1SnMnjavt"}'
  type: local


💸 Transactions
Now we can test sending a transaction from one of our keys to the other. We can do that with the following command:
```sh
gmd tx bank send [from_key_or_address] [to_address] [amount] [flags]
```
Set your keys as variables to make it easier to add the address:
```sh
export KEY1=gm1sa3xvrkvwhktjppxzaayst7s7z4ar06rk37jq7
export KEY2=gm13nf52x452c527nycahthqq4y9phcmvat9nejl2
```
So using our information from the keys command, we can construct the transaction command like so to send 42069stake from one address to another:
```sh
gmd tx bank send $KEY1 $KEY2 42069stake --keyring-backend test
```
You'll be prompted to accept the transaction:

auth_info:
  fee:
    amount: []
    gas_limit: "200000"
    granter: ""
    payer: ""
  signer_infos: []
  tip: null
body:
  extension_options: []
  memo: ""
  messages:
  - '@type': /cosmos.bank.v1beta1.MsgSend
    amount:
    - amount: "42069"
      denom: stake
    from_address: gm1sa3xvrkvwhktjppxzaayst7s7z4ar06rk37jq7
    to_address: gm13nf52x452c527nycahthqq4y9phcmvat9nejl2
  non_critical_extension_options: []
  timeout_height: "0"
signatures: []
confirm transaction before signing and broadcasting [y/N]:

Type y if you'd like to confirm and sign the transaction. Then, you'll see the confirmation:

code: 0
codespace: ""
data: ""
events: []
gas_used: "0"
gas_wanted: "0"
height: "0"
info: ""
logs: []
raw_log: '[]'
timestamp: ""
tx: null
txhash: 677CAF6C80B85ACEF6F9EC7906FB3CB021322AAC78B015FA07D5112F2F824BFF

⚖️ Balances
Then, query your balance:
```sh
gmd query bank balances $KEY2
```
This is the key that received the balance, so it should have increased past the initial STAKING_AMOUNT:

balances:
- amount: "10000000000000000000042069"
  denom: stake
pagination:
  next_key: null
  total: "0"

The other key, should have decreased in balance:
```sh
gmd query bank balances $KEY1
```
Response:

balances:
- amount: "9999999999999999999957931"
  denom: stake
pagination:
  next_key: null
  total: "0"
