# Blockchain_and_IoT
How to build a simple IoT device and connect it to the Ethereum blockchain

This IoT device is connected to the Ethereum blockchain and is used to open a door (in this case the door lock is represented by an LED) when the appropriate amount of funds are sent by a user on the blockchain. 

This is a simple example and requires a more rigorously-tested version in order to implement it in production but it demonstrates how an IoT device can be connected, controlled, and responded to in response to certain events on an Ethereum blockchain.

This idea came from the book *Mastering Blockchain* by Imran Bashir.

## Table of Contents
- [Setting Up Raspberry Pi](#setting-up-raspberry-pi)
  - [Command Line](#command-line)
- [First Boot](#first-boot)
  - [Update Locale](#update-locale)
  - [Disable screen from turning off](#disable-screen-from-turning-off)
- [Description of Different Ethereum Networks](#description-of-different-ethereum-networks)
  - [MainNet](#mainnet)
  - [TestNet (Ropsten)](#testnet-ropsten)
  - [PrivateNet (Ethereum Version 1.5.6)](#privatenet-ethereum-version-156)
    - [Network ID](#network-id)
    - [Genesis File](#genesis-file)
    - [Data Directory](#data-directory)
    - [Flags](#flags)
    - [Static Nodes](#static-nodes)
    - [Create privategenesis.json](#create-privategenesisjson)
    - [Start private network](#start-private-network)
    - [Error](#error)
  - [PrivateNet (Ethereum Version 1.7.0 Unstable)](#privatenet-ethereum-version-170-unstable)
    - [Create Genesis File](#create-genesis-file)
    - [Initialize Genesis File](#initialize-genesis-file)
    - [IP Address and Port](#ip-address-and-port)
    - [Enode](#enode)
    - [Start Blockchain](#start-blockchain)
- [Extra Info](#extra-info)

## Setting Up Raspberry Pi
When this project was initially taken on I assumed it was going to be a plug and play type of scenario. I was wrong. The reason is because I have an older model of the Raspberry Pi and I had to do a lot of research to find work arounds. This is mostly due to Ethereum clients being developed for the newer models Raspberry Pi 2 and Raspberry Pi 3. I have Raspberry Pi Model B. [Here is a handy guide](https://www.element14.com/community/community/raspberry-pi/blog/2016/11/21/how-to-identify-which-model-of-the-raspberry-pi-you-have) on how to identify which Raspberry Pi model you have.

After doing many hours of research I finally came across an excellent repo to install Ethereum on nearly every model of Raspberry Pi. [Link](https://github.com/EthEmbedded/Raspi-Eth-Install)

Here is the more detailed documenation of [raspbian-ua-netinst](https://github.com/debian-pi/raspbian-ua-netinst)

### Command Line
Command line options can by accessed by `./geth help` and/or the following link: [Command Line Options](https://github.com/ethereum/go-ethereum/wiki/Command-Line-Options)

## First Boot
### Update Locale
\*Must be in root

Configure your default locale: `dpkg-reconfigure locales`

Configure your timezone: `dpkg-reconfigure tzdata`

[Link](https://github.com/debian-pi/raspbian-ua-netinst)

### Disable screen from turning off
\*Make sure you are logged into your user account\*

To disable the screen from turning off run the following commands

`cd /boot`

`nano cmdline.txt`

Append to end of cmdline.txt file

`consoleblank=0`

Here is the [link](https://raspberrypi.stackexchange.com/a/61080) where I found out how to do this.

## Description of Different Ethereum Networks

### MainNet
MainNet is the current live network of ethereum. The current version of MainNet is __Metropolis__

### TestNet (Ropsten)
Used by developers or users as a test platform to test smart contrats and other blockchain-related proposals.

Connect to TestNet: `./geth --testnet`

### PrivateNet (Ethereum Version 1.5.6)
Allows creation of an independent private network that can be used as a distributed ledger between participating entities and for the development and testing of smart contracts. Create an entirely new blockchain.

Need three components to create PrivateNet
1. Network ID
2. Genesis file
3. Data directory to store blockchain data\*

\*Data directory is not strictly required. If there is more than one blockchain active on the system, then data directory should be specified so that a separate directory is used for the new blockchain.

PrivateNet differs from MainNet and TestNet in that it uses its own unique genesis block and Network ID

On MainNet, geth knows about peers by default and connects automatically.

On PrivateNet, geth needs to be configured by specifying appropriate flags and configuration in order for it to be able to be discoverable by other peers or to discover other peers.

Should disable node discovery so that other nodes on the internet can not discover your private network and is truly private.

#### Network ID
Network ID can be any positive number except 1 and 3. 1 and 3 are already in use by MainNet and TestNet respectively.

Network ID 786 was chosen as an example in the book.

#### Genesis file
The genesis file contains necessary fields required for a custom genesis block. This is the first block in the network and does not point to any previous block. The Ethereum protocal does rigorous checking to ensure that no other node on the internet can participate in the consensus mechanism, unless they have the same genesis block.

Here is a custom genesis file that was chosen as an example in the book. This file can be saved in a text file with the JSON extenstion. For example, `privategenesis156.json`
```
{
    "nonce": "0x0000000000000042",
    "timestamp": "0x0",
    "parentHash":"0x0000000000000000000000000000000000000000000000000000000000000000",
    "extraData": "0x0",
    "gasLimit": "0x4c4b40",
    "difficulty": "0x400",
    "mixhash":"0x0000000000000000000000000000000000000000000000000000000000000000",
    "coinbase": "0x0000000000000000000000000000000000000000",
    "alloc": { }
}
```
Ether can be preallocated by specifying beneficiary addresses and the amount of Wei in `alloc`, but it is usually not necessary because being on the private network, either can be mined very quickly.

#### Data Directory
This is the directory where the blockchain data for the private Ethereum network will be saved. The example chosen in the book is `~/.ethereum/privatenet`

In the geth client, many parameters are specified in order to launch, fine-tune the configuration, and launch the private network. Here are the flags:

#### Flags
* `--nodiscover`: Ensures that the node is not automatically discoverable if it happens to have the same genesis file and Network ID.
* `--maxpeers`: Used to specify the number of peers allowed to be connected to the PrivateNet. If set to 0, then no one will be able to connect. Setting to 0 can be used for private testing.
* `--rpc`: This is used to enable the RPC interface in geth.
* `--rpcapi`: Takes a list of API's to be allowed as a parameter. Examples include `eth,web3` which will enable the web3 and eth interface over RPC.
* `--rpcport`: Sets up the TCP RPC port. For example, 9999.
* `--rpccorsdomain`: Specifies the URL that is allowed to connect to th private geth node and perform RPC operations.
* `--port`: Specifies the TCP port that will be used to listen to the incoming connections from other peers.
* `--identity`: A string that specifies the name of a private node.

#### Static Nodes
If there is a need to connect to a specific set of peers, then these nodes can be added to a file where the `chaindata` and `keystore` files are saved. For example, can be saved in the `~/.ethereum/privatenet` directory. The filename should be named `static-nodes.json`. This can be valuable in a private network. Here is an example of the json file:
```
[
"enode://44352ede5b9e792e437c1c0431c1578ce3676a87e1f588434aff1299d30325c233c8d426fc57a25380481c8a36fb3be2787375e932fb4885885f6452f6efa77f@xxx.xxx.xxx.xxx:TCP_PORT"
]
```
`xxx` is the public IP address and `TCP_Port` can be any valid and available TCP port on the system. The long hex string is the node ID.

#### Create privategenesis156.json
`cd /go-ethereum/build/bin`

` mkdir privether`

`nano privategenesis156.json`
 
#### Start Private Network
`./geth --datadir ~/.ethereum/privatenet init ./privether/privategenesis156.json`

#### Error
Received an error when trying to run `./geth init ./privether/privategenesis156.json`.

The error `Fatal: invalid genesis file: json: cannot unmarshal hex string of odd length into Go value of type hexutil.Bytes`

I searched for an answer and came across [this helpful answer.](https://ethereum.stackexchange.com/a/15314/16203)

Since I have Geth version 1.7.0, I needed to use an updated json file.

### PrivateNet (Ethereum Version 1.7.0 Unstable)
[Private Network Set Up](https://github.com/ethereum/go-ethereum/wiki/Private-network)

[JavaScript Console Commands](https://github.com/ethereum/go-ethereum/wiki/JavaScript-Console#eth)

#### Create Genesis File

`genesis170.json` file
```
{
    "config": {
        "chainId": 15,
        "homesteadBlock": 0,
        "eip155Block": 0,
        "eip158Block": 0
    },
    "difficulty": "200",
    "gasLimit": "2100000",
    "alloc": {
        "7df9a875a174b3bc565e6424a0050ebc1b2d1d82": { "balance": "300000" },
        "f41c74c9ae680c1aa78f42e5647a62f353b7bdde": { "balance": "400000" }
    }
}
```

#### Initialize Genesis File
To create a database using the above genesis block, run the following command. This will import and set canonical genesis block for your chain.

`./geth --datadir ~/.ethereum/privatenet init genesis170.json`

#### IP Address and Port
Need to change port number on any other node wanting to connect to the blockchain using:

`./geth --port xxxx`

Need to get ip address from ethernet/router/WiFi address from any node connected to the blockchain using the command. If we are not on the same internal network then the node already connected to the blockchain needs to enable port forwarding:

`ip address`

#### Enode
Need to get enode from any node connected to the blockchain and supply it to `bootnodes` command. `enode` is supplied when you run `./geth --datadir ~/.ethereum/privatenet --networkid yyyy` or when the JavaScript console is running with the command `admin.nodeInfo`

`./geth --bootnodes enode`

or

when JavaScript console is started (see directly below), can use the command `admin.addPeer(nodeURL)` to connect to the blockchain.

Pass a `nodeURL` to connect a to a peer on the network. The `nodeURL` needs to be in [enode URL format](https://github.com/ethereum/wiki/wiki/enode-url-format). geth will maintain the connection until it shuts down and attempt to reconnect if the connection drops intermittently.

You can find out your own node URL by using nodeInfo or looking at the logs when the node boots up e.g.:

```
[P2P Discovery] Listening, enode://6f8a80d14311c39f35f516fa664deaaaa13e85b2f7493f37f6144d86991ec012937307647bd3b9a82abe2974e1407241d54947bbb39763a4cac9f77166ad92a0@54.169.166.226:30303
```

#### Start Blockchain
Set network ID. Future runs of geth on this data directory will use the genesis block you have defined. The last command `console` starts an interactive JavaScript environment

`./geth --datadir ~/.ethereum/privatenet --networkid yyyy console`

After starting the JavaScript environment you can check to see if the other nodes are connected to each other on the private blockchain by running

`admin.peers`

## Extra Info
To install a GUI for any device you can use [LXDE](http://lxde.org/). To install, run the following commands. Each of them will take some time. LXDE package very likely depends on the [XORG](https://www.x.org/wiki/) package.

`sudo apt-get install xorg`

`sudo apt-get install lxde`
