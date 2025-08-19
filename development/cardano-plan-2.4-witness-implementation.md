# Phase 2.4: Witness Implementation (Cardano)

Witnessing allows the system to observe and react to on-chain events, including committee rotations and vault operations. Following the Bitcoin pattern, this involves querying Cardano blocks, filtering transactions relevant to the system, and recording UTXO changes on-chain. The implementation handles Cardano's block structure and transaction format while integrating with the existing witnessing infrastructure. Library selection between Cardano serialization lib and Pallas requires evaluation based on project needs.

### Chain-agnostic witnessing capabilities required by DEX operations

- **Deterministic event model and idempotency**: Ingest blocks/transactions deterministically and emit deposit/egress events exactly once despite restarts or replays.
- **Finality and reorg tolerance**: Apply configurable confirmation depth and reorg handling so upstream state never regresses.
- **Deposit abstraction**: Uniform interface to discover, attribute, and validate deposits across chains (address parsing, amount thresholds, asset identity).
- **Egress confirmation tracking**: Correlate broadcast transactions and mark items complete only after finality; handle change accounting.
- **Tracked-data pipeline to state chain**: Periodic submission of compact chain metrics (height, fees/limits, replay tokens) used by pallets and builders.
- **Throughput and backpressure**: Chunked processing and bounded buffering to maintain stable latency under variable block/tx rates.
- **Persistence and recovery**: Durable storage of processed ranges/watermarks and necessary ephemeral context for safe resumption.
- **Resilience and reliability**: Retry/backoff around RPC, multi-endpoint failover, and health checks to withstand node outages.
- **Normalization and validation**: Strict address/asset normalization and basic structural validation to avoid false positives/negatives.
- **Security and resource safeguards**: Input size limits, spam resistance, and safe parsing to mitigate DoS vectors.
- **Configurability**: Per-chain constants (confirmation depth, witness period, fee sampling cadence) and feature flags for environment differences.
- **Testability**: Mocks/simulators for RPC and chain streams; deterministic tests for deposit classification and egress confirmation.
- **Observability and auditability**: Structured logs, metrics, and optional audit trails for critical state transitions (deposits/settlements).

- **Backfill and gap repair**: Detect ingest gaps after downtime and re-scan historical ranges deterministically, reconciling without duplicate event emission.
- **Scheduling, isolation, and fairness**: Per-chain concurrency limits and fair work queues so a slow/failed chain cannot starve others or block different work types.
- **Endpoint diversity and optional quorum**: Multiple providers per chain (primary/backup) with configurable selection and optional cross-checks for critical reads.
- **Circuit breakers and health gating**: Gate state updates and event emission behind health checks; pause/resume per chain when lag or error rates breach thresholds.
- **Latency SLOs and staleness detection**: Track end-to-end detection latency; raise alerts or degrade gracefully when SLOs are exceeded or ingest becomes stale.
- **Exactly-once publishing semantics**: Define idempotency keys for deposits and egress confirmations; ensure retries to the state chain remain idempotent.
- **Runtime safety toggles**: Safe-mode controls per chain and feature (deposits, egress, tracking) to isolate incidents without full downtime.
- **Schema/versioning for tracked data**: Forward/backward-compatible schema and migrations so pallet/builder upgrades do not break witness submissions.
- **Resource safeguards and isolation**: Per-chain CPU/memory budgets, bounded batch sizes, and dynamic backpressure to protect node and worker stability.
- **Auditable replay**: Ability to replay from retained chain data and produce identical outputs using deterministic pipelines and pinned configuration snapshots.

### Purpose and required characteristics for DEX functionality (informed by BTC/ETH)

- #### Continuous chain state tracking
  Maintain a coherent view of Cardano chain progress and publish compact tracked data to the state chain so downstream pallets have up‑to‑date height/params for all DEX flows.
  - Maintain canonical tip height and submit tracked data to the state chain each epoch (cf. `witness/common/chunked_chain_source/chain_tracking.rs`, `witness/eth/chain_tracking.rs`, `witness/dot/dot_chain_tracking.rs`).
  - For Cardano: capture slot/epoch-derived height and protocol parameter snapshot relevant to fees and min UTXO.

