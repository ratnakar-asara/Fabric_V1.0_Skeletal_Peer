# Getting Started with v1.0 Hyperledger Fabric
This document demonstrates an example using the Hyperledger Fabric V1 architecture.
The scenario will include the creation and joining of channels, client side authentication,
and the deployment and invocation of chaincode.  CLI will be used for the creation and
joining of the channel and the node SDK will be used for the client authentication,
and chaincode functions utilizing the channel.

Docker-compose will be used to create a consortium of three organizations, each
running a single peer, as well as a "solo" orderer and a Certificate Authority (CA).
Additionally, each peer (i.e. org) will be provisioned their own instance of couchDB, offering
a mechanism to visualize the ledger and perform complex/rich queries.
The cryptographic material, based on standard PKI implementation, has been pre-generated
and is baked into the source code in order to expedite the flow.  The CA, responsible for
issuing, revoking and maintaining said crypto material represents one of the organizations and
is needed by the client (node SDK) for authentication.  (In an enterprise scenario, each
organization might have their own CA, with more complex security measures implemented - e.g.
cross-signing certificates, etc.)

The network will be generated automatically upon execution of `docker-compose up`,
but the APIs for create channel and join channel will be explained and demonstrated;
as such, a user can go through the steps to manually generate their own network
and channel, or quickly jump to the application development phase.

