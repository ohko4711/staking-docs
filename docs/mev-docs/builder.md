---
sidebar_position: 120
---

# MEV Builder

This is an example:

```yaml title="docker-compose.yml"
version: "3.8"

services:
  execution:
    image: flashbots-geth-builderflashbots/builder:sha-7577ac8
    command:
      - --networkid=648
      - --state.scheme=path
      - --verbosity=3
      - --datadir=/execution-data
      - --http
      - --http.addr=0.0.0.0
      - --http.port=8545
      - --http.vhosts=*
      - --http.corsdomain=*
      - --http.api=admin,engine,net,eth,web3,debug,txpool,flashbots
      - --ws
      - --ws.addr=0.0.0.0
      - --ws.port=8546
      - --ws.api=admin,engine,net,eth,web3,debug
      - --ws.origins=*
      - --allow-insecure-unlock
      - --nat=extip:${IP_ADDRESS}
      - --authrpc.port=8551
      - --authrpc.addr=0.0.0.0
      - --authrpc.vhosts=*
      - --authrpc.jwtsecret=/el-cl-genesis-data/jwt/jwtsecret
      - --syncmode=snap
      - --rpc.allow-unprotected-txs
      - --metrics
      - --metrics.addr=0.0.0.0
      - --metrics.port=6060
      - --port=30303
      - --discovery.port=30303
      - --bootnodes=${EL_BOOTNODES}
      - --ethstats=Geth-Lighthouse-MEV-Builder:PleaseChangeThisEthstatsSecret@116.202.172.145:3000
      - --builder
      - --builder.algotype=greedy
      - --builder.beacon_endpoints=http://beacon:5052
      - --builder.genesis_fork_version=0x10000001
      - --builder.bellatrix_fork_version=0x30000001
      - --builder.genesis_validators_root=0x6e1583608783377f139b7bac661aff7d5cca656af16ac34f9d0eda27dbf1316c
      - --builder.secret_key=0xa804b34c7f4e95f284b4fa4acb10b4c803789aed629f6e4517a5346fff4f9072
      - --builder.remote_relay_endpoint=http://<relay-ip>:9062
      - --builder.secondary_remote_relay_endpoints=http://<relay-ip>:9062/
      - --builder.submission_offset=3s
      - --miner.extradata="Geth-Lighthouse-Mev-Builder"
    volumes:
      - ./execution-data:/execution-data
      - ./el-cl-genesis-data:/el-cl-genesis-data
    ports:
      - "8545:8545" 
      - "8546:8546" 
      - "8551:8551" 
      - "6060:6060" 
      - "30303:30303/tcp"
      - "30303:30303/udp"
    environment:
      BUILDER_TX_SIGNING_KEY: "0xa804b34c7f4e95f284b4fa4acb10b4c803789aed629f6e4517a5346fff4f9072"
    restart: unless-stopped

  beacon:
    image: sigp/lighthouse:v5.1.3
    command:
      - lighthouse
      - beacon_node
      - --debug-level=info
      - --datadir=/consensus-data
      - --testnet-dir=/el-cl-genesis-data/custom_config_data
      - --enr-address=${IP_ADDRESS}
      - --enr-udp-port=9000
      - --enr-tcp-port=9000
      - --enr-quic-port=9001
      - --listen-address=0.0.0.0
      - --port=9000
      - --http
      - --http-address=0.0.0.0
      - --http-port=5052
      - --http-allow-sync-stalled
      - --execution-endpoints=http://execution:8551
      - --jwt-secrets=/el-cl-genesis-data/jwt/jwtsecret
      - --metrics
      - --metrics-address=0.0.0.0
      - --metrics-allow-origin=*
      - --metrics-port=5054
      - --trusted-peers=${CL_TRUSTPEERS}
      - --boot-nodes=${CL_BOOTNODES}
      - --checkpoint-sync-url=${CL_CHECKPOINT}
      - --subscribe-all-subnets
      - --import-all-attestations
      - --builder=http://execution:8551
      - --always-prepare-payload
      - --prepare-payload-lookahead=12000
    volumes:
      - ./consensus-data:/consensus-data 
      - ./el-cl-genesis-data:/el-cl-genesis-data
    ports:
      - "9000:9000/tcp"
      - "9000:9000/udp"
      - "9001:9001/udp"
      - "5052:5052"
      - "5054:5054"
    depends_on:
      - execution
    restart: unless-stopped
```

```bash title="start.sh"
export IP_ADDRESS=$(curl -4 ifconfig.io)

if [ -z "$IP_ADDRESS" ]; then
    echo "Failed to retrieve IP address"
    exit 1
fi

echo "Using IP address: $IP_ADDRESS"

export CL_CHECKPOINT=https://checkpointz.endurancehub.org/


# Read contents from files and assign to variables
EL_BOOTNODES=$(cat bootnode_info/el_enode)
CL_TRUSTPEERS=$(cat bootnode_info/cl_peer)
CL_BOOTNODES=$(cat bootnode_info/cl_enr)
CL_STATICPEERS=$(cat bootnode_info/cl_static_address)

# Export the variables
export EL_BOOTNODES
export CL_TRUSTPEERS
export CL_BOOTNODES
export CL_STATICPEERS

docker compose -f compose.yaml up -d
```