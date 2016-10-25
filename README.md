# V1.0 Skeletal Environment
The V1.0 work is on its way (please see https://jira.hyperledger.org/browse/FAB-37 for details).

Users wishing to use pre V1.0 fabric should use branch `v0.6` under https://gerrit.hyperledger.org/r/#/admin/projects/fabric,branches

The approach being taken is to have a minimal skeletal implementation 

* to be able to send end-to-end requests through the system
* allow incremental hooking up of v1.0 components and features

**NOTE - None of the multi-peer network environment support in 0.6 is ready in the skeletal environment. The ONLY flow available currently in the skeletal environment is the one described here. This is expected to be a temporary state as features get added.**

The skeletal environment modifies the original CLI interface to send `deploy` and `invoke` requests to the `peer`.

## Steps for running a deploy and invoke

### Terminal 1 - Start the SOLO Orderer
* vagrant ssh
* cd $GOPATH/src/github/hyperledger/fabric/orderer
* go build
* ./orderer

### Terminal 2 - Start the peer
* vagrant ssh
* rm /var/hyperledger/*
* cd $GOPATH/src/github/hyperledger/fabric
* make peer
* cd peer
* CORE_PEER_COMMITTER_LEDGER_ORDERER=0.0.0.0:5151 peer node start

``It is recommended the rm /var/hyperledger/* step be done every time the peer is started in this skeleton environment.``

``The CORE_PEER_COMMITTER_LEDGER_ORDERER env var is needed to override the `orderer` property in fabric/peer/core.yaml. It can be removed once core.yaml is updated with the right default.``

At this point you should see activity on the Orderer window that the peer is in communication with the orderer. **If not, STOP.**

### Terminal 3 - Send a deploy request
* vagrant ssh
* cd $GOPATH/src/github/hyperledger/fabric/peer
* CORE_PEER_COMMITTER_LEDGER_ORDERER=0.0.0.0:5151 peer chaincode deploy -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02 -c '{"Args":["init","a","100","b","200"]}'

The above should succeed and you should see activity on both `peer` and `orderer` windows. 

**NOTE - you will also see an ERROR in the `peer` that the chaincode went down. This is OK as the endorser brings down the chaincode after deploy, pending a successful commit**

Copy the chaincode ID from the `peer` logs. For example 

`d8f2ef95a72aa85b0f92580580176479fcd0874f6b0855ae21b98dcb926353d357e94cc80b5fb9b789d3a0acfdf143166b3bdc3e483feabeb4ab0b0a530b83a6` 

is the ID from the peer log snippet

``18:22:52.813 [chaincode] deregisterHandler -> DEBU 0d7 Deregistered handler with key: d8f2ef95a72aa85b0f92580580176479fcd0874f6b0855ae21b98dcb926353d357e94cc80b5fb9b789d3a0acfdf143166b3bdc3e483feabeb4ab0b0a530b83a6``

**NOTE - once user provided chaincode name is implemented (not too distant in future), peer generated hash based chaincode IDs will not be needed.**


### Same as previous Terminal 3 - Send a invoke request

CORE_LOGGING_LEVEL=debug CORE_PEER_COMMITTER_LEDGER_ORDERER=0.0.0.0:5151 peer chaincode invoke -n _chaincode-id_  -c '{"Args":["invoke","a","b","10"]}'

where _chaincode-id_ is the ID collected from the peer log on the deploy command. 

The above should succeed and you should see activity on both `peer` and `orderer` windows. The `invoke` command would bring up the chaincode as shown by `docker ps` command.

You should also see (thanks to the debug loglevel) `ProposalResult` from the response to the request.

## Conclusion
The CLI has been instrumented to

* convert the user request to a `Proposal` and send it to the `peer`
* receive a `ProposalResult` from the `peer`, convert it to a `Transaction` and send it to the `orderer`

The `orderer` will then send the transaction to the `peer` for committing to the ledger.

The fact that `docker ps` after the `invoke request` shows a container running implies that the commit of the deploy took place successfully.

**NOTE "Query" request has been disabled. However, "invoke" have been enhanced to collect the return value from the chaincode.**

```
Excercise

The `Invoke` method of the chaincode can be modified to return state values (chaincode_example02 returns nil). This can be a stronger indication of the commit at work - multiple invokes should show previously committed ledger values returned in ProposalResult.
```


