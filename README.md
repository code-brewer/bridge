# SMOOT-Bridge

**SMOOT-Bridge** is a decentralized, generic message-passing protocol for secure, bidirectional communication between blockchain ecosystems — spanning both EVM-compatible chains (Ethereum, Polygon) and non-EVM chains (Stellar/Soroban).

The protocol's flagship use case is cross-chain **token transfer** for [ERC-3643](https://www.erc3643.org/)-compliant (and ERC-20-compatible) tokens, using a lock/mint and burn/unlock mechanism relayed by an off-chain agent network. The underlying architecture is designed to be extended to additional chains and message types over time.

## How it works

| Contract / Entity       | Chain     | Role                                                                                          |
| ------------------------ | --------- | ---------------------------------------------------------------------------------------------- |
| **TokenHome**             | Ethereum  | Registers the original token; **locks** it when transferring out to another chain.              |
| **TokenRemote**           | Polygon   | **Mints**/registers the mapped token when a transfer arrives from the home chain.                |
| **Gateway**               | Both      | Message bus that relays cross-chain calls (`outboundCall` / `inboundCall`) between contracts.    |
| **SMOOT-Bridge Agents**   | Off-chain | Decentralized relayer network that watches `outboundCall` events and submits matching `inboundCall`s on the destination chain. |

**Transfer flow (Ethereum → Polygon, and back):**
1. User calls `send` on `TokenHome` (or `TokenRemote` for the return trip).
2. The token is locked (or burned) and the source `Gateway` emits an `outboundCall` event.
3. An Agent captures the event, signs, and relays it as an `inboundCall` to the `Gateway` on the destination chain.
4. The destination `Gateway` triggers `TokenRemote` to mint (or `TokenHome` to unlock/transfer) the asset to the recipient.

## Repository layout

| Path | Description |
| --- | --- |
| [`contracts/`](contracts) | Smart contracts, split per chain: [`contracts_ethereum`](contracts/contracts_ethereum), [`contracts_polygon`](contracts/contracts_polygon) (token, message-bridge, and NFT-market contracts), [`contracts_soroban`](contracts/contracts_soroban) (Stellar), and [`contracts_merkle_proofs`](contracts/contracts_merkle_proofs). |
| [`agent/`](agent) | The off-chain SMOOT-Bridge Agent (a.k.a. `cross-route`) that watches chains and relays cross-chain messages. |
| [`apps/`](apps) | Front-end applications: [`app_bridge_ui`](apps/app_bridge_ui) (current bridge UI, supports Stellar and Polygon), [`app_nftMarket`](apps/app_nftMarket) (NFT marketplace UI), and the deprecated [`app_erc3643`](apps/app_erc3643) (superseded by `app_bridge_ui`). |
| [`services/`](services) | Backend services: [`routeService_multiSig`](services/routeService_multiSig) (multi-sig message routing) and [`scanService_nftMarket`](services/scanService_nftMarket) (chain scanning for the NFT market). |
| [`deployment/`](deployment) | Deployment tooling and Dockerfiles for the agent and scan service. |
| [`sdks/`](sdks) | Client SDKs (reserved for future use). |
| [`design/`](design) | Design documents. |
| [`requirement/`](requirement) | Requirement documents. |
| [`docs/`](docs) | Additional docs, including deployment tutorials and event model reference. |
| [`test/`](test) | Integration/setup test scaffolding. |

## Getting started

Each component is independently runnable; see its own README for setup details:

- Bridge UI: [`apps/app_bridge_ui`](apps/app_bridge_ui/README.md)
- NFT market UI: [`apps/app_nftMarket`](apps/app_nftMarket/README.md)
- Agent: [`agent/README.md`](agent/README.md) — start via `agent/index.js`
- Route service: `node services/routeService_multiSig/src/startMsgRouter.js`
- Scan service: `node services/scanService_nftMarket/src/startScanChainService.js`
- Ethereum contracts: [`contracts/contracts_ethereum/README.md`](contracts/contracts_ethereum/README.md)
- Polygon contracts: [`contracts/contracts_polygon/README.md`](contracts/contracts_polygon/README.md)
- Soroban (Stellar) contracts: [`contracts/contracts_soroban/README.md`](contracts/contracts_soroban/README.md)

## License

Licensed under the [Apache License 2.0](LICENSE).

