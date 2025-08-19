# Cardano Integration Plan

## Overview

This document is a plan for integrating Cardano support into the Partner Chains MCS project. It outlines phases, milestones, and deliverables, and tracks progress against them. The plan follows established patterns from existing chains (Bitcoin, Ethereum, Polkadot, Arbitrum) and will be updated as milestones are completed.

## Implementation Status & Next Steps

### Phase 1: Foundation Complete
- **Core type system integration** - Cardano types implemented with required traits
- **Address infrastructure framework** - Basic structure for Cardano address handling
- **Crypto operations and multisig integration** - Ed25519 threshold signatures implemented
- **Transaction structure and API framework** - Base transaction types and API traits defined
- **Instance system integration** - Cardano integrated into chain instance system
- **Fee estimation API structure** - Framework established (implementation pending)
- **Deposit channel lifecycle framework** - State management structure in place
- **Block rewards integration** - ADA support added to block rewards pallet
- **P2P integration with message routing** - Cardano messages routed through P2P system
- **State chain observer integration** - Cardano multisig client integrated
- **Ceremony ID counter management** - Integration with ceremony system

### Phase 2: Core Implementation

1. **RPC Client Implementation**
   
   Status: ✅ Complete on branch `origin/issue/etcm12227_2` (compared to base `origin/issue/etcm12057`).

   Progress evidence (files changed between branches):
   - Implemented `AdaRpcClient` with full API in `multichain-engine/src/ada/rpc.rs` (block/tx fetch, submit, tip, UTXO, protocol params, network info, evaluate_transaction, evaluate_with_additional_utxos) using the Ogmios client.
   - Added `AdaRetryRpcClient` with exponential backoff and jitter in `multichain-engine/src/ada/retry_rpc.rs`.
   - Added mock client for testing in `multichain-engine/src/ada/mock_client.rs`.
   - Added integration and performance tests under `multichain-engine/src/ada/integration_tests/` and unit tests in `multichain-engine/src/ada/tests.rs`.
   - Extended Ogmios utilities: new chain sync modules and tests in `toolkit/utils/ogmios-client/`.

   Functional scope delivered:
   - Block by hash/height, transaction by hash, submit transaction (CBOR), tip, UTXO queries (incl. native tokens), protocol parameters, network info.
   - Execution unit evaluation endpoints (with and without additional UTXOs).
   - Optional Chain Sync integration for accurate block/tx retrieval.

2. **Address System** (may be needed for deposit channels)
   
   Cardano uses bech32 encoding for addresses with network-specific prefixes and stake delegation components. The current implementation uses 32-byte arrays as placeholder addresses, which are incompatible with Cardano networks. This task implements Cardano address derivation from cryptographic keys, supporting both payment and stake addresses, and handling network-specific formatting (mainnet vs testnet). Address handling is required for deposit channels, vault operations, and user-facing functionality.
   
   - Implement proper Bech32 Cardano addresses and derivation logic
   - Replace placeholder byte arrays with real address derivation
   - Add network-specific address format support

3. **Transaction Construction** (CBOR transaction building)
   
   Cardano transactions use CBOR (Concise Binary Object Representation) encoding with a structure that includes inputs, outputs, metadata, and witness sets. The current placeholder implementation uses hardcoded fees and empty transaction structures. This task implements CBOR transaction encoding that handles multiple inputs and outputs, references UTXOs correctly, and calculates appropriate fees. This supports transaction-related functionality, from simple transfers to multi-signature operations.
   
   - Implement CBOR transaction encoding with proper input/output handling
   - Replace hardcoded fees with dynamic calculation
   - Add proper UTXO reference handling

4. **Witness Implementation** (required for committee rotation)
   
   Witnessing allows the system to observe and react to on-chain events, including committee rotations and vault operations. Following the Bitcoin pattern, this involves querying Cardano blocks, filtering transactions relevant to the system, and recording UTXO changes on-chain. The implementation handles Cardano's block structure and transaction format while integrating with the existing witnessing infrastructure. Library selection between Cardano serialization lib and Pallas requires evaluation based on project needs.
   
   - Follow Bitcoin pattern: query blocks, filter transactions, record UTXOs on-chain
   - Library choice: Cardano serialization lib vs Pallas (Pallas recommended for multichain-engine)
   - Implement block witnessing for committee rotation support

5. **Fee Calculation** (linked to witnessing)
   
   Cardano fees are calculated based on transaction size, execution units for smart contracts, and current network parameters that change over time. Cardano's fee structure requires network parameters (min fee coefficient, UTXO cost per word, etc.) that are witnessed from the blockchain. This implementation cannot be done independently of witnessing since fee calculation depends on current network state. The system handles both transaction fees and scenarios involving metadata and multiple outputs.
   
   - Implement Cardano-specific fee algorithms using network parameters
   - Cannot be done independently - part of witnessing implementation

