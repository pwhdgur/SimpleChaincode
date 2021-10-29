# SimpleChaincode Lifecycle Management
- Reference Material : the linux foundation lfd272

## Set Netowrk && Environment

### test-network 구동
- hyperledger fabric v2.2
./network.sh up createChannel -ca -s couchdb

### CouchDB URL
- http://localhost:5984/_utils 확인 가능
- COUCHDB_USER=admin
- COUCHDB_PASSWORD=adminpw

### Terminal 1 Environment variables for Org1MS 
- export PATH=${PWD}/../bin:$PATH
- export FABRIC_CFG_PATH=$PWD/../config/
- cd $HOME/go/src/github.com/pwhdgur/hyperledger/fabric-samples/test-network
- export CORE_PEER_TLS_ENABLED=true
- export CORE_PEER_LOCALMSPID="Org1MSP"
- export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
- export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
- export CORE_PEER_ADDRESS=localhost:7051

### Terminal 2 Environment variables for Org2MS 
- export PATH=${PWD}/../bin:$PATH
- export FABRIC_CFG_PATH=$PWD/../config/
- cd $HOME/go/src/github.com/pwhdgur/hyperledger/fabric-samples/test-network
- export CORE_PEER_TLS_ENABLED=true
- export CORE_PEER_LOCALMSPID="Org2MSP"
- export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
- export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
- export CORE_PEER_ADDRESS=localhost:9051

## Deploy Chaincode
- peer channel list
- (option) ./network.sh deployCC -ccn simple_chaincode -ccp../lfd272/chaincodes/simple_chaincode -ccl javascript-ccv 1.0

### Chaincode Package (terminal 1 && terminal 2)
- peer lifecycle chaincode package simple_chaincode.tar.gz --path ../lfd272/chaincodes/simple_chaincode --lang node --label simple_chaincode_1.0

### Chaincode Install (terminal 1)
- peer lifecycle chaincode install simple_chaincode.tar.gz

### Chaincode Install (terminal 2)
- peer lifecycle chaincode install simple_chaincode.tar.gz

- Chaincode code package identifier: simple_chaincode_1.0:4fae17d0c4436adfcdf4914ed509ee76060d48963eea565959d6c59597ea08d3
- export PACKAGE_ID=simple_chaincode_1.0:4fae17d0c4436adfcdf4914ed509ee76060d48963eea565959d6c59597ea08d3

### Chaincode Approveformyorg (terminal 1)
- peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name simple_chaincode --version 1.0 --package-id $PACKAGE_ID --sequence 1 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

### Chaincode Approveformyorg (terminal 2)
- peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name simple_chaincode --version 1.0 --package-id $PACKAGE_ID --sequence 1 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem


### (Optional) Chaincode checkcommitreadiness for termianl 1
- peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name simple_chaincode --version 1.0 --sequence 1 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --output json


### Chaincode Commit terminal 1 (Org 1 && Org 2)
- peer lifecycle chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name simple_chaincode --version 1.0 --sequence 1 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt

### (Optional) Chaincode querycommitted for termianl 1 
- peer lifecycle chaincode querycommitted --channelID mychannel --name simple_chaincode --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

## Chaincode Invoking
- put->get->del->get

### Invoke : put
- peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n simple_chaincode --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"put","Args":["k", "v"]}'

### Invoke : get
- peer chaincode query -C mychannel -n simple_chaincode -c '{"function":"get","Args":["k"]}'

### Invoke : del
- peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n simple_chaincode --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"del","Args":["k"]}'

### Invoke : get
- peer chaincode query -C mychannel -n simple_chaincode -c '{"function":"get","Args":["k"]}'

## Chaincode Simple_Chaincode Code View

### put
async put(ctx, key, value) {
    await ctx.stub.putState(key, Buffer.from(value));
}

### get
async get(ctx, key) {
    const value = await ctx.stub.getState(key);
    if (!value || value.length === 0) {
        throw new Error(`The asset ${key} does not exist`);
    }

    return value.toString();
}

### del
async del(ctx, key) {
    await ctx.stub.deleteState(key);
}

## Chaincode Simple_Chaincode_CompositeKey View

### put_compositeKey
async put(ctx, objType, key, value) {
    const compositeKey = this._createCompositeKey(ctx, objType, key);
    await ctx.stub.putState(compositeKey, Buffer.from(value));
}

### get_compositeKey
async get(ctx, objType, key) {
    const compositeKey = this._createCompositeKey(ctx, objType, key);
    const value = await ctx.stub.getState(compositeKey);
    if (!value || value.length === 0) {
        throw new Error(`The asset ${key} of type ${objType} does not exist`);
    }

    return value.toString();
}

### del_compositeKey
async del(ctx, objType, key) {
    const compositeKey = this._createCompositeKey(ctx, objType, key);
    await ctx.stub.deleteState(compositeKey);
}

#### _createCompositeKey
_createCompositeKey(ctx, objType, key) {
    if (!key || key === "") {
        throw new Error(`A key should be a non-empty string`);
    }

    if (objType === "") {
        return key;
    }

     return ctx.stub.createCompositeKey(objType, [key]);
}

### getByRange (from, to)
getStateByRange : list의 모든 값을 가져온다.

async getByRange(ctx, keyFrom, keyTo) {
    const iteratorPromise = ctx.stub.getStateByRange(keyFrom, keyTo);
        
    let results = [];
    for await (const res of iteratorPromise) {
        results.push({
            key:   res.key,
            value: res.value.toString()
        });
    }

    return JSON.stringify(results);
}

### getByType
async getByType(ctx, objType) {
    const iteratorPromise = ctx.stub.getStateByPartialCompositeKey(objType, []);
        
    let results = [];
    for await (const res of iteratorPromise) {
        const splitKey = ctx.stub.splitCompositeKey(res.key);
        results.push({
            objType: splitKey.objectType,
            key:     splitKey.attributes[0],
            value:   res.value.toString()
        });
    }

    return JSON.stringify(results);
}

## Docker Log
- docker logs peer0.org1.example.com

## Clear Up
- ./network.sh down