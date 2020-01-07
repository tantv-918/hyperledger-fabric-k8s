kubectl apply -f kubernetes/fabric-pv.yaml

kubectl apply -f kubernetes/fabric-pvc.yaml

kubectl apply -f kubernetes/backup-org1peer0-pv.yaml

kubectl apply -f kubernetes/backup-org1peer0-pvc.yaml

kubectl apply -f kubernetes/backup-org1peer1-pv.yaml

kubectl apply -f kubernetes/backup-org1peer1-pvc.yaml

kubectl apply -f kubernetes/backup-org2peer0-pv.yaml

kubectl apply -f kubernetes/backup-org2peer0-pvc.yaml

kubectl apply -f kubernetes/backup-org2peer1-pv.yaml

kubectl apply -f kubernetes/backup-org2peer1-pvc.yaml

kubectl apply -f kubernetes/fabric-tools.yaml

kubectl exec -it fabric-tools -- mkdir /fabric/config

kubectl cp config/configtx.yaml fabric-tools:/fabric/config/

kubectl cp config/crypto-config.yaml fabric-tools:/fabric/config/

kubectl cp config/chaincode/ fabric-tools:/fabric/config/

kubectl exec -it fabric-tools -- mkdir -p /opt/gopath/src/github.com/hyperledger

kubectl cp ~/go/src/github.com/hyperledger/fabric  fabric-tools:/opt/gopath/src/github.com/hyperledger/

kubectl cp ~/go/src/github.com/golang  fabric-tools:/opt/gopath/src/github.com/

kubectl exec -it fabric-tools -- /bin/bash
cryptogen generate --config /fabric/config/crypto-config.yaml
exit

kubectl exec -it fabric-tools -- /bin/bash
cp -r crypto-config /fabric/
cp -r /opt/gopath/src/github.com/hyperledger/fabric/core/chaincode/lib/cid /opt/gopath/src/github.com/hyperledger/fabric/core/chaincode/
exit

kubectl exec -it fabric-tools -- /bin/bash
cp /fabric/config/configtx.yaml /fabric/
cd /fabric
configtxgen -profile TwoOrgsOrdererGenesis -channelID $SYS_CHANNEL -outputBlock genesis.block
configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ${CHANNEL_NAME}.tx -channelID $CHANNEL_NAME
exit

kubectl exec -it fabric-tools -- /bin/bash
cd /fabric
configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate Org1MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org1MSP
configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate Org2MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org2MSP
exit

