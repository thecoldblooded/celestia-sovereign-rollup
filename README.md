This tutorial provides instructions for creating a sovereign gm-world rollup using Rollkit and Celestia’s data availability and consensus layer to submit Rollkit blocks. The tutorial includes setting up Ignite CLI, building a Cosmos-SDK application-specific rollup blockchain, and posting data to Celestia. Initially, developers will learn how to create a local data availability (DA) devnet on their own machine or in the cloud. It is advisable to test your rollup on a local devnet for DA before deploying it to a testnet, which eliminates the need for testnet tokens and deployment until you are ready.

Memory: 1 GB RAM CPU: Single Core AMD Disk: 25 GB SSD Storage OS: Ubuntu 22.10 x64

This process can be performed on any machine, whether it is a developer testing things on a laptop or using a virtual machine in the cloud. The tutorial has been tested on a machine with the specifications mentioned above.

Before proceeding, make sure that you have Docker installed on your computer and that you are running a local devnet with a Rollkit rollup. To begin, run the local-celestia-devnet by executing the command below.

```sh
docker run --platform linux/amd64 -p 26650:26657 -p 26659:26659 ghcr.io/celestiaorg/local-celestia-devnet:main
```
It's worth noting that the command provided above is not the same as the command given in the "Running a Local Celestia Devnet" tutorial by Celestia Labs. In this example, port 26657 on the Docker container will be linked to the local port 26650. This is done to prevent port conflicts with the Rollkit node since both the devnet and node are running on the same machine.

To verify that you can submit rollup blocks to your Celestia Devnet for DA and consensus, open a new terminal window and query the balance on the account you will be using for this purpose. This step is essential to ensure that you have sufficient funds to post blocks to the local network.

```sh
curl -X GET http://0.0.0.0:26659/balance
```

After querying your balance, you will receive a response that displays your balance in TIA x 10-6. The output will look similar to this:

{"denom":"utia","amount":"999995000000000"}

To start, stop, or remove your container, you will need to locate the Container ID that is currently running by using the following command:

```sh
docker ps
```

Then stop the container:

```sh
docker stop CONTAINER_ID_or_NAME
```

To obtain the ID or name of a stopped container, you can use the "docker ps -a" command, which will provide a list of all containers (both running and stopped) along with their respective details. Here's an example of how to use this command:

```sh
docker ps -a
```

This will give you an output similar to this:

```sh
CONTAINER ID   IMAGE                                            COMMAND            CREATED         STATUS         PORTS                                                                                                                         NAMES
t7ve54eg23w1   ghcr.io/celestiaorg/local-celestia-devnet:main   "/entrypoint.sh"   13 minutes ago   Up 6 minutes   1317/tcp, 9090/tcp, 0.0.0.0:26657->26657/tcp, :::26657->26657/tcp, 26656/tcp, 0.0.0.0:26659->26659/tcp, :::26659->26659/tcp   musing_matsumoto
```

In the given scenario, you have the option to restart the container using either its container ID (t7ve54eg23w1) or its name (musing_matsumoto). To initiate the restart process, execute the following command:

```sh
docker start t7ve54eg23w1
```

In case you wish to remove the container at any point, you can use the "docker rm" command and provide the container ID or name as an argument. Here's an example of how to do this:

```sh
docker rm CONTAINER_ID_or_NAME
```

With your Celestia devnet up and running, the next step is to install Golang. Golang will be used to build and execute the Cosmos-SDK blockchain. The Ignite CLI includes scaffolding commands that simplify blockchain development by generating all the necessary components to start a new Cosmos SDK blockchain.

To install Golang (these commands are for amd64/linux):

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

Use the following command to install Ignite CLI:

```sh
curl https://get.ignite.com/cli! | bash
```

If you encounter any issues during the installation process, refer to the full installation guide on docs.ignite.com. The above command was tested on amd64/linux.

To verify that the installation was successful, check your version by executing the following command:

```sh
ignite version
```

Open a new tab or window in your terminal and execute this command to generate the necessary components for your rollup blockchain: Scaffold the chain:

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

To initiate your rollup, download the init.sh script that is required to start the chain.

```sh
# From inside the `gm` directory
wget https://raw.githubusercontent.com/rollkit/docs/main/docs/scripts/gm/init-local.sh
```

Execute the init-local.sh script.

```sh
bash init-local.sh
```

By doing this, your rollup will be started and connected to the local Celestia devnet that you have set up. Now, let's take a look at some of the features. To view the list of keys, use the following command:

```sh
gmd keys list --keyring-backend test
```

Now that we have started our rollup and explored a bit, let's test sending a transaction from one of our keys to the other. To do so, we can use the following command:

```sh
gmd tx bank send [from_key_or_address] [to_address] [amount] [flags]
```

Assign the keys to variables to simplify adding the address.

```sh
export KEY1=gm1sbxxxxxxxxxxxxxxxx
export KEY2=gm13jhxxxxxxxxxxxxxxx
```

Based on the output from the keys command, we can create the transaction command as follows to transfer 42069 stake from one address to another:

```sh
gmd tx bank send $KEY1 $KEY2 42069 stake --keyring-backend test
```

You'll be prompted to accept the transaction.

Balances Then, query your balance:

```sh
gmd query bank balances $KEY2
```

The key that received the balance should now have a higher balance than the initial STAKING_AMOUNT value, as shown in the "balances" output which shows a balance of "10000000000000000000042069" for that key. On the other hand, the balance of the other key used for the transaction should have decreased.

```sh
gmd query bank balances $KEY1
```

Response:

```sh
balances:

amount: "9999999999999999999957931" denom: stake pagination: next_key: null total: "0"
```
