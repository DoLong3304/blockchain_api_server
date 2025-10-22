# Blockchain API Server

A TypeScript/Express server that exposes a unified REST API to query balances, gas fees, prices, price history, transactions, NFTs, token metadata, and simple portfolio summaries across multiple chains (Bitcoin + EVM networks).

- EVM support via Alchemy + ethers (Ethereum, Polygon, Arbitrum, Optimism, Base)
- Bitcoin support via Blockstream public API
- Market data via CoinGecko (single and batch)
- Works with both simple and CAIP asset ID formats

## Features

- Balances
  - Native coins and ERC-20 (EVM via Alchemy); Bitcoin balances via Blockstream
- Gas prices
  - Legacy and EIP-1559 (base, priority) for EVM networks
- Prices and price history
  - Powered by CoinGecko; batch prices supported
- Transactions
  - EVM transfers via Alchemy; Bitcoin transactions via Blockstream
- NFTs
  - Owners for a token or collection, NFTs held by an address, NFT metadata (via Alchemy)
- Token metadata
  - Native and ERC-20 (name, symbol, decimals, total supply)
- Portfolio summary
  - Combines balances and prices to compute total value
- Clean error envelope and input validation
  - Helpful error codes (e.g., INVALID_ADDRESS, MISSING_API_KEY, UNSUPPORTED_NETWORK)

## Tech stack

- Node.js + TypeScript
- Express, CORS, dotenv
- Axios for HTTP
- ethers v6 for EVM RPC
- Providers: Alchemy, CoinGecko, Blockstream

## Supported networks

- EVM: ethereum, polygon, arbitrum, optimism, base
- Bitcoin: bitcoin

## Requirements

- Node.js 18+ (recommended) and npm
- Alchemy API key for EVM features (required for most EVM endpoints)
- Optional CoinGecko API key (public endpoints often work without a key, but rate limits apply)

## Installation

1. Clone and install

```powershell
# From PowerShell
git clone https://github.com/DoLong3304/blockchain_api_server.git
cd blockchain_api_server
npm install
```

2. Create a .env file

```env
# .env
COINGECKO_API_KEY=your_coingecko_api_key
ALCHEMY_API_KEY=your_alchemy_api_key
BITCOIN_API_URL=https://blockstream.info/api
PORT=3000
NODE_ENV=development
```

3. Run in development (TypeScript)

```powershell
npm run dev
```

4. Build and run in production

```powershell
npm run build
npm start
```

The server logs something like:

```
🚀 Blockchain API Server is running on http://localhost:3000
📊 API Documentation: http://localhost:3000/api
```

Health check: GET http://localhost:3000/

## API overview

Base path: `/api`

- GET /balance/:address/:assetId
- GET /gas/:networkId?type=legacy|eip1559
- GET /price/:assetId?currency=usd
- GET /price/:assetId/history?days=7&currency=usd
- GET /history/:address/:assetId?page=1&limit=50
- GET /nft/owners/:contractAddress/:networkId?tokenId=...
- GET /nft/owned/:owner/:networkId?contractAddress=0x...
- GET /token/metadata/:assetId
- GET /nft/metadata/:assetId
- POST /balances body: { address, assetIds: string[] }
- POST /prices body: { assetIds: string[], currency?: string }
- POST /portfolio body: { address, assetIds: string[], currency?: string }

All responses follow a common envelope:

```json
{
  "success": true,
  "data": { ... },
  "timestamp": 1690000000000
}
```

On error:

```json
{
  "success": false,
  "error": {
    "code": "INVALID_ADDRESS",
    "message": "Invalid address format",
    "details": null
  },
  "timestamp": 1690000000000
}
```

## Asset ID formats

This API accepts both simple (legacy) and CAIP-style asset identifiers.

- Bitcoin
  - Simple: `bitcoin`
  - CAIP: `bip122:000000000019d6689c085ae165831e93/slip44:0`
- Native EVM coins
  - Simple: `ethereum`, `polygon`, `arbitrum`, `optimism`, `base`
  - CAIP: `eip155:1/slip44:60` (Ethereum mainnet example)
- ERC-20
  - Simple: `ethereum:0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48` (USDC)
  - CAIP: `eip155:1/erc20:0xA0b8...6eB48`
- NFTs
  - Simple: `ethereum:0x...Contract:1234`
  - CAIP: `eip155:1/erc721:0x...Contract:1234` or `eip155:1/erc1155:0x...:1234`

The helper `parseAssetId()` determines network and asset type from any of the above.

## Endpoint details and examples

Below are quick examples using PowerShell's Invoke-RestMethod. Replace host and params as needed.

- Balance (EVM native or ERC-20; Bitcoin)

```powershell
# ETH balance (native)
Invoke-RestMethod -Uri "http://localhost:3000/api/balance/0x742d35Cc6634C0532925a3b844Bc454e4438f44e/ethereum" -Method Get

# ERC-20 balance (USDC on Ethereum)
Invoke-RestMethod -Uri "http://localhost:3000/api/balance/0x742d35Cc6634C0532925a3b844Bc454e4438f44e/ethereum:0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48" -Method Get

# Bitcoin balance
Invoke-RestMethod -Uri "http://localhost:3000/api/balance/bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh/bitcoin" -Method Get
```

- Gas price (EVM)

```powershell
Invoke-RestMethod -Uri "http://localhost:3000/api/gas/ethereum?type=eip1559" -Method Get
```