kubectl exec -it fabric-tools -- /bin/bash
chmod a+rx /fabric/* -R
exit

kubectl apply -f kubernetes/org1-ca_deploy.yaml
kubectl apply -f kubernetes/org1-ca_svc.yaml

kubectl apply -f kubernetes/org2-ca_deploy.yaml
kubectl apply -f kubernetes/org2-ca_svc.yaml

kubectl apply -f kubernetes/example-orderer_deploy.yaml
kubectl apply -f kubernetes/example-orderer_svc.yaml

kubectl apply -f kubernetes/org1peer0_deploy.yaml
kubectl apply -f kubernetes/org1peer1_deploy.yaml
kubectl apply -f kubernetes/org1peer0_svc.yaml
kubectl apply -f kubernetes/org1peer1_svc.yaml

kubectl apply -f kubernetes/org2peer0_deploy.yaml
kubectl apply -f kubernetes/org2peer1_deploy.yaml
kubectl apply -f kubernetes/org2peer0_svc.yaml
kubectl apply -f kubernetes/org2peer1_svc.yaml

kubectl exec -it fabric-tools -- /bin/bash
cd /fabric
export ORDERER_URL="example-orderer:31010"
export CORE_PEER_ADDRESSAUTODETECT="false"
export CORE_PEER_NETWORKID="nid1"
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_MSPCONFIGPATH="/fabric/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp"
export FABRIC_CFG_PATH="/etc/hyperledger/fabric"
export CORE_PEER_ADDRESS="example-org1peer0:30110"
peer channel create -o ${ORDERER_URL} -c ${CHANNEL_NAME} -f /fabric/${CHANNEL_NAME}.tx
exit

kubectl exec -it fabric-tools -- /bin/bash
export CORE_PEER_NETWORKID="nid1"
export ORDERER_URL="example-orderer:31010"
export FABRIC_CFG_PATH="/etc/hyperledger/fabric"
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_MSPCONFIGPATH="/fabric/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp"

export CORE_PEER_ADDRESS="example-org1peer0:30110"
peer channel fetch newest -o ${ORDERER_URL} -c ${CHANNEL_NAME}
peer channel join -b ${CHANNEL_NAME}_newest.block
rm -rf /${CHANNEL_NAME}_newest.block

export CORE_PEER_ADDRESS="example-org1peer1:30110"
peer channel fetch newest -o ${ORDERER_URL} -c ${CHANNEL_NAME}
peer channel join -b ${CHANNEL_NAME}_newest.block
rm -rf /${CHANNEL_NAME}_newest.block
exit

kubectl exec -it fabric-tools -- /bin/bash
export CORE_PEER_NETWORKID="nid1"
export ORDERER_URL="example-orderer:31010"
export FABRIC_CFG_PATH="/etc/hyperledger/fabric"
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_MSPCONFIGPATH="/fabric/crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp"

export CORE_PEER_ADDRESS="example-org2peer0:30110"
peer channel fetch newest -o ${ORDERER_URL} -c ${CHANNEL_NAME}
peer channel join -b ${CHANNEL_NAME}_newest.block

rm -rf /${CHANNEL_NAME}_newest.block

export CORE_PEER_ADDRESS="example-org2peer1:30110"
peer channel fetch newest -o ${ORDERER_URL} -c ${CHANNEL_NAME}
peer channel join -b ${CHANNEL_NAME}_newest.block

rm -rf /${CHANNEL_NAME}_newest.block
exit

kubectl exec -it fabric-tools -- /bin/bash
cp -r /fabric/config/chaincode $GOPATH/src/
export CHAINCODE_NAME="mychaincode"
export CHAINCODE_VERSION="1.0"
export FABRIC_CFG_PATH="/etc/hyperledger/fabric"
export CORE_PEER_MSPCONFIGPATH="/fabric/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp"
export CORE_PEER_LOCALMSPID="Org1MSP"

export CORE_PEER_ADDRESS="example-org1peer0:30110"
peer chaincode install -n ${CHAINCODE_NAME} -v ${CHAINCODE_VERSION} -p chaincode/mychaincode/

export CORE_PEER_ADDRESS="example-org1peer1:30110"
peer chaincode install -n ${CHAINCODE_NAME} -v ${CHAINCODE_VERSION} -p chaincode/mychaincode/

exit

kubectl exec -it fabric-tools -- /bin/bash
cp -r /fabric/config/chaincode $GOPATH/src/
export CHAINCODE_NAME="mychaincode"
export CHAINCODE_VERSION="1.0"
export FABRIC_CFG_PATH="/etc/hyperledger/fabric"
export CORE_PEER_MSPCONFIGPATH="/fabric/crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp"
export CORE_PEER_LOCALMSPID="Org2MSP"

export CORE_PEER_ADDRESS="example-org2peer0:30110"
peer chaincode install -n ${CHAINCODE_NAME} -v ${CHAINCODE_VERSION} -p chaincode/mychaincode/

export CORE_PEER_ADDRESS="example-org2peer1:30110"
peer chaincode install -n ${CHAINCODE_NAME} -v ${CHAINCODE_VERSION} -p chaincode/mychaincode/

exit

kubectl exec -it fabric-tools -- /bin/bash
export CHAINCODE_NAME="mychaincode"
export CHAINCODE_VERSION="1.0"
export FABRIC_CFG_PATH="/etc/hyperledger/fabric"
export CORE_PEER_MSPCONFIGPATH="/fabric/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp"
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_ADDRESS="example-org1peer0:30110"
export ORDERER_URL="example-orderer:31010"
export FABRIC_LOGGING_LEVEL=debug

peer chaincode instantiate -o ${ORDERER_URL} -C ${CHANNEL_NAME} -n ${CHAINCODE_NAME} -v ${CHAINCODE_VERSION} -P "OR ('Org1MSP.peer','Org2MSP.peer') " -c '{"Args":["init","a","300","b","600"]}'
exit

kubectl exec -it fabric-tools -- /bin/bash
export CORE_PEER_LOCALMSPID="Org1MSP"
export FABRIC_CFG_PATH="/etc/hyperledger/fabric"
export CORE_PEER_MSPCONFIGPATH="/fabric/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp"
export CORE_PEER_ADDRESS="example-org1peer0:30110"
export ORDERER_URL="example-orderer:31010"
export FABRIC_LOGGING_LEVEL=debug
peer channel update -f /fabric/Org1MSPanchors.tx -c $CHANNEL_NAME  -o $ORDERER_URL
exit

kubectl exec -it fabric-tools -- /bin/bash
export CORE_PEER_LOCALMSPID="Org2MSP"
export FABRIC_CFG_PATH="/etc/hyperledger/fabric"
export CORE_PEER_MSPCONFIGPATH="/fabric/crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp"
export CORE_PEER_ADDRESS="example-org2peer0:30110"
export ORDERER_URL="example-orderer:31010"
export FABRIC_LOGGING_LEVEL=debug
peer channel update -f /fabric/Org2MSPanchors.tx -c $CHANNEL_NAME  -o $ORDERER_URL
exit

kubectl exec -it fabric-tools -- /bin/bash
export FABRIC_CFG_PATH="/etc/hyperledger/fabric"
export ORDERER_URL="example-orderer:31010"
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_MSPCONFIGPATH="/fabric/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp"
export CORE_PEER_ADDRESS="example-org1peer0:30110"

peer chaincode invoke --peerAddresses example-org1peer0:30110 -o example-orderer:31010 -C mychannel -n mychaincode -c '{"Args":["CreateStudent","20156425","Trinh Van Tan"]}'

peer chaincode query -C mychannel -n mychaincode -c '{"Args": ["GetAllStudents"]}'
exit
