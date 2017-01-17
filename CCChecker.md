####Copied the below content from the JIRA [FAB-1475](https://jira.hyperledger.org/browse/FAB-1475) and Gerrit [3563](https://gerrit.hyperledger.org/r/#/c/3563/)

 With pre-consensus simulation, multiple chains and relaxation by the ledger
    to simulate versions of chaincode state concurrently, we can now allow
    chaincode framework to execute invokes concurrently. This CR enables this.
    
This CR enables concurrency basically by removing the FSM states that
    enforced serialization (so basically all the FSM changes in chaincode/hander.go
    and chaincode/shim/handler.go).
    
The CR also has a "Chaincode Checker" program which has the potential for
    much bigger things
       . the tooling test their chaincodes for consistency
       . the tooling for stressing the fabric
    
The concurrency enablement was tested with the "ccchecker".
    
"ccchecker" comes with a sample "newkeyperinvoke" chaincode that should
    NEVER fail ledger consistency checks. To test simply follow these steps


#### vagrant window 1 - start orderer

	orderer

#### vagrant window 1 - start peer

	peer node start


####vagrant window 3 - bring up chaincode for test

	//deploy the chaincode used by ccchecker out of the box
	peer chaincode deploy -n mycc -p github.com/hyperledger/fabric/examples/ccchecker/chaincodes/newkeyperinvoke -c '{"Args":[""]}'
	        
	//wait for commit say for about 10 secs and then issue a query to bring the CC up
	peer chaincode query -n mycc -c '{"Args":["get","a"]}'
	    
	//verify the chaincode is up
	docker ps
	

####vagrant window 4 - run test

	cd examples/ccchecker
	go build
	./ccchecker
	
The above reads from `ccchecker.json` and executes tests concurrently.
