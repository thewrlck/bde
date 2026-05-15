# Bitcoin Dockerized Environment

This repository contains a modular, dockerized development environment for Bitcoin-related services. The environment is designed to be easily deployable, configurable, and extensible.

## Architecture

The environment follows a modular architecture where each service is:
- Independently containerized
- Built from source
- Configurable through environment variables
- Data-persistent

## Current Services

The environment currently includes:

1. **Bitcoin Core Node** (v31.0)
   - Full Bitcoin node with RPC and P2P ports exposed
   - ZMQ publishers for rawblock, rawtx, and hashblock
   - Built without wallet support; IPC enabled
   - Includes transaction indexing and server mode enabled
   - Configurable RPC authentication

2. **Electrum Server** (v3.3.0)
   - Built on top of mempool/electrs (lightmode)
   - Provides Electrum protocol support
   - Connects to a remote Bitcoin node via `BITCOIN_RPC_URL`, authenticating with `RPC_USER`/`RPC_PASS`

3. **Ordinals Server** (v0.23.1)
   - Built on top of ord
   - Provides Ordinals protocol support
   - Includes indexing for addresses, runes, sats, and transactions

4. **Mempool** (v3.3.1)
   - Block explorer and mempool visualizer from https://github.com/mempool/mempool
   - Built from source: API (Node.js + Rust GBT), web frontend (Angular + nginx), MariaDB
   - Configured with `MEMPOOL_BACKEND=esplora`, pointing at the existing electrs container's HTTP REST API (enabled via `--http-addr` on port 3000)
   - Connects to Bitcoin Core RPC for `CORE_RPC_*` queries

## Adding New Services

To add a new service to this environment:

1. Create a new `Dockerfile.<service-name>` in the root directory
2. Create a new `docker-compose-<service-name>.yml` file
3. Add a new directory for service data persistence
4. Update this README with the new service information
5. Add any new environment variables to `.env.example`

### Service Template

```yaml
# docker-compose-<service-name>.yml
services:
  <service-name>:
    build:
      context: .
      dockerfile: Dockerfile.<service-name>
    container_name: <service-name>
    volumes:
      - ./<service-name>:/path/to/data
    ports:
      - ${SERVICE_PORT}:<internal_port>
    environment:
      - VAR1=${VAR1}
      - VAR2=${VAR2}
    command:
      [
        "./<service-name>"
      ]
```

## Prerequisites

- Docker
- Docker Compose
- Git

## Environment Variables

Create a `.env` file in the root directory, copying `.env.example` with the following variables:

```env
# Bitcoin Core
RPC_PORT=8332
P2P_PORT=8333
ZMQ_RAWBLOCK_PORT=28332
ZMQ_RAWTX_PORT=28333
ZMQ_HASHBLOCK_PORT=28334
RPC_AUTH=<your_rpc_auth_string>
RPC_USER=<your_rpc_username>
RPC_PASS=<your_rpc_password>

# Electrum Server
ELECTRS_PORT=50001
ESPLORA_PORT=3000
BITCOIN_RPC_URL=<host:port of remote bitcoin node, e.g. node.example.com:8332>

# Ordinals Server
ORDINALS_PORT=80

# Mempool
ESPLORA_REST_API_URL=http://host.docker.internal:3000
CORE_RPC_HOST=host.docker.internal
CORE_RPC_PORT=8332
MEMPOOL_WEB_PORT=8081
MEMPOOL_DB_PASSWORD=<mempool db user password>
MEMPOOL_DB_ROOT_PASSWORD=<mempool db root password>

# Add new service variables here
```

## Directory Structure

```
.
├── bitcoin/                       # Bitcoin Core data directory
├── electrs/                       # Electrum server data directory
├── ordinals/                      # Ordinals server data directory
├── mempool/                       # Mempool data (cache + mariadb)
├── Dockerfile.bitcoin             # Bitcoin Core Dockerfile
├── Dockerfile.electrs             # Electrum server Dockerfile
├── Dockerfile.ordinals            # Ordinals server Dockerfile
├── Dockerfile.mempool-backend     # Mempool API Dockerfile
├── Dockerfile.mempool-frontend    # Mempool web Dockerfile
├── docker-compose-bitcoin.yml
├── docker-compose-electrs.yml
├── docker-compose-ordinals.yml
└── docker-compose-mempool.yml
```

## Usage

Each service can be started independently:

```bash
# Start Bitcoin Node
docker compose -f docker-compose-bitcoin.yml up -d

# Start Electrum Server
docker compose -f docker-compose-electrs.yml up -d

# Start Ordinals Server
docker compose -f docker-compose-ordinals.yml up -d

# Start Mempool (electrs must be running with --http-addr enabled)
docker compose -f docker-compose-mempool.yml up -d

# Start a new service
docker compose -f docker-compose-<service-name>.yml up -d
```

## Ports

Each service exposes its own ports:
- Bitcoin Core RPC: 8332
- Bitcoin Core P2P: 8333
- Bitcoin Core ZMQ rawblock: 28332
- Bitcoin Core ZMQ rawtx: 28333
- Bitcoin Core ZMQ hashblock: 28334
- Electrum Server: 50001
- Electrs HTTP REST (esplora): 3000
- Ordinals Server: 80
- Mempool web: 8081
- New Service: <port>

## Data Persistence

All services maintain data persistence in their respective directories:
- Bitcoin Core data: `./bitcoin`
- Electrum server data: `./electrs`
- Ordinals server data: `./ordinals`
- Mempool data: `./mempool` (`./mempool/data` for backend cache, `./mempool/mysql` for mariadb)
- New Service data: `./<service-name>`

## Building from Source

Each service is built from source using their respective Dockerfiles. The build process is service-specific:

1. Bitcoin Core: Built from source using CMake
2. Electrum Server: Built from source using Cargo (Rust)
3. Ordinals Server: Built from source using Cargo (Rust)
4. Mempool: Built from source — API via Node.js + Rust (rust-gbt napi-rs addon), frontend via Angular, served by nginx
5. New Service: <build process>

## Contributing

1. Fork the repository
2. Create a new branch for your service
3. Add your service following the template structure
4. Update the README with your service information
5. Submit a pull request

## Security Notes

- RPC interfaces should be properly secured in production
- Use strong authentication credentials
- Consider using reverse proxies for public-facing services

## License

This project uses components with their respective licenses:
- Bitcoin Core: MIT
- Electrum Server: MIT
- Ordinals: MIT
- Mempool: AGPL-3.0
- New Service: <license> 
