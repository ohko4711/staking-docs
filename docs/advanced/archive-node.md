---
sidebar_position: 110
---

# Run Archive Node

This is an example:

```yaml title="docker-compose.yml"
version: "3.8"

services:
  execution:
    image: ethereum/client-go:v1.14.8
    command:
      - --networkid=648
      - --state.scheme=hash
      - --verbosity=3
      - --datadir=/execution-data
      - --http
      - --http.addr=0.0.0.0
      - --http.port=8545
      - --http.vhosts=*
      - --http.corsdomain=*
      - --http.api=eth,web3,txpool,debug,net
      - --ws
      - --ws.addr=0.0.0.0
      - --ws.port=8546
      - --ws.api=eth,web3,txpool
      - --ws.origins=*
      - --nat=extip:${IP_ADDRESS}
      - --authrpc.port=8551
      - --authrpc.addr=0.0.0.0
      - --authrpc.vhosts=*
      - --authrpc.jwtsecret=/el-cl-genesis-data/jwt/jwtsecret
      - --syncmode=full
      - --gcmode=archive
      - --metrics
      - --metrics.addr=0.0.0.0
      - --metrics.port=9001
      - --port=30303
      - --discovery.port=30303
      - --bootnodes=${EL_BOOTNODES}
    volumes:
      - ./execution-data:/execution-data
      - ../../el-cl-genesis-data:/el-cl-genesis-data
    ports:
      - "8545:8545" 
      - "8546:8546" 
      - "9001:9001"
      - "30303:30303/tcp"
      - "30303:30303/udp"
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
      - --suggested-fee-recipient=0x8fBAE29f7BEbF106eB5f5C0E3f9F60d870DD6b41
      - --metrics
      - --metrics-address=0.0.0.0
      - --metrics-allow-origin=*
      - --metrics-port=5054
      - --trusted-peers=${CL_TRUSTPEERS}
      - --boot-nodes=${CL_BOOTNODES}
      - --subscribe-all-subnets
      - --import-all-attestations
      - --genesis-backfill
      - --reconstruct-historic-states
    volumes:
      - ./consensus-data:/consensus-data 
      - ../../el-cl-genesis-data:/el-cl-genesis-data
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