## Prerequisites and setup
[Docker](https://www.docker.com/products/overview) - v1.12 or higher
[Docker Compose](https://docs.docker.com/compose/overview/) - v1.8 or higher
[Node.js](https://nodejs.org/en/download/) - comes with the node package manager (npm).
If you already have npm on your machine, issue the following command to retrieve the latest package:
```bash
npm install npm@latest
```
then execute the following to see your version:
```bash
npm -v
```
You're looking for a version higher than 2.1.8.

_cURL the source code_
Download the [cURL](https://curl.haxx.se/download.html) tool if not already installed.
Next, execute the following commands:
`curl -L https://raw.githubusercontent.com/hyperledger/fabric/master/examples/alpha/alpha.tar.gz -o alpha.tar.gz 2> /dev/null;  tar -xvf alpha.tar.gz`
This command pulls and extracts all of the necessary source code to set up your network - docker compose script,
channel generate/join script, crypto material for identity attestation, etc. Prior to issuing
the command, make sure you are in a working directory where you want the code to go.  Next, from the
exact same working directory, issue the below command:
`curl -OOOOOO https://raw.githubusercontent.com/hyperledger/fabric-sdk-node/master/examples/balance-transfer/{config.json,deploy.js,helper.js,invoke.js,query.js,package.json}`
This will pull the javascript code for issuing your deploy, invoke and query calls.  It also
retrieves dependencies for the node SDK modules.  At this point you have installed all of the
prerequisites and source code.

Your directory should contain:
```bash
JDoe-mbp:alpha JohnDoe$ pwd
/Users/JohnDoe/alpha
JDoe-mbp:alpha JohnDoe$ ls
alpha.tar.gz				channel_test.sh				src        config.json       deploy.js       helper.js
ccenv					docker-compose-gettingstarted.yml	    tmp         invoke.js     query.js      package.json
```
The following are also directories - ccenv, src, tmp.
If everything is present, move on to network generation and application development.

## Using Docker
You do not need to manually pull any images.  The images for - `fabric-peer`,
`fabric-orderer`, `fabric-ca`, and `cli` are specified in the .yml file and will
automatically download, extract, and run when you execute the `docker-compose` commands.

## Editing conifg.json
Use an editor or command line to open the config.json that you retrieved with your second
curl command.  You will need to make two minor edits at the top of the file.  You will see:
```bash
{
   "chainName":"fabric-client1",
   "chaincodeID":"mycc",
   "channelID":"myc1",
   "goPath":"../../test/fixtures",
   "chaincodePath":"github.com/example_cc",
   "keyValueStore":"/tmp/fabric-client-kvs",
   "waitTime":"30000",
   "caserver":{
      "ca_url":"http://localhost:7054"
   },
```
Change the "goPath" to the absolute path of the working directory where you curled
the source code.  Issue an `echo $PWD` to ascertain this value.  Next, change the
"chaincodePath" to "example_cc".  Now the file should look similar to the following:
```bash
{
   "chainName":"fabric-client1",
   "chaincodeID":"mycc",
   "channelID":"myc1",
   "goPath":"/Users/JohnDoe",
   "chaincodePath":"example_cc",
   "keyValueStore":"/tmp/fabric-client-kvs",
   "waitTime":"30000",
   "caserver":{
      "ca_url":"http://localhost:7054"
   },
   ```
This assumes that the above user curled the files to their $HOME directory, and
an `echo $PWD` revealed a path of /Users/JohnDoe.  Make sure to save these changes.

## Commands
The channel commands are:
* `create` - create and name a channel in the `orderer` and get back a genesis
block for the channel.  The genesis block is named in accordance with the channel name.
* `join` - use the genesis block from the `create` command to issue a join request to a Peer.

### Use Docker to create and join a channel
Ensure the hyperledger/fabric-ccenv image is tagged as latest:
```bash
docker-compose -f docker-compose-gettingstarted.yml build
```
Create network entities, create channel, join peers to channel:
```bash
docker-compose -f docker-compose-gettingstarted.yml up -d
```
Behind the scenes this started nine containers (3 peers, 3 couchDB, solo order, CLI and CA)
in detached mode.  A script - channel_test.sh - embedded within the docker-compose-gettingstarted.yml issued
the create channel and join channel commands within the CLI container.  In the end, you
are left with a network and a channel containing three peers - peer0, peer1, peer2.

View your containers:
```bash
# if you have no other containers running, you will see nine
docker ps
```
## Manually create and join channel
To manually exercise the createChannel and joinChannel APIs through the CLI container, you will
need to edit the Docker Compose file.  Use an editor to open docker-compose-gettingstarted.yml and
comment out the `channel_test.sh` command in your cli image.  Simply place a `#` to the left
of the command.  For example:
```bash
cli:
  container_name: cli
  <CONTENT REMOVED FOR BREVITY>
  working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
#  command: sh -c './channel_test.sh; sleep 1000'
```
Exec into the cli container:
```bash
docker exec -it cli sh
```
If successful, you should see the following in your terminal:
```bash
/opt/gopath/src/github.com/hyperledger/fabric/peer #
```
Send createChannel API to Ordering Service:
```
CORE_PEER_COMMITTER_LEDGER_ORDERER=orderer:7050 peer channel create -c myc1
```
This will return a genesis block - myc1.block - that you can issue join commands with.
Next, send a joinChannel API to peer0 and pass in the genesis block as an argument.
The channel is defined within the genesis block:
```
CORE_PEER_COMMITTER_LEDGER_ORDERER=orderer:7050 CORE_PEER_ADDRESS=peer0:7051 peer channel join -b myc1.block
```
To join the other peers to the channel, simply reissue the above command with peer1
or peer2 specified.  For example:
```
CORE_PEER_COMMITTER_LEDGER_ORDERER=orderer:7050 CORE_PEER_ADDRESS=peer1:7051 peer channel join -b myc1.block
```
Once the peers have all joined the channel, you are able to issues queries against
any peer without having to deploy chaincode to each of them.

### Use node SDK to register/enroll user and deploy/invoke/query
The individual javascript programs will exercise the SDK APIs to register and enroll the client with
the provisioned Certificate Authority.  Once the client is properly authenticated,
the programs will demonstrate basic chaincode functionalities - deploy, invoke, and query.  Make
sure you are in the working directory where you pulled the source code.

Register and enroll the user & deploy chaincode:
```bash
node deploy.js
```
Issue an invoke. Move units from "a" to "b":
```bash
node invoke.js
```
Query against key value "a":
```bash
node query.js
```
You will receive a "200 response" in your terminal if each command is successful.

### Use cli to deploy, invoke and query
Run the deploy command.  This command is deploying a chaincode named `mycc` to
`peer0` on the Channel ID `myc1`.  The constructor message is initializing `a` and
`b` with values of 100 and 200 respectively.
```
CORE_PEER_ADDRESS=peer0:7051 CORE_PEER_COMMITTER_LEDGER_ORDERER=orderer:7050 peer chaincode deploy -C myc1 -n mycc -p github.com/hyperledger/fabric/examples -c '{"Args":["init","a","100","b","200"]}'
```
Run the invoke command.  This invocation is moving 10 units from `a` to `b`.
```
CORE_PEER_ADDRESS=peer0:7051 CORE_PEER_COMMITTER_LEDGER_ORDERER=orderer:7050 peer chaincode invoke -C myc1 -n mycc -c '{"function":"invoke","Args":["move","a","b","10"]}'
```
Run the query command.  The invocation transferred 10 units from `a` to `b`, therefore
a query against `a` should return the value 90.
```
CORE_PEER_ADDRESS=peer0:7051 CORE_PEER_COMMITTER_LEDGER_ORDERER=orderer:7050 peer chaincode query -C myc1 -n mycc -c '{"function":"invoke","Args":["query","a"]}'
```
You can issue an `exit` command at any time to exit the cli container.

## Utilizing CouchDB
In your browser go to `localhost:5984/_utils` to access the CouchDB UI.

## Troubleshooting
If you have existing containers running you may receive an error indicating that a port is
already occupied.  If this occurs, you will need to kill the container that is using said port.

Remove a specific docker container:
```bash
docker rm <containerID>
```
Force removal:
```bash
docker rm -f <containerID>
```
Remove all docker containers:
```bash
docker rm -f $(docker ps -aq)
```
This will merely kill docker containers (i.e. stop the process).  You will not lose any images.

Remove an image:
```bash
docker rmi <imageID>
```
Forcibly remove:
```bash
docker rmi -f <imageID>
```
Remove all images:
```bash
docker rmi -f $(docker images -q)
```

## Using Vagrant
If you want to clear out previous environments (LevelDB and pre-existing blocks):
```
rm -rf /var/hyperledger/*
```

Then, from the /fabric directory, build the executables with `make orderer` and `make peer`
commands. Switch to the /build/bin directory.

### Create a channel

_Vagrant window 1 - start orderer_

```
ORDERER_GENERAL_LOGLEVEL=debug ./orderer
```
_Vagrant window 2 - ask orderer to create a chain_

```
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/msp/sampleconfig peer channel create -c myc1
```

Upon successful creation, a genesis block `myc1.block` is saved in the
/build/bin directory.

### Join a channel

_Vagrant window 3 - start the peer in a "chainless" mode_

```
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/msp/sampleconfig peer node start --peer-defaultchain=false
```

`"--peer-defaultchain=true"` is the default. It allows users to continue to work
with the default **test_chainid** without having to join a chain.

`"--peer-defaultchain=false"` starts the peer with only the channels that were
joined by the peer. If the peer never joined a channel, it would start up without
any channels. In particular, it does not have the default **test_chainid** support.

To join channels, a peer MUST be started with the `"--peer-defaultchain=false"` option.

_Vagrant window 2 - direct the peer to join a channel_

```
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/msp/sampleconfig peer channel join -b myc1.block
```

where `myc1.block` is the block that was received from the `orderer` following the
`peer channel create`create command.

At this point the user can issue transactions.
### Use the channel to deploy and invoke chaincodes

_Vagrant window 2 - deploy a chaincode to myc1_

```
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/msp/sampleconfig peer chaincode deploy -n mycc -C myc1 -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02 -c '{"Args":["init","a","100","b","200"]}'
```

Note the use of `-C myc1` to target the chaincode deployment against the `myc1` channel.

Wait for the deploy to get committed (by default the `solo orderer` can take
up to 10 seconds to send a batch of transactions to be committed.)

_Vagrant window 2 - invoke chaincode_

```
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/msp/sampleconfig peer chaincode invoke -n mycc -C myc1 -c '{"Args":["invoke","a","b","10"]}'
```

Wait for up to 10 seconds for the invoke to be committed.

_Vagrant window 2 - query chaincode_

```
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/msp/sampleconfig peer chaincode query -n mycc -C myc1 -c '{"Args":["query","a"]}'
```

To reset, clear out the `fileSystemPath` directory (defined in core.yaml) and
remove `myc1.block`.
