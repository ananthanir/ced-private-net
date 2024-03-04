# Running an Ethereum Node (Tekku + Besu)
First create a random secret
```
openssl rand -hex 32 | tr -d "\n" > jwtsecret.hex
```
Create the besu configuration (config.toml)
```
network="GOERLI"            
data-path="/path to data directory to store networkdata"
rpc-http-enabled=true     
rpc-http-host="0.0.0.0"     
rpc-http-cors-origins=["*"]
rpc-ws-enabled=true       
rpc-ws-host="0.0.0.0"       
host-allowlist=["*"]        
engine-host-allowlist=["*"]
engine-rpc-enabled=true  
engine-jwt-secret= "/path to jwtsecret.hex"
```
Run Besu
```
besu --config-file "config.toml"
```
Configure Teku (config.yaml)
```
network: "goerli"  
ee-endpoint: "http://localhost:8551"
ee-jwt-secret-file: "/path to jwtsecret.hex"
metrics-enabled: true  
rest-api-enabled: true
```
Run Teku
```
teku --config-file config.yaml --ignore-weak-subjectivity-period-enabled
```

# Private Network PoW
Create a configuration file
```
{
  "config": {
    "berlinBlock": 0,
    "ethash": {
      "fixeddifficulty": 1000
    },
    "chainID": 1337
  },
  "nonce": "0x42",
  "gasLimit": "0x1000000",
  "difficulty": "0x10000",
  "alloc": {
    "fe3b557e8fb62b89f4916b721be55ceb828dbd73": {
      "privateKey": "8f2a55949038a9610f50fb23b5883af3b4ecb3c3bb792cbcefbd1542c692be63",
      "balance": "0xad78ebc5ac6200000"
    },
    "f17f52151EbEF6C7334FAD080c5704D77216b732": {
      "privateKey": "ae6ae8e5ccbfb04590405997ee2d52d2b330726137b875053c36d94e974d162f",
      "balance": "90000000000000000000000"
    }
  }
}
```
Start the first node as the bootnode
```
besu --data-path=node1 --genesis-file=./genesis.json --network-id 123 --rpc-http-enabled --rpc-http-api=ADMIN,ETH,NET,MINER,WEB3 --host-allowlist="*" --rpc-http-cors-origins="all" --rpc-http-host "0.0.0.0" --miner-enabled --miner-coinbase="0x9aC31F4E54c731de531894eE7DC24784370f5AE8"
```
Start the second node as the bootnode
```
besu --data-path=node2 --genesis-file=./genesis.json --network-id 123 --rpc-http-enabled --rpc-http-api=ADMIN,ETH,NET,MINER,WEB3 --host-allowlist="*" --rpc-http-cors-origins="all" --rpc-http-host "0.0.0.0" --miner --rpc-http-port "8546" --p2p-port "30301" --bootnodes="enode://f9c6a6cc19248cf256936f086bb57bd85f4edd0d98eb65f1b966e85a12a6985c7375e1b510a6c6ef095998ba2760dfab4f5b2093936f01563d54d9ef0f4f6bac@127.0.0.1:30303"
```

# Private Network QBFT

Create directories
```
QBFT-Network/
├── Node-1
│   ├── data
├── Node-2
│   ├── data
├── Node-3
│   ├── data
└── Node-4
    ├── data
```
Create a configuration file
```
{
  "genesis": {
    "config": {
      "chainId": 1337,
      "berlinBlock": 0,
      "qbft": {
        "blockperiodseconds": 2,
        "epochlength": 30000,
        "requesttimeoutseconds": 4
      }
    },
    "nonce": "0x0",
    "timestamp": "0x58ee40ba",
    "gasLimit": "0x47b760",
    "difficulty": "0x1",
    "mixHash": "0x63746963616c2062797a616e74696e65206661756c7420746f6c6572616e6365",
    "coinbase": "0x0000000000000000000000000000000000000000",
    "alloc": {
      "fe3b557e8fb62b89f4916b721be55ceb828dbd73": {
        "privateKey": "8f2a55949038a9610f50fb23b5883af3b4ecb3c3bb792cbcefbd1542c692be63",
        "comment": "private key and this comment are ignored.  In a real chain, the private key should NOT be stored",
        "balance": "0xad78ebc5ac6200000"
      },
      "627306090abaB3A6e1400e9345bC60c78a8BEf57": {
        "privateKey": "c87509a1c067bbde78beb793e6fa76530b6382a4c0241e5e4a9ec0a0f44dc0d3",
        "comment": "private key and this comment are ignored.  In a real chain, the private key should NOT be stored",
        "balance": "90000000000000000000000"
      },
      "f17f52151EbEF6C7334FAD080c5704D77216b732": {
        "privateKey": "ae6ae8e5ccbfb04590405997ee2d52d2b330726137b875053c36d94e974d162f",
        "comment": "private key and this comment are ignored.  In a real chain, the private key should NOT be stored",
        "balance": "90000000000000000000000"
      }
    }
  },
  "blockchain": {
    "nodes": {
      "generate": true,
      "count": 4
    }
  }
}
```
Generate node keys and a genesis file
```
besu operator generate-blockchain-config --config-file=qbftConfigFile.json --to=networkFiles --private-key-file-name=key
```
Copy the genesis & keys file to the QBFT-Network directory

Start the first node as the bootnode
```
besu --data-path=data --genesis-file=./genesis.json --rpc-http-enabled --rpc-http-api=ADMIN,ETH,NET,MINER,WEB3,QBFT --host-allowlist="*" --rpc-http-cors-origins="all"
```
Start Node-2
```
besu --data-path=data --genesis-file=./genesis.json --bootnodes=<Node-1 Enode URL> --p2p-port=30304 --rpc-http-enabled --rpc-http-api=ADMIN,ETH,NET,MINER,WEB3,QBFT --host-allowlist="*" --rpc-http-cors-origins="all" --rpc-http-port=8546
```
Start Node-3
```
besu --data-path=data --genesis-file=./genesis.json --bootnodes=<Node-1 Enode URL> --p2p-port=30305 --rpc-http-enabled --rpc-http-api=ADMIN,ETH,NET,MINER,WEB3,QBFT --host-allowlist="*" --rpc-http-cors-origins="all" --rpc-http-port=8547
```
Start Node-4
```
besu --data-path=data --genesis-file=../genesis.json --bootnodes=<Node-1 Enode URL> --p2p-port=30306 --rpc-http-enabled --rpc-http-api=ETH,NET,QBFT --host-allowlist="*" --rpc-http-cors-origins="all" --rpc-http-port=8548
```
Confirm the private network is working
```
curl -X POST --data '{"jsonrpc":"2.0","method":"qbft_getValidatorsByBlockNumber","params":["latest"], "id":1}' localhost:8545
```
