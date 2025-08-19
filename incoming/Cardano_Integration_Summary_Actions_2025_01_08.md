# Cardano Integration Summary - Required Actions
**Date:** January 8, 2025  
**Source:** Meeting transcript with Michał Skowron  
**Status:** Action items to be implemented

## Document Structure & Content Changes

### 1. **Document Simplification (HIGH PRIORITY)**
- **Remove verbose technical implementation details** - They make AI work harder and are difficult to maintain
- **Remove file structure listings** - These become outdated and add bloat to context
- **Remove integration points section** - Current descriptions are unclear and don't provide value
- **Split document** - Keep only planning content, move technical details elsewhere
- **Focus on actionable tasks only** - Remove descriptive content that doesn't guide implementation

### 2. **Phase Restructuring**
- **Move witness implementation from Phase 3 to Phase 2** - Required for proper committee rotation
- **Reorganize Phase 2 priority order:**
  1. RPC Client Implementation (Michał working on submit_transaction)
  2. Address System (may be needed for deposit channels)
  3. Transaction Construction (CBOR) 
  4. Witness Implementation (now Phase 2 priority)
  5. Fee Calculation (linked to witnessing)
  6. UTXO Management (linked to witnessing)
- **Merge Phase 3 into testing/validation only**

### 3. **New Tasks to Add**
- **Cardano side logic/scripts** - Vault capability needed for rewards use case
- **Yaci Dev Kit setup documentation** - Reference the local Cardano environment
- **Library selection for witnessing** - Document choice between Cardano serialization lib vs Pallas

## Content Corrections & Clarifications

### 4. **Use Case Focus**
- **Primary use case: Rewards payment in ADA** - Simplest implementation path
- **Secondary use case: Swap functionality** - More complex, requires witnessing
- **Clarify witnessing requirements** - Not needed for basic rewards flow, required for committee rotation

### 5. **Dependency Clarifications**
- **RPC Client: Use Ogmios client** - Already available in existing codebase
- **Local testing: Yaci Dev Kit** - Straightforward setup, creates blocks
- **No DB sync initially** - Only needed for partner chains operations, not basic Cardano operations
- **Witnessing approach: Follow Bitcoin pattern** - Query blocks and filter transactions

### 6. **Technical Approach Updates**
- **UTXO Management: Use same approach as Bitcoin** - Witness each block, filter transactions, record UTXOs on-chain
- **Fee Calculation: Part of witnessing implementation** - Cannot be done independently
- **Error Handling: Let AI handle during implementation** - Not a separate priority task

## Implementation Strategy Changes

### 7. **Task Creation**
- **Convert Phase 2 items into specific GitHub tasks** - With clear titles and descriptions
- **Remove deposit/withdrawal logic as separate task** - May work automatically with other implementations
- **Focus on error-driven development** - Let compilation errors guide next implementation steps

### 8. **Documentation Standards**
- **Remove untested code examples** - They mislead AI and cause compilation issues
- **Keep only verified, tested information** - Avoid speculation about implementation details
- **Maintain concise, factual tone** - Already implemented in recent updates

## Immediate Actions Required

### 9. **Content Removal**
- [ ] Remove "Technical Implementation Details" section entirely
- [ ] Remove "Files Created/Modified" section
- [ ] Remove or significantly simplify "Integration Points" section
- [ ] Remove verbose test coverage descriptions
- [ ] Remove any code snippets or implementation examples

### 10. **Content Addition**
- [ ] Add Cardano side logic/scripts as Phase 2 task
- [ ] Add witness implementation to Phase 2 priority list
- [ ] Add library selection notes (Cardano serialization lib vs Pallas)
- [ ] Add use case prioritization (rewards first, then swaps)
- [ ] Add dependency clarifications (Ogmios, Yaci Dev Kit)

### 11. **Reorganization**
- [ ] Move witness implementation from Phase 3 to Phase 2
- [ ] Reorder Phase 2 tasks by actual priority and dependencies
- [ ] Consolidate Phase 3 to testing/validation only
- [ ] Create clear separation between core functionality and optimizations

## Meeting Decisions Summary

**Agreed Approach:**
- Start with rewards use case (simpler, no witnessing required initially)
- Use Ogmios client for RPC operations
- Follow Bitcoin pattern for UTXO management and witnessing
- Use Yaci Dev Kit for local Cardano testing
- Focus on task creation rather than phase numbering
- Prioritize TDD approach for AI-assisted development

**Rejected/Deferred:**
- Complex planning documentation with technical details
- DB sync integration for basic operations
- Detailed error handling as separate concern

---

**Note:** This action list should be implemented to create a focused, maintainable planning document that effectively guides AI-assisted development without overwhelming context or outdated technical details.