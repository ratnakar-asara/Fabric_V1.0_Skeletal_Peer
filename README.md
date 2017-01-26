# Fabric V1.0 Skeletal Environment
The V1.0 work is on its way (please see https://jira.hyperledger.org/browse/FAB-37 for details).

The approach being taken is to have a skeletal implementation in with minimal component hookups to enable

* users to run deploy and invoke Proposals to the peer
* allow incremental hooking up of components

**NOTE - None of the multi-peer network environment support in 0.5 is ready in the skeletal environment. The ONLY flow available currently in the skeletal environment is the one described here. This is hopefully a temporary state.**

The skeletal environment modifies the original CLI interface to send `deploy` and `invoke` requests to the `peer`.

## Steps for running a deploy and invoke

### Start the SOLO Orderer (First terminal)
* vagrant ssh
* cd $GOPATH/src/github.com/hyperledger/fabric/orderer
* make orderer
* orderer

### Start the peer (Second terminal)
* vagrant ssh
* rm /var/hyperledger/*
* cd $GOPATH/src/github.com/hyperledger/fabric
* make peer
* CORE_PEER_MSPCONFIGPATH=./msp/sampleconfig peer node start --peer-defaultchain=true

At this point you should see activity on the `orderer` window that the peer is in communication with the orderer. **If not, STOP.**

### Send a deploy request (Third terminal)
* vagrant ssh
* cd $GOPATH/src/github.com/hyperledger/fabric
* CORE_PEER_MSPCONFIGPATH=./msp/sampleconfig peer chaincode deploy -n mycc -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02 -c '{"Args":["init","a","100","b","200"]}'

Note the name of the chaincode is `mycc`.

**NOTE 1**
The above should succeed and you should see activity in the `peer` terminal. After about 10 seconds there should be activity in the `orderer` terminal as well (by default the `solo` orderer waits for about 10 seconds to gather more transactions before delivering to the peer for committing.)

**NOTE 2**
You will also see the following error message in the `peer` window that the chaincode went down:

  ```
  Error handling chaincode support stream: stream error: code = 1 desc = "context canceled"
  ```

This is OK as the endorser brings down the chaincode after deploy pending a successful commit.


### Send a invoke request (still in the Third terminal)
CORE_PEER_MSPCONFIGPATH=./msp/sampleconfig peer chaincode invoke -n mycc  -c '{"Args":["invoke","a","b","10"]}'

The above should succeed and you should see activity in the `peer` and `orderer` terminals. The `invoke` command brings up the chaincode as shown by `docker ps` command.

### Send a query request (still in the Third terminal)
CORE_PEER_MSPCONFIGPATH=./msp/sampleconfig peer chaincode query -n mycc -c '{"Args":["query","a"]}'

##Multichannel setup
Follow the instructions [here](https://github.com/hyperledger/fabric/blob/master/docs/channel-setup.md) for multichannel setup.