- Price and price history (CoinGecko)

```powershell
# Current price of Bitcoin (simple format)
Invoke-RestMethod -Uri "http://localhost:3000/api/price/bitcoin?currency=usd" -Method Get

# ETH price history for last 7 days (CAIP)
Invoke-RestMethod -Uri "http://localhost:3000/api/price/eip155:1/slip44:60/history?days=7&currency=usd" -Method Get
```

- Transactions

```powershell
Invoke-RestMethod -Uri "http://localhost:3000/api/history/0x742d35Cc6634C0532925a3b844Bc454e4438f44e/ethereum?page=1&limit=25" -Method Get
```

- NFTs (Alchemy)

```powershell
# Owners of a specific token
Invoke-RestMethod -Uri "http://localhost:3000/api/nft/owners/0x...Contract/ethereum?tokenId=1234" -Method Get

# Owners for a collection (no tokenId)
Invoke-RestMethod -Uri "http://localhost:3000/api/nft/owners/0x...Contract/ethereum" -Method Get

# NFTs owned by an address (optionally filter by contract)
Invoke-RestMethod -Uri "http://localhost:3000/api/nft/owned/0x...Owner/ethereum?contractAddress=0x...Contract" -Method Get

# NFT metadata (CAIP)
Invoke-RestMethod -Uri "http://localhost:3000/api/nft/metadata/eip155:1/erc721:0x...Contract:1234" -Method Get
```

- Token metadata

```powershell
# Native
Invoke-RestMethod -Uri "http://localhost:3000/api/token/metadata/ethereum" -Method Get

# ERC-20 (simple)
Invoke-RestMethod -Uri "http://localhost:3000/api/token/metadata/ethereum:0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48" -Method Get
```

- Batch and portfolio

```powershell
# Multiple balances
Invoke-RestMethod -Uri "http://localhost:3000/api/balances" -Method Post -ContentType "application/json" -Body (@{
  address = "0x742d35Cc6634C0532925a3b844Bc454e4438f44e";
  assetIds = @("ethereum", "ethereum:0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48", "bitcoin")
} | ConvertTo-Json)

# Multiple prices
Invoke-RestMethod -Uri "http://localhost:3000/api/prices" -Method Post -ContentType "application/json" -Body (@{
  assetIds = @("bitcoin", "ethereum", "eip155:1/erc20:0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48");
  currency = "usd"
} | ConvertTo-Json)

# Portfolio summary
Invoke-RestMethod -Uri "http://localhost:3000/api/portfolio" -Method Post -ContentType "application/json" -Body (@{
  address = "0x742d35Cc6634C0532925a3b844Bc454e4438f44e";
  assetIds = @("ethereum", "eip155:1/erc20:0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48")
} | ConvertTo-Json)
```

## Project structure

```
package.json
server.ts
src/
  app.ts                  # Express app, middleware, health check, routes mount
  router/
    blockchain.router.ts  # Route definitions
  controller/
    blockchain.controller.ts # Request validation and shaping responses
  service/
    blockchain.service.ts # Orchestration across providers
  provider/
    alchemy.ts            # EVM balance/gas/history/NFTs/metadata via Alchemy
    coingecko.ts          # Prices and history via CoinGecko
    bitcoin.ts            # Bitcoin balance/history via Blockstream
  data/
    networks.ts           # Network constants and mappings
    parser.ts             # AssetId parsing, validators, unit converters
    types.ts              # Typed API response models
```

## Configuration notes

- Alchemy
  - Set `ALCHEMY_API_KEY` to enable EVM features.
  - Supported network IDs: `ethereum`, `polygon`, `arbitrum`, `optimism`, `base`.
- CoinGecko
  - Public endpoints work without an API key but are heavily rate-limited; set `COINGECKO_API_KEY` if you have one.
- Bitcoin
  - Uses Blockstream API by default; override with `BITCOIN_API_URL`.
- CORS
  - CORS is enabled by default via `cors()` middleware.

## Troubleshooting

- 400 INVALID_ADDRESS
  - Ensure the address format matches the asset/network. EVM addresses must be 0x + 40 hex chars. Bitcoin supports legacy, P2SH, and bech32.
- 400 INVALID_GAS_TYPE
  - `type` must be `legacy` or `eip1559`.
- 400/404 Price not found
  - For tokens, prefer CAIP `eip155:<chainId>/erc20:<contract>`.
- 400 UNSUPPORTED_NETWORK or 400 UNSUPPORTED_ASSET_TYPE
  - Check that the network/asset is one of the supported sets.
- 401/403 MISSING_API_KEY or invalid API key
  - Set `ALCHEMY_API_KEY`; verify it has access to the chosen network.
- Rate limits / timeouts
  - CoinGecko and Alchemy have rate limits. Add caching, backoff, or a key with higher quotas.
- Local development
  - Use `npm run dev` for ts-node. For production, `npm run build` then `npm start`.

If you continue to see issues, enable more logging around provider calls or print upstream response payloads in the error `details` field.

## Development

- Add networks
  - Update `NETWORKS`, `ALCHEMY_NETWORKS`, and `COINGECKO_COIN_IDS` in `src/data/networks.ts`.
- Extend providers
  - Implement missing features in `provider/*` and wire them in `BlockchainService`.
- Tests
  - Add a test runner and basic endpoint tests; consider mocking upstream APIs.

## License

ISC © 2025