- #### Deposit discovery and attribution (ingress)
  Reliably detect inbound funds to vault/deposit addresses (ADA and tokens), attribute them once, and surface deposits to the state chain to unlock order funding and flows.
  - Scan blocks and attribute inbound funds to known deposit/vault addresses; deduplicate idempotently across restarts (BTC: `witness/btc/deposits.rs`; EVM: `witness/evm/evm_deposits.rs`).
  - Cardano: parse EUTXO outputs, support ADA and native tokens, respect min-ADA on outputs and multi-asset bundles.
  - Enforce minimum deposit thresholds and produce well-formed deposit events (cf. existing pallet interfaces).

- #### Egress confirmation and settlement
  Track outbound transactions until they reach configured finality and then mark withdrawals/settlements complete while accounting for change back to the vault.
  - Observe broadcast transactions and confirm finalization to mark egress items complete (BTC: `witness/btc/vault_swaps.rs`; EVM: receipts via `EvmRetryRpcApi`).
  - Cardano: confirm by tx hash and output matching after required confirmation depth; handle change outputs to vault.

- #### Vault UTXO lifecycle management signals
  Monitor vault UTXO shape and costs to trigger consolidation/rotation suggestions so the vault remains efficient and transactions are constructible under size/fee limits.
  - Identify fragmentation and signal when consolidation or rotation is advisable (pattern from BTC UTXO handling in `witness/btc/*`).
  - Cardano: apply size/fee-aware consolidation heuristics compatible with protocol parameters and min-ADA rules.

- #### Finality and reorg safety
  Prevent reorg-induced inconsistencies by applying confirmation depth and lag-safety so only safe, monotonic observations are committed to the state chain.
  - Apply confirmation depth and lag safety before committing witness effects (BTC pattern; common `lag_safety.rs`, `strictly_monotonic.rs`).
  - Cardano: use a configurable confirmation depth; Chain Sync stream plus lag-safety before state updates.

- #### Chunked, resilient ingestion
  Bound latency and memory while ingesting by processing in small chunks and using restartable streams to withstand node hiccups and long rollforwards.
  - Bound memory/latency with chunked-by-time/index pipelines and restartable streams (common chunked sources; DOT subscribe-and-restart pattern).
  - Cardano: Ogmios Chain Sync streaming with timeout/restart and chunking.

- #### Replay protection context for builders
  Summary: Supply builders with up‑to‑date replay tokens (e.g., slot/epoch details) and constraints to ensure constructed transactions are unique and accepted on-chain.
  - Provide replay data to transaction builders (EVM uses nonces/base fee; ARB adds L1 cost via node interface; see `eth/chain_tracking.rs`, `arb/chain_tracking.rs`).
  - Cardano: tracked data should include current slot/era-derived replay token and fee parameters used by builders.

- #### Fee and parameter sampling
  Keep current fee and size/cost parameters to price operations, estimate refunds, and select viable transaction constructions across network conditions.
  - Periodically sample fee-related metrics for pricing/estimation (EVM fee history; ARB L1/L2 base fees).
  - Cardano: capture `min_fee_coefficient`, `min_fee_constant`, `max_tx_size`, and min UTXO parameters to inform fee/refund logic.

- #### Persistence and idempotency
  Persist progress (processed ranges/watermarks) so witnessing is restart‑safe and events are emitted exactly once to upper layers.
  - Store processed block ranges and watermarks durably (DB RLE bitmap via `db/persistent.rs`) to survive restarts and prevent duplicate events.

- #### Health, metrics, and backoff
  Expose health/throughput metrics and wrap RPC with retries/backoff to keep witnessing serviceable under partial outages.
  - Expose progress metrics and use retry/backoff around RPC (BTC/ETH retry clients; common retrier).
  - Cardano: instrument Ogmios health/tip checks; reuse retry RPC wrapper.

- #### Address and encoding correctness
  Strictly validate and normalize Cardano addresses and asset encodings to avoid false attributions and ensure deterministic matching.
  - Ensure robust address parsing/encoding for deposit matching (cf. BTC script parsing, EVM checksums).
  - Cardano: Bech32 addresses and multi-asset policy/id handling; strict validation to avoid false positives.