6. **UTXO Management** (linked to witnessing)
   
   Cardano's UTXO model requires management of unspent transaction outputs, including selection algorithms for transaction construction and tracking of UTXO states across committee rotations. Following the Bitcoin approach, UTXOs are tracked by witnessing blocks and recording changes on-chain. The implementation handles UTXO consolidation during committee changes, UTXO selection for transaction efficiency, and state maintenance across network operations. This supports vault operations and transaction construction.
   
   - Use same approach as Bitcoin: witness each block, filter transactions, record UTXOs on-chain
   - Implement UTXO selection algorithms and tracking infrastructure

7. **Batch Operations**
   
   Batch operations process multiple transactions in a single operation, reducing network overhead and improving system throughput. The batch API currently returns empty data instead of actual batched transactions. This implementation creates batch transaction structures, handles multiple transaction inputs and outputs, and accounts for Cardano's transaction size limits. Batch operations support rewards distribution and vault management operations where multiple recipients or multiple UTXO operations occur.
   
   - Implement actual batch transaction creation (currently returns empty data)
   - Add batch processing for multiple transactions
   - Optimize transaction throughput

8. **Cardano Side Logic/Scripts** (new task)
   
   This task creates the Cardano-side infrastructure to support the rewards use case, similar to the vault scripts developed for Bitcoin. The implementation includes setting up vault capabilities using Cardano's script system, creating funding mechanisms for testing, and developing operational scripts for the Yaci Dev Kit local testing environment. This supports end-to-end testing and demonstration of the rewards functionality. The scripts handle ADA deposits, committee key management, and reward distribution compatible with Cardano's UTXO model.
   
   - Vault capability needed for rewards use case (similar to Bitcoin approach)
   - Setup scripts for Yaci Dev Kit local testing environment
   - Create funding and testing scripts for ADA rewards

### Phase 3: Testing & Validation Only
- **Comprehensive Test Suite** - Add integration tests for all Cardano functionality
- **Testnet Integration** - Connect to Cardano testnet using Yaci Dev Kit for testing
- **Mock Infrastructure** - Complete mock Cardano node for unit testing
- **Address Validation** - Test bech32 encoding/decoding and derivation
- **Transaction Validation** - Verify CBOR encoding and network acceptance
- **Error Scenarios** - Test network failures, invalid transactions, and edge cases

### Phase 4: Production Readiness (Future)
- **Performance Optimization** - Benchmark and optimize transaction processing
- **Documentation** - User guides, API documentation, and operational runbooks
- **Monitoring & Metrics** - Implement observability infrastructure
- **Network Parameters** - Dynamic fee calculation based on current network state
- **Mainnet Preparation** - Security audits and stress testing

### Phase 5: Advanced Features (Future)
- **Smart Contracts** - Plutus script support evaluation
- **Native Tokens** - Multi-asset support beyond ADA
- **Staking** - Delegation and rewards integration
- **CCM Support** - Cross-chain messaging (if needed)

## Known Limitations

### Current Limitations & Stubs
- **RPC Client** - Implemented; requires an Ogmios endpoint for integration tests
- **Address System** - Simplified byte arrays instead of proper Cardano bech32 addresses
- **Transaction Building** - Placeholder implementation with hardcoded fees and empty inputs
- **Batch Operations** - Returns empty data instead of actual transaction batches
- **UTXO Management** - No UTXO selection or optimization logic implemented
- **Fee Calculation** - Hardcoded values instead of real Cardano network parameter calculations

### Technical Approach
- **Local Testing** - Use Yaci Dev Kit (straightforward setup, creates blocks)
- **RPC Operations** - Use Ogmios client (already available in existing codebase)
- **No DB sync initially** - Only needed for partner chains operations, not basic Cardano operations
- **Witnessing approach** - Follow Bitcoin pattern (query blocks and filter transactions)
- **Error Handling** - Let AI handle during implementation (not a separate priority task)

### Dependencies & Compatibility
- **Substrate Framework** - Compatible with existing Substrate pallets
- **External Libraries** - Ed25519 signature verification (`ed25519_dalek`), Blake2 hashing
- **Cardano Libraries** - Cardano serialization lib (more mature) vs Pallas (newer, recommended for this project)

## Compilation & Testing Status
- **Clean compilation** - All Cardano integration code compiles without errors
- **Type safety** - All Cardano types implement required traits (Encode, Decode, MaxEncodedLen)
- **Pattern matching** - Exhaustive coverage for all chain-related pattern matches
- **Basic validation** - Core asset mapping and type conversion tests pass
- **Mock framework** - Mock Ada RPC client added with unit tests
- **Integration framework** - P2P, ceremony, and state chain observer integration working
- **Functional testing** - RPC client covered by unit/integration/performance tests (requires Ogmios/Yaci env)

---

This plan serves as a reference for the foundational Cardano integration. It defines phased work, deliverables, and current progress. Phase 2 implementation can proceed with the existing foundation.

**Integration Status:** Foundation Complete - Ready for Core Implementation Phase

**Next Priority:** RPC Client Implementation → Address System → Transaction Construction → Witness Implementation

**Development Approach:** TDD with AI assistance, error-driven development, focus on rewards use case first