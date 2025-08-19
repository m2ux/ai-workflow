# Phase 2.4: Witness Implementation (Cardano)

Witnessing allows the system to observe and react to on-chain events, including committee rotations and vault operations. Following the Bitcoin pattern, this involves querying Cardano blocks, filtering transactions relevant to the system, and recording UTXO changes on-chain. The implementation handles Cardano's block structure and transaction format while integrating with the existing witnessing infrastructure. Library selection between Cardano serialization lib and Pallas requires evaluation based on project needs.

Deliverables:
- Follow Bitcoin pattern: query blocks, filter transactions, record UTXOs on-chain
- Library choice: Cardano serialization lib vs Pallas (Pallas recommended for multichain-engine)
- Implement block witnessing for committee rotation support

Notes:
- This section is extracted verbatim from the main plan to serve as a standalone implementation brief for Phase 2.4.

## Compatibility assessment: reuse of existing witnessing features

The following features from existing chains can be reused for Cardano (EUTXO, probabilistic finality, chain-sync via Ogmios). Where provenance lists both, the pattern is shared; otherwise it indicates the closest source to follow.

| Witnessing feature | Provenance | Rationale for compatibility with Cardano |
| --- | --- | --- |
| Block ingestion via retrying RPC client with backoff | Bitcoin | Cardano’s Ogmios client supports robust RPC; retry/backoff pattern from BTC `BtcRetryRpcClient` applies to `AdaRetryRpcClient` for read ops (blocks, txs, UTXOs, tip). |
| ChainSync-backed block/tx retrieval | Bitcoin | BTC uses a polling client; Cardano additionally supports Chain Sync (already implemented). Use Chain Sync where available, fall back to HTTP RPC for tip/params. |
| Processed-block tracking (RLE bitmap) and persistence | Bitcoin | Identical pattern to avoid reprocessing and to survive restarts. Store processed Cardano block heights using existing persistent DB. |
| Reorg handling with confirmation depth and retry policy | Bitcoin | Ouroboros has probabilistic finality like BTC. Apply a BTC-like confirmation depth and simple retry policy (Cardano-specific constants). |
| Lag-safety/strictly-monotonic chain source wrappers | Bitcoin | Reuse witness/common `lag_safety` and `strictly_monotonic` wrappers with Cardano block indices. |
| Chunked scanning pipeline (time- or index-ranged) | Bitcoin | Reuse `chunked_chain_source` to bound memory and throughput when scanning Cardano blocks. |
| Deposit detection by UTXO scanning | Bitcoin | Cardano is EUTXO; reuse BTC pattern to match outputs to known deposit/vault addresses (Cardano address codec differs, but logic is the same). |
| Vault UTXO accounting and consolidation planning | Bitcoin | Consolidation logic (selection thresholds/limits) maps to Cardano UTXOs; implement Cardano-specific size/fee cost model but reuse flow. |
| Fee/tracked data witnessing | Bitcoin, Ethereum | Maintain `CardanoTrackedData` similar to BTC/EVM tracked data. Populate from `get_protocol_parameters` (min fee A/B, etc.) and tip/slot for replay protection. |
| Replay protection provider | Ethereum | Reuse the provider pattern; Cardano uses slot-based `CardanoReplayProtection` analogous to EVM’s nonce/chain-id structure. |
| Witness period scheduling utilities | Bitcoin, Ethereum | Reuse `witness_period` helpers for routing/scheduling batches; Cardano WITNESS_PERIOD can follow conservative default initially. |
| Metrics/health checks hooks | Bitcoin, Ethereum | Reuse existing metrics/health surfaces in engine; wire Cardano RPC health/tip checks. |
| Deposit channel lifecycle hooks | Bitcoin, Ethereum | Generic hooks (fetch scheduled/completed, recycle address) apply to Cardano channels. |

Additional features to reuse from other supported chains:

| Witnessing feature | Provenance | Rationale for compatibility with Cardano |
| --- | --- | --- |
| Finalised vs unfinalised chain sources | Polkadot | Cardano consensus (Ouroboros) exposes finality via confirmations; mirror DOT’s separate sources to optionally track tips (unfinalised) while committing only after depth. |
| Subscription-based header stream with resilient restart | Polkadot | Cardano Chain Sync over Ogmios provides a streaming protocol; reuse DOT’s subscribe-and-restart pattern with timeouts and stream restarts on stalls. |
| Runtime-agnostic “tracked data” update pipeline | Polkadot | Use common chain-tracking builder to derive Cardano tracked data (tip height, protocol params snapshot) and submit to state chain. |
| Chunked-by-time chain tracking builder | Polkadot | Reuse time-chunked processing to bound memory/latency for Cardano header/UTXO scans. |
| Logging wrapper for chain source | Common | Enable structured per-block logging for Cardano stream using existing source logging adapter. |
| Lag-safety wrapper (reorg margin) | Common | Apply margin-based safe emission of block ranges before finality; Cardano can use confirmations or slot-based offset similar to BTC behavior. |
| Strictly-monotonic wrapper | Common | Ensure non-decreasing block indices emitted to downstream processors; compatible with Cardano’s ordered Chain Sync stream. |
| ExternalChainSource abstraction | Common | Treat Cardano as an external chain source, enabling composition with wrappers (logging, lag-safety, chunking). |
| Gas/fee tracked-data pattern via per-block sampling | Arbitrum | For Cardano, adapt to protocol parameters per epoch/era (min fee A/B), analogous to ARB/EVM per-block fee sampling; store in tracked data. |
| Retry-limit semantics in event retrieval | Polkadot | Use bounded retries for unfinalised streams (bad/missing blocks) and unlimited for finalised; map to Chain Sync staleness handling. |

Non-applicable (not required for Cardano witnessing):
- EVM log filtering/bloom-based event scanning (Ethereum-only). Cardano has no logs; use UTXO scanning.
- EVM receipt correlation and contract address deployments (Ethereum-only). Cardano uses UTXOs and addresses.