#### Cardano-specific additional requirements (no BTC/ETH equivalent today)

- Era/slot-aware height computation and stability window
  - Maintain era summaries and convert slots to block heights across era boundaries (variable slot lengths).
  - Apply an Ouroboros stability window for reorg safety; persist and honor Chain Sync rollbacks beyond simple confirmation depth.

- Ogmios Chain Sync intersection management
  - Persist the last intersection point; on restart, resume via FindIntersect and handle RollForward/RollBackward streams.
  - Use back-pressure and bounded buffers to avoid memory growth on long rollforwards.

- EUTXO multi-asset deposit classification
  - Parse outputs with multi-asset bundles; match policy IDs/asset names against supported assets and vault/deposit addresses.
  - Enforce dynamic min-ADA per-output (depends on size and bundle contents) to reject dust/invalid deposits.

- Address type coverage and validation
  - Handle enterprise/base/pointer addresses (Shelley), Byron legacy addresses, and stake references; validate network tags and HRPs.
  - Treat script (Plutus) vs non-script addresses distinctly when classifying deposits.

- Transaction semantics filtering unique to Cardano
  - Identify and ignore collateral inputs/outputs from failed-script transactions when attributing deposits.
  - Distinguish reference inputs and inline datums which do not transfer value to vaults.

- Protocol parameter snapshot for builders
  - Track fee constants (min fee A/B), max tx size, min UTXO rules, and per-era cost models/step prices for downstream fee/refund estimation.
  - Expose in `CardanoTrackedData` for use by transaction builders/evaluators.

- Native token handling nuances
  - Treat token amounts as base units (no on-chain decimals); keep witnessing strictly on-chain exactness.
  - Support asset-specific deposit thresholds (policy-scoped) if required by DEX policy.

- Consolidation heuristics aware of min-ADA and reference scripts
  - Consolidation planning must account for larger outputs due to reference scripts/datums; compute min-ADA correctly post-merge.

- Slot/epoch timing for replay protection
  - Provide a slot-based replay token and current epoch timing to builders; ensure consistency around epoch transitions.


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

## Library evaluation: Pallas vs Cardano Serialization Library (CSL)

| Criteria | Pallas (Rust) | Cardano Serialization Library (CSL) |
| --- | --- | --- |
| Purpose fit for witnessing | Strong: native Rust types/codecs for blocks/txs/addresses; supports Ouroboros mini-protocols if we ever run Chain Sync without Ogmios | Adequate for serialization/addresses/tx building; less focused on mini-protocols; typically used for tx construction, wallet flows |
| Integration with current design (Ogmios JSON) | Good: keep Ogmios for transport; use Pallas for address/CBOR decode where needed | Works, but API is oriented to serialization; may add weight when most ingest is JSON via Ogmios |
| Rust ergonomics | Excellent: pure Rust crates on crates.io, zero-FFI/wasm; good type coverage across eras | Usable in Rust, but primarily designed for wasm bindings; ergonomics may be less idiomatic in Rust backends |
| Performance | High: zero-copy/streaming-friendly decoding patterns | Solid, but often geared toward portability; may be heavier if used beyond serialization
| Maintenance/coverage | Actively maintained, good ledger/era coverage; modular crates (primitives, network, codec) | Mature and widely used; however, newer work gravitates to CML; CSL usage in Rust servers is less common |
| Address/UTXO parsing | Provided via primitives; fits EUTXO use-cases for deposit detection | Supported; stable Bech32/CBOR handling |
| Transaction building (future phases) | Possible, but more manual; better paired with ledger-specific crates or CML | Strong: ready-made tx/metadata encoders; widely used in app-layer tooling |
| Packaging/reproducibility | Crates.io, versioned | Often consumed via git/wasm workflows; may require extra setup in Rust-only environments |
| Licensing | Permissive OSS | Permissive OSS |

Recommendation:
- For Phase 2.4 witnessing (ingest, UTXO/deposit detection, tracked-data): prefer Pallas + Ogmios (keep Ogmios for Chain Sync, use Pallas for types/CBOR/address utilities as needed).
- For later transaction construction/broadcast phases: consider introducing CSL (or CML) specifically for tx building and metadata serialization if required; keep witnessing path on Pallas.

