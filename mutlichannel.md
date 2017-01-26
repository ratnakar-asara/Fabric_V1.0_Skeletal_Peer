# Multichannel Setup

This document describes the CLI for creating channels and directing peers to join
channels. The CLI uses `channel` APIs that are also available in the SDK.

Prior to starting, ensure that you have a current clone of the master branch of
Hyperledger Fabric and that you have followed the instructions for setting up a
[development environment](http://hyperledger-fabric.readthedocs.io/en/latest/dev-setup/devenv/).
You can clone from [github](https://github.com/hyperledger/fabric)
or [gerrit](https://gerrit.hyperledger.org/r/#/admin/projects/fabric).  If cloning from Gerrit, follow
the Fabric documentation [here](http://hyperledger-fabric.readthedocs.io/en/latest/Gerrit/gerrit/#working-with-a-local-clone-of-the-repository) to execute a proper clone.

The channel commands are:
* `create` - create a channel in the `orderer` and get back a genesis block for the channel
* `join` - use the genesis block from the `create` command to issue a join request to a Peer

```
NOTE - The main JIRA items for the work are:
   https://jira.hyperledger.org/browse/FAB-1022
   https://jira.hyperledger.org/browse/FAB-1547

The commands are a work in progress. In particular, there will be more configuration
parameters to the commands. Some relevant JIRA items are:
    https://jira.hyperledger.org/browse/FAB-1642
    https://jira.hyperledger.org/browse/FAB-1639
    https://jira.hyperledger.org/browse/FAB-1580
```

## Using docker
You do not need to manually pull any images.  The images for - `fabric-peer`,
`fabric-orderer`, and `cli` are specified in the .yml file and will automatically
download and extract when you execute the `setenv.sh` script.

### Editing docker-compose-channel.yml
You do, however, need to codify the mapping in your cli image.  Use an editor to
open `docker-compose-channel.yml`, then scroll down to the bottom of the file and
inspect the contents of the cli image.  You will see the following path defined:

  ```bash
volumes:
    - /var/run/:/host/var/run/
    #in the "- <HOST>:/opt/gopath/src/github.com/hyperledger/fabric/examples/" mapping below, the HOST part
    #should be modified to the path on the host. This will work as is in the Vagrant environment
    -/opt/gopath/src/github.com/hyperledger/fabric/examples/:/opt/gopath/src/github.com/hyperledger/fabric/examples/
    ```
The initial portion of the path needs to be modified to match the location of
the Fabric codebase on your local machine.  For example, if you simply cloned the
codebase without setting a $GOPATH, then your path might be similar to:
```bash
- /Users/JohnDoe/github.com/hyperledger/fabric/examples/
```
OR if you have cloned in your $GOPATH (the likely case for most developers), simply
execute an `echo $GOPATH` in your terminal and replace `/opt/gopath/` with your
machine's configuration.  If an `echo $GOPATH` reveals `/opt/gopath/` then you do
not need to make any modifications (this is a likely scenario for users running
Linux).  You do NOT need to modify the second half of this path.
That is the path for your cli container.  As such, the initial portion of your
path might be similar to:
```bash
- /Users/JohnDoe/work/src/github.com/hyperledger/fabric/examples/
```

### Create a channel
Copy the contents of the /docs/multichannel directory to your current working
directory, or navigate through your Fabric clone to the /docs/multichannel directory.
To avoid an onerous setup, it is recommended to simply follow the path in your clone.
For example:
```bash
cd $GOPATH/src/github.com/hyperledger/fabric/docs/multichannel
```
If you are manually copying the files, you will need to create the identical
setup as the Fabric tree.  For example, your current working directory prior to
executing a `./setenv.sh` should look similar to the following:
```bash
IBM-mbp:<YOUR_CURRENT_DIRECTORY> IBM$ pwd
/Users/IBM/<YOUR_CURRENT_DIRECTORY>
IBM-mbp:<YOUR_CURRENT_DIRECTORY> IBM$ ls
ccenv				docker-compose-channel.yml
channel-setup.md		setenv.sh
```
_Make the shell script an executable_

Ensure you are in the `/docs/multichannel` directory where the setenv.sh script
resides.  OR ensure that you have properly copied the files to another directory
on your machine and that your are in that directory.
In your terminal, issue a change mode command:
```bash
chmod +x setenv.sh
```
_Execute the shell script_
```
./setenv.sh
```
This will download and extract the three images and spin up the containers in
detached mode.  You may be prompted to enter your machine's root password at one
or more intervals during this process.

Once the images have been downloaded and extracted, execute a `docker ps` in your
terminal.  Assuming you have no other containers running, you should see three
containers - peer0, orderer, and cli.  It will look similar to the following:
```bash
CONTAINER ID        IMAGE                                COMMAND                  CREATED             STATUS              PORTS                                            NAMES
42d90c5ff6eb        rameshthoomu/fabric-peer-x86_64      "/bin/sh"                About an hour ago   Up About an hour                                                     cli
6eedbbc2298f        rameshthoomu/fabric-peer-x86_64      "peer node start --pe"   About an hour ago   Up About an hour    0.0.0.0:7051->7051/tcp, 0.0.0.0:7053->7053/tcp   peer0
6d75b816fb97        rameshthoomu/fabric-orderer-x86_64   "orderer"                About an hour ago   Up About an hour    0.0.0.0:5005->5005/tcp, 7050/tcp                 orderer
```

_Send createChannel API to Ordering Service_

First, start the CLI container.  You will issue all of the subsequent commands from within this container.
```
docker-compose -f docker-compose-channel.yml run cli
```
If the CLI container has started successfully, you should see the following in
your terminal:
```
/opt/gopath/src/github.com/hyperledger/fabric/peer #
```
In the above shell execute the create command:
```
CORE_PEER_COMMITTER_LEDGER_ORDERER=orderer:7050 peer channel create -c myc1
```
This will create a channel genesis block file `myc1.block` to issue `join` commands with.

### Join a channel
Execute the `join` command to peer0 in the CLI container.  This command is passing
the genesis block - `myc1.block` - to your peer.  You will receive a 200 response
upon a successful join.

```
CORE_PEER_COMMITTER_LEDGER_ORDERER=orderer:7050 CORE_PEER_ADDRESS=peer0:7051 peer channel join -b myc1.block
```

### Use the channel to deploy and invoke chaincodes
Run the deploy command.  This command is deploying a chaincode named `mycc` to
`peer0` on the Channel ID `myc1`.  The constructor message is initializing `a` and
`b` with values of 100 and 200 respectively.
```
CORE_PEER_ADDRESS=peer0:7051 CORE_PEER_COMMITTER_LEDGER_ORDERER=orderer:7050 peer chaincode deploy -C myc1 -n mycc -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02 -c '{"Args":["init","a","100","b","200"]}'
```

Run the invoke command.  This invocation is moving 10 units from `a` to `b`.
```
CORE_PEER_ADDRESS=peer0:7051 CORE_PEER_COMMITTER_LEDGER_ORDERER=orderer:7050 peer chaincode invoke -C myc1 -n mycc -c '{"Args":["invoke","a","b","10"]}'
```

Run the query command.  The invocation transferred 10 units from `a` to `b`, therefore
a query against `a` should return the value 90.
```
CORE_PEER_ADDRESS=peer0:7051 CORE_PEER_COMMITTER_LEDGER_ORDERER=orderer:7050 peer chaincode query -C myc1 -n mycc -c '{"Args":["query","a"]}'
```
You can issue an `exit` command at any time to exit the cli container.  

## Troubleshooting
If you receive the following error:
```
ERROR: for cli  Cannot start service cli: Mounts denied: amespaces for more info.
.
/src/github.com/hyperledger/fabric/examples
is not shared from OS X and is not known to Docker.
You can configure shared paths from Docker -> Preferences... -> File Sharing.
See https://docs.docker.com/docker-for-mac/osxfs/#n
ERROR: Encountered errors while bringing up the project.
```
then you have failed to properly configure the volumes path on your cli image.
Recall, this path needs to reflect the exact structure of the Fabric codebase on
your local machine.

If the `setenv.sh` is failing to execute or you are receiving a permissions denied
response, then you need to ensure that it has been properly reconfigured to be an
executable file.  It may be the case that you need to execute the change mode command
as root user.  From the directory where the `setenv.sh` resides:
```bash
sudo chmod +x setenv.sh
```
If you want to kill your docker containers and restart:
```bash
docker rm -f $(docker ps -aq)
```
This will merely stop the docker containers.  You will not lose any images.
If you want to remove all images from your machine, and run the script from
scratch:
```
docker rmi $(docker images -q)
```
If the images have dependent containers, then you will need to remove the containers
first.  Issue the `docker rm -f $(docker ps -aq)` command to do so.

## Using Vagrant
Build the executables with `make orderer` and `make peer` commands. Switch to build/bin directory.

### Create a channel

_Vagrant window 1 - start orderer_

```
ORDERER_GENERAL_LOGLEVEL=debug ./orderer
```
_Vagrant window 2 - ask orderer to create a chain_

```
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/msp/sampleconfig peer channel create -c myc1
```

Upon successful creation, a genesis block `myc1.block` is saved in build/bin directory.

### Join a channel

_Vagrant window 3 - start the peer in a "chainless" mode_

```
#NOTE - clear the environment with rm -rf /var/hyperledger/* after updating fabric to get channel support.

peer node start --peer-defaultchain=false
```

`"--peer-defaultchain=true"` is the default. It allows users to continue to work
with the default **TEST_CHAINID** without having to join a chain.

`"--peer-defaultchain=false"` starts the peer with only the channels that were
joined by the peer. If the peer never joined a channel, it would start up without
any channels. In particular, it does not have the default **TEST_CHAINID** support.

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
