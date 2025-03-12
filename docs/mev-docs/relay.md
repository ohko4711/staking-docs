---
sidebar_position: 110
---

# MEV Relay

This is an example:

```yaml title="docker-compose.yml"
version: '3.8'

services:
  redis:
    image: redis
    restart: always
    ports:
      - '6379:6379'

  memcached:
    image: memcached
    restart: always
    ports:
      - '11211:11211'

  db:
    image: postgres
    restart: always
    volumes:
      - './postgres-data:/var/lib/postgresql/data'
    ports:
      - '5432:5432'
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: passwordpasswordpassword
      POSTGRES_DB: postgres

  housekeeper:
    image: flashbots/mev-boost-relay:sha-634cce7
    environment:
      GENESIS_FORK_VERSION: "0x10000001"
      GENESIS_VALIDATORS_ROOT: "0x6e1583608783377f139b7bac661aff7d5cca656af16ac34f9d0eda27dbf1316c"
      BELLATRIX_FORK_VERSION: "0x30000001"
      CAPELLA_FORK_VERSION: "0x40000001"
      DENEB_FORK_VERSION: "0x50000001"
    command:
      housekeeper
      --network custom
      --db postgres://postgres:passwordpasswordpassword@db:5432/postgres?sslmode=disable
      --beacon-uris ${BEACON_URL}
      --redis-uri redis:6379
    depends_on:
      - redis
      - memcached
      - db

  api:
    image: flashbots/mev-boost-relay:sha-634cce7
    environment:
      GENESIS_FORK_VERSION: "0x10000001"
      GENESIS_VALIDATORS_ROOT: "0x6e1583608783377f139b7bac661aff7d5cca656af16ac34f9d0eda27dbf1316c"
      BELLATRIX_FORK_VERSION: "0x30000001"
      CAPELLA_FORK_VERSION: "0x40000001"
      DENEB_FORK_VERSION: "0x50000001"
    ports:
      - "9062:9062"
    command: 
      api
      --network custom
      --secret-key 0x607a11b45a7219cc61a3d9c5fd08c7eebd602a6a19a977f8d3771d5711a550f2
      --db postgres://postgres:passwordpasswordpassword@db:5432/postgres?sslmode=disable
      --beacon-uris ${BEACON_URL}
      --blocksim ${EXECUTION_URL}
      --listen-addr 0.0.0.0:9062
      --redis-uri redis:6379
      --memcached-uris memcached:11211
    depends_on:
      - redis
      - memcached
      - db

  website:
    image: flashbots/mev-boost-relay:sha-634cce7
    environment:
      GENESIS_FORK_VERSION: "0x10000001"
      GENESIS_VALIDATORS_ROOT: "0x6e1583608783377f139b7bac661aff7d5cca656af16ac34f9d0eda27dbf1316c"
      BELLATRIX_FORK_VERSION: "0x30000001"
      CAPELLA_FORK_VERSION: "0x40000001"
      DENEB_FORK_VERSION: "0x50000001"
    ports:
      - "9060:9060"
    command:
      website
      --network custom
      --db postgres://postgres:passwordpasswordpassword@db:5432/postgres?sslmode=disable
      --listen-addr 0.0.0.0:9060
      --redis-uri redis:6379
      --link-beaconchain https://beacon.fusionist.io/
      --link-data-api /
      --link-etherscan https://explorer-endurance.fusionist.io/
    depends_on:
      - redis
      - memcached
      - db
```

```bash title="start.sh"
export IP_ADDRESS=$(curl -4 ifconfig.io)

if [ -z "$IP_ADDRESS" ]; then
    echo "Failed to retrieve IP address"
    exit 1
fi

echo "Using IP address: $IP_ADDRESS"

export BEACON_URL=http://<beacon-node-ip>:5052
export EXECUTION_URL=http://<execution-node-ip>:8545

docker compose up -d
```