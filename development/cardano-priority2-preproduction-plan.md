# 🎯 Priority 2 (Pre-Production) Action Items Plan
**Cardano RPC Integration - Pre-Production Readiness**

---

## 📋 Overview & Strategic Priorities

Based on the comprehensive code review findings, this document outlines the structured pre-production improvements that will enhance reliability, observability, and maintainability before full production deployment.

### Priority Matrix

| **Priority** | **Category** | **Impact** | **Effort** | **Timeline** | **Dependencies** |
|--------------|--------------|------------|------------|--------------|------------------|
| **P2.1** | ✅ Native Token Handling | High | Medium | **COMPLETED** | None |
| **P2.2** | Configuration Validation | Medium | Low | 1-2 days | None |
| **P2.3** | Performance Optimizations | Medium | High | 4-5 days | P2.2 |
| **P2.4** | Integration Testing | Medium | Medium | 2-3 days | P2.2, P2.3 |
| **P2.5** | Structured Logging & Monitoring | High | Medium | 3-4 days | None |
| **P2.6** | Documentation Enhancement | Low | Low | 1-2 days | All others |

**Total Estimated Timeline: 10-12 days**

---

## 🔧 P2.2: Chain Sync Configuration Validation

### Current State Analysis
- ✅ `ChainSyncConfig` struct exists with basic parameters
- ⚠️ No validation of configuration values (e.g., negative batch sizes, zero max blocks)
- ⚠️ No bounds checking or logical consistency validation
- ⚠️ Poor error messages for invalid configurations

### Implementation Plan

#### 2.1 Add Configuration Validation
```rust
// Proposed validation additions to ChainSyncConfig
impl ChainSyncConfig {
    pub fn validate(&self) -> Result<(), ConfigValidationError> {
        // Validate max_stored_blocks
        if self.max_stored_blocks == 0 {
            return Err(ConfigValidationError::InvalidMaxStoredBlocks(
                "must be greater than 0"
            ));
        }
        
        if self.max_stored_blocks > 10000 {
            return Err(ConfigValidationError::InvalidMaxStoredBlocks(
                "maximum allowed is 10,000 blocks to prevent memory issues"
            ));
        }
        
        // Validate sync_batch_size  
        if self.sync_batch_size == 0 || self.sync_batch_size > 100 {
            return Err(ConfigValidationError::InvalidSyncBatchSize(
                "must be between 1 and 100"
            ));
        }
        
        // Validate initial_sync_points
        if self.initial_sync_points.is_empty() {
            return Err(ConfigValidationError::EmptyInitialSyncPoints);
        }
        
        // Validate logical consistency
        if self.enable_chain_sync && !self.enable_transaction_index {
            log::warn!("Chain Sync enabled without transaction index may impact performance");
        }
        
        Ok(())
    }
    
    pub fn safe_defaults() -> Self {
        Self {
            enable_chain_sync: true,
            max_stored_blocks: 1000,
            sync_batch_size: 10,
            enable_transaction_index: true,
            initial_sync_points: vec![BlockPoint::origin()],
        }
    }
}

#[derive(Debug, thiserror::Error)]
pub enum ConfigValidationError {
    #[error("Invalid max_stored_blocks: {0}")]
    InvalidMaxStoredBlocks(&'static str),
    
    #[error("Invalid sync_batch_size: {0}")]
    InvalidSyncBatchSize(&'static str),
    
    #[error("Initial sync points cannot be empty")]
    EmptyInitialSyncPoints,
}
```

#### 2.2 Integration Points
```rust
// Update constructors to validate configuration
impl AdaRpcClient {
    pub async fn new_with_chain_sync_config(
        endpoint: String, 
        config: ChainSyncConfig
    ) -> Result<Self> {
        // Validate configuration before proceeding
        config.validate()?;
        
        // ... existing implementation
    }
}
```

### Files to Modify
- `toolkit/utils/ogmios-client/src/chain_sync/manager.rs`
- `multichain-engine/src/ada/rpc.rs`
- Add comprehensive tests in `toolkit/utils/ogmios-client/tests/config_validation.rs`

### Acceptance Criteria
- [ ] All configuration parameters have validation bounds
- [ ] Clear error messages for invalid configurations
- [ ] Comprehensive test coverage for edge cases
- [ ] Documentation updated with valid parameter ranges

---

## ⚡ P2.3: Performance Optimizations

### Current State Analysis
- ⚠️ Each RPC call creates a new connection (no connection pooling)
- ⚠️ No request batching for multiple UTXO queries
- ⚠️ Chain Sync processes blocks sequentially without parallelization
- ⚠️ No caching of frequently accessed data (protocol parameters, network info)

### Implementation Strategy

#### 3.1 Connection Pooling
```rust
// New connection pool manager
use deadpool::managed::{Manager, Pool, PoolError};

pub struct OgmiosConnectionManager {
    endpoint: String,
}

impl Manager for OgmiosConnectionManager {
    type Type = OgmiosClients;
    type Error = OgmiosClientError;
    
    async fn create(&self) -> Result<Self::Type, Self::Error> {
        client_for_url(&self.endpoint).await
    }
    
    async fn recycle(&self, conn: &mut Self::Type) -> Result<(), Self::Error> {
        // Health check the connection
        conn.get_tip().await.map(|_| ())
    }
}

pub struct OgmiosConnectionPool {
    pool: Pool<OgmiosConnectionManager>,
}

impl OgmiosConnectionPool {
    pub async fn new(endpoint: String, pool_size: usize) -> Result<Self> {
        let manager = OgmiosConnectionManager { endpoint };
        let config = deadpool::managed::PoolConfig::new(pool_size);
        let pool = Pool::builder(manager).config(config).build()?;
        
        Ok(Self { pool })
    }
    
    pub async fn get_connection(&self) -> Result<deadpool::managed::Object<OgmiosConnectionManager>> {
        self.pool.get().await.map_err(Into::into)
    }
    
    pub async fn execute_with_connection<F, R>(&self, operation: F) -> Result<R>
    where
        F: FnOnce(&OgmiosClients) -> BoxFuture<'_, Result<R>>,
    {
        let conn = self.get_connection().await?;
        operation(&*conn).await
    }
}
```

#### 3.2 Request Batching
```rust
// Batch UTXO queries for efficiency
impl AdaRpcApi for AdaRpcClient {
    async fn query_utxos_batch(&self, 
        address_batches: &[Vec<String>]
    ) -> Result<Vec<Vec<CardanoUtxo>>> {
        // Process batches in parallel with bounded concurrency
        use futures::stream::{FuturesUnordered, StreamExt};
        
        let max_concurrent = 5; // Configurable limit
        let mut futures = FuturesUnordered::new();
        
        for batch in address_batches.chunks(max_concurrent) {
            for addresses in batch {
                futures.push(self.query_utxos(addresses));
            }
            
            // Collect results for this chunk
            let mut results = Vec::new();
            while let Some(result) = futures.next().await {
                results.push(result?);
            }
        }
        
        Ok(results)
    }
}
```

#### 3.3 Response Caching
```rust
// Cache frequently accessed, slowly changing data
use std::time::{Duration, Instant};
use tokio::sync::Mutex;

pub struct CardanoDataCache {
    protocol_parameters: Arc<Mutex<Option<(Instant, CardanoProtocolParameters)>>>,
    network_info: Arc<Mutex<Option<(Instant, CardanoNetworkInfo)>>>,
    cache_ttl: Duration,
}

impl CardanoDataCache {
    pub fn new(cache_ttl: Duration) -> Self {
        Self {
            protocol_parameters: Arc::new(Mutex::new(None)),
            network_info: Arc::new(Mutex::new(None)),
            cache_ttl,
        }
    }
    
    pub async fn get_protocol_parameters<F, Fut>(&self, fetch_fn: F) -> Result<CardanoProtocolParameters>
    where
        F: FnOnce() -> Fut,
        Fut: Future<Output = Result<CardanoProtocolParameters>>,
    {
        let mut cache = self.protocol_parameters.lock().await;
        
        if let Some((timestamp, params)) = cache.as_ref() {
            if timestamp.elapsed() < self.cache_ttl {
                return Ok(params.clone());
            }
        }
        
        // Cache expired or empty, fetch new data
        let new_params = fetch_fn().await?;
        *cache = Some((Instant::now(), new_params.clone()));
        
        Ok(new_params)
    }
}
```

### Files to Modify
- `multichain-engine/src/ada/rpc.rs` - Add caching and batching
- `multichain-engine/src/ada/retry_rpc.rs` - Integrate connection pooling
- Add new file: `multichain-engine/src/ada/connection_pool.rs`
- Add new file: `multichain-engine/src/ada/cache.rs`
- Update `multichain-engine/Cargo.toml` - Add `deadpool` dependency

### Performance Targets
- [ ] 25% improvement in request throughput
- [ ] 50% reduction in connection establishment overhead
- [ ] Cache hit rate >80% for protocol parameters
- [ ] Memory usage remains stable under sustained load

---

## 🧪 P2.4: Integration Testing with Real Ogmios

### Current State Analysis
- ✅ Good unit test coverage for individual components
- ⚠️ No integration tests against real Ogmios server
- ⚠️ No end-to-end testing of Chain Sync scenarios
- ⚠️ No performance benchmarking under realistic conditions

### Implementation Plan

#### 4.1 Test Environment Setup
```rust
// Docker-based test environment
use testcontainers::{clients::Cli, images::generic::GenericImage, Container};

pub struct CardanoTestEnvironment<'a> {
    cardano_node: Container<'a, GenericImage>,
    ogmios_server: Container<'a, GenericImage>,
    test_data: TestDataSet,
}

impl<'a> CardanoTestEnvironment<'a> {
    pub async fn setup(docker: &'a Cli) -> Result<Self> {
        // Start Cardano node in test mode
        let cardano_node = docker.run(
            GenericImage::new("cardano-node", "latest")
                .with_exposed_port(3001)
                .with_env_var("NETWORK", "testnet")
                .with_wait_for(WaitFor::message_on_stdout("listening on"))
        );
        
        // Start Ogmios server connected to the node
        let ogmios_server = docker.run(
            GenericImage::new("cardanosolutions/ogmios", "latest")
                .with_exposed_port(1337)
                .with_env_var("CARDANO_NODE_SOCKET", "/node.socket")
                .with_wait_for(WaitFor::message_on_stdout("Ogmios started"))
        );
        
        let test_data = TestDataSet::load_from_fixtures()?;
        
        Ok(Self {
            cardano_node,
            ogmios_server,
            test_data,
        })
    }
    
    pub async fn wait_for_sync(&self) -> Result<()> {
        // Wait for initial blockchain sync
        let client = AdaRpcClient::new(self.ogmios_endpoint()).await?;
        
        // Poll until we reach expected test height
        let target_height = self.test_data.expected_tip_height;
        loop {
            let tip = client.get_tip().await?;
            if tip.height >= target_height {
                break;
            }
            tokio::time::sleep(Duration::from_secs(1)).await;
        }
        
        Ok(())
    }
    
    pub fn ogmios_endpoint(&self) -> String {
        format!("ws://localhost:{}", self.ogmios_server.get_host_port_ipv4(1337))
    }
    
    pub fn get_test_addresses(&self) -> Vec<String> {
        self.test_data.addresses.clone()
    }
}
```

#### 4.2 Integration Test Scenarios
```rust
// Test file: multichain-engine/tests/integration/ada_rpc_integration.rs

#[tokio::test]
async fn test_basic_rpc_operations() {
    let docker = Cli::default();
    let env = CardanoTestEnvironment::setup(&docker).await.unwrap();
    env.wait_for_sync().await.unwrap();
    
    let client = AdaRpcClient::new(env.ogmios_endpoint()).await.unwrap();
    
    // Test get_tip
    let tip = client.get_tip().await.unwrap();
    assert!(tip.height > 0);
    
    // Test query_utxos
    let addresses = env.get_test_addresses();
    let utxos = client.query_utxos(&addresses).await.unwrap();
    assert!(!utxos.is_empty());
    
    // Test get_protocol_parameters
    let params = client.get_protocol_parameters().await.unwrap();
    assert!(params.min_fee_coefficient > 0);
}

#[tokio::test]
async fn test_chain_sync_flow() {
    let docker = Cli::default();
    let env = CardanoTestEnvironment::setup(&docker).await.unwrap();
    
    let config = ChainSyncConfig::safe_defaults();
    let client = AdaRpcClient::new_with_chain_sync_config(
        env.ogmios_endpoint(),
        config
    ).await.unwrap();
    
    // Start chain sync
    client.start_chain_sync().await.unwrap();
    
    // Wait for some blocks to be processed
    tokio::time::sleep(Duration::from_secs(10)).await;
    
    // Test block retrieval
    let block = client.get_block_by_height(1).await.unwrap();
    assert!(!block.transactions.is_empty());
}

#[tokio::test]
async fn test_native_token_processing() {
    let docker = Cli::default();
    let env = CardanoTestEnvironment::setup(&docker).await.unwrap();
    let client = AdaRpcClient::new(env.ogmios_endpoint()).await.unwrap();
    
    // Query UTXOs that are known to contain native tokens
    let token_addresses = env.test_data.native_token_addresses;
    let utxos = client.query_utxos(&token_addresses).await.unwrap();
    
    // Verify native tokens are properly validated and parsed
    let total_tokens: usize = utxos.iter()
        .map(|utxo| utxo.native_tokens.len())
        .sum();
    
    assert!(total_tokens > 0, "Expected to find native tokens in test UTXOs");
    
    // Verify all tokens have valid structure
    for utxo in utxos {
        for token in utxo.native_tokens {
            assert_eq!(token.policy_id.len(), 28, "Policy ID must be 28 bytes");
            assert!(token.asset_name.len() <= 32, "Asset name must be ≤32 bytes");
            assert!(token.amount > 0, "Token amount must be positive");
        }
    }
}
```

#### 4.3 Performance Benchmarks
```rust
// Test file: multichain-engine/tests/performance/ada_performance_benchmarks.rs

use criterion::{criterion_group, criterion_main, Criterion};

fn benchmark_utxo_queries(c: &mut Criterion) {
    let rt = tokio::runtime::Runtime::new().unwrap();
    
    c.bench_function("query_utxos_100_addresses", |b| {
        b.to_async(&rt).iter(|| async {
            let client = setup_test_client().await;
            let addresses = generate_test_addresses(100);
            client.query_utxos(&addresses).await.unwrap()
        })
    });
    
    c.bench_function("query_utxos_batch_vs_sequential", |b| {
        b.to_async(&rt).iter(|| async {
            let client = setup_test_client().await;
            let address_batches = generate_address_batches(10, 50);
            client.query_utxos_batch(&address_batches).await.unwrap()
        })
    });
}

criterion_group!(benches, benchmark_utxo_queries);
criterion_main!(benches);
```

### New Test Files Structure
```
multichain-engine/tests/
├── integration/
│   ├── ada_rpc_integration.rs
│   ├── chain_sync_integration.rs
│   └── error_handling_integration.rs
├── performance/
│   ├── ada_performance_benchmarks.rs
│   └── chain_sync_performance.rs
└── fixtures/
    ├── test_addresses.json
    ├── sample_utxos.json
    └── cardano_testnet_config.json
```

### Acceptance Criteria
- [ ] Integration tests pass against real Ogmios server
- [ ] Chain Sync flow tested end-to-end
- [ ] Performance benchmarks establish baseline metrics
- [ ] Error scenarios properly tested (network failures, invalid data)
- [ ] Native token processing verified with real data

---

## 📊 P2.5: Structured Logging & Monitoring

### Current State Analysis
- ✅ Basic logging exists with `log::warn!` calls
- ⚠️ No structured logging with contextual metadata
- ⚠️ No metrics collection for operational monitoring
- ⚠️ No health checks or performance indicators
- ⚠️ No correlation IDs for tracking requests across components

### Implementation Strategy

#### 5.1 Structured Logging Migration
```rust
// Replace log::warn! with structured tracing
use tracing::{info, warn, error, instrument};

impl AdaRpcApi for AdaRpcClient {
    #[instrument(
        skip(self),
        fields(
            endpoint = %self.endpoint,
            operation = "query_utxos",
            address_count = addresses.len()
        )
    )]
    async fn query_utxos(&self, addresses: &[String]) -> Result<Vec<CardanoUtxo>> {
        let start = Instant::now();
        
        match self.ogmios_client.query_utxos(addresses).await {
            Ok(ogmios_utxos) => {
                info!(
                    duration_ms = start.elapsed().as_millis(),
                    utxo_count = ogmios_utxos.len(),
                    "UTXO query completed successfully"
                );
                
                // Process results...
            },
            Err(e) => {
                error!(
                    duration_ms = start.elapsed().as_millis(),
                    error = %e,
                    "UTXO query failed"
                );
                return Err(anyhow::anyhow!("Failed to query UTXOs: {}", e));
            }
        }
    }
}

// Enhanced native token logging
for asset in assets {
    match CardanoNativeToken::new(policy_id.to_vec(), asset.name, asset.amount) {
        Ok(token) => native_tokens.push(token),
        Err(e) => {
            warn!(
                operation = "native_token_validation",
                policy_id = %hex::encode(&policy_id),
                asset_name = %hex::encode(&asset.name),
                amount = asset.amount,
                error = %e,
                utxo_id = %format!("{}:{}", hex::encode(&utxo.transaction.id), utxo.index),
                "Invalid native token skipped"
            );
        }
    }
}
```

#### 5.2 Metrics Collection
```rust
// Key metrics to track using prometheus-style metrics
use prometheus::{Counter, Histogram, Gauge, register_counter, register_histogram, register_gauge};

lazy_static! {
    // Request metrics
    static ref ADA_RPC_REQUESTS_TOTAL: Counter = register_counter!(
        "ada_rpc_requests_total",
        "Total number of ADA RPC requests",
        &["operation", "status"]
    ).unwrap();
    
    static ref ADA_RPC_REQUEST_DURATION: Histogram = register_histogram!(
        "ada_rpc_request_duration_seconds",
        "Duration of ADA RPC requests",
        &["operation"]
    ).unwrap();
    
    // Chain Sync metrics
    static ref ADA_CHAIN_SYNC_BLOCKS_PROCESSED: Counter = register_counter!(
        "ada_chain_sync_blocks_processed_total",
        "Total blocks processed by Chain Sync"
    ).unwrap();
    
    static ref ADA_CHAIN_SYNC_LAG: Gauge = register_gauge!(
        "ada_chain_sync_lag_blocks",
        "Number of blocks Chain Sync is behind the tip"
    ).unwrap();
    
    // Native token metrics
    static ref ADA_NATIVE_TOKENS_PROCESSED: Counter = register_counter!(
        "ada_native_tokens_processed_total",
        "Total native tokens processed"
    ).unwrap();
    
    static ref ADA_NATIVE_TOKENS_INVALID: Counter = register_counter!(
        "ada_native_tokens_invalid_total",
        "Total invalid native tokens encountered",
        &["reason"]
    ).unwrap();
    
    // Connection metrics
    static ref ADA_CONNECTION_POOL_ACTIVE: Gauge = register_gauge!(
        "ada_connection_pool_active_connections",
        "Number of active connections in the pool"
    ).unwrap();
}

// Usage in implementation
impl AdaRpcApi for AdaRpcClient {
    async fn query_utxos(&self, addresses: &[String]) -> Result<Vec<CardanoUtxo>> {
        let timer = ADA_RPC_REQUEST_DURATION.with_label_values(&["query_utxos"]).start_timer();
        
        match self.internal_query_utxos(addresses).await {
            Ok(result) => {
                ADA_RPC_REQUESTS_TOTAL.with_label_values(&["query_utxos", "success"]).inc();
                timer.observe_duration();
                Ok(result)
            }
            Err(e) => {
                ADA_RPC_REQUESTS_TOTAL.with_label_values(&["query_utxos", "error"]).inc();
                timer.observe_duration();
                Err(e)
            }
        }
    }
}
```

#### 5.3 Health Checks
```rust
// Health check implementation
use std::time::Duration;

pub struct AdaHealthCheck {
    rpc_client: Arc<AdaRpcClient>,
    chain_sync_manager: Option<Arc<Mutex<ChainSyncManager<OgmiosClients>>>>,
    last_successful_check: Arc<Mutex<Option<Instant>>>,
}

#[derive(Debug, Clone, serde::Serialize)]
pub struct AdaHealthStatus {
    pub status: HealthLevel,
    pub rpc_connectivity: bool,
    pub chain_sync_status: Option<ChainSyncHealth>,
    pub last_request_duration_ms: Option<u64>,
    pub chain_sync_lag_blocks: Option<u64>,
    pub error_details: Option<String>,
}

#[derive(Debug, Clone, serde::Serialize)]
pub enum HealthLevel {
    Healthy,
    Degraded,
    Unhealthy,
}

impl AdaHealthCheck {
    pub async fn check(&self) -> AdaHealthStatus {
        let mut status = AdaHealthStatus {
            status: HealthLevel::Healthy,
            rpc_connectivity: false,
            chain_sync_status: None,
            last_request_duration_ms: None,
            chain_sync_lag_blocks: None,
            error_details: None,
        };
        
        // Test RPC connectivity
        let start = Instant::now();
        match self.rpc_client.get_tip().await {
            Ok(tip) => {
                status.rpc_connectivity = true;
                status.last_request_duration_ms = Some(start.elapsed().as_millis() as u64);
                
                // Check if response time is acceptable
                if start.elapsed() > Duration::from_secs(5) {
                    status.status = HealthLevel::Degraded;
                    status.error_details = Some("RPC response time exceeded 5 seconds".to_string());
                }
            }
            Err(e) => {
                status.status = HealthLevel::Unhealthy;
                status.error_details = Some(format!("RPC connectivity failed: {}", e));
                return status;
            }
        }
        
        // Check Chain Sync status if available
        if let Some(chain_sync_manager) = &self.chain_sync_manager {
            match chain_sync_manager.lock().await.get_sync_state().await {
                ChainSyncState::Syncing | ChainSyncState::UpToDate => {
                    let lag = chain_sync_manager.lock().await.get_chain_lag().await.unwrap_or(0);
                    status.chain_sync_lag_blocks = Some(lag);
                    
                    if lag > 10 {
                        status.status = HealthLevel::Degraded;
                        status.error_details = Some(format!("Chain Sync lagging by {} blocks", lag));
                    }
                }
                other_state => {
                    status.status = HealthLevel::Degraded;
                    status.error_details = Some(format!("Chain Sync in state: {:?}", other_state));
                }
            }
        }
        
        // Update last successful check
        if status.status == HealthLevel::Healthy {
            *self.last_successful_check.lock().await = Some(Instant::now());
        }
        
        status
    }
}
```

### Files to Modify
- `multichain-engine/src/ada/rpc.rs` - Add tracing calls and metrics
- `multichain-engine/src/ada/retry_rpc.rs` - Add retry metrics and structured logging
- Add new file: `multichain-engine/src/ada/health.rs`
- Add new file: `multichain-engine/src/ada/metrics.rs`
- Update `multichain-engine/src/lib.rs` - Register health checks and metrics
- Update `multichain-engine/Cargo.toml` - Add `tracing`, `prometheus` dependencies

### Acceptance Criteria
- [ ] All RPC operations emit structured logs with context
- [ ] Key metrics are collected and exportable via `/metrics` endpoint
- [ ] Health checks provide actionable status information
- [ ] Performance degradation is automatically detected
- [ ] Correlation IDs allow tracing requests across components

---

## 📚 P2.6: Enhanced API Documentation

### Current State Analysis
- ✅ Good inline documentation for public APIs
- ⚠️ Missing usage examples and best practices
- ⚠️ No troubleshooting guides for common issues
- ⚠️ Limited documentation on Chain Sync vs basic mode trade-offs

### Implementation Plan

#### 6.1 Enhanced Module Documentation
```rust
//! # Cardano RPC Client
//! 
//! This module provides a comprehensive interface for interacting with Cardano blockchain
//! through the Ogmios protocol. It supports both basic RPC operations and advanced
//! Chain Sync functionality for real-time blockchain monitoring.
//! 
//! ## Quick Start
//! 
//! ### Basic RPC Operations
//! ```rust
//! use multichain_engine::ada::rpc::{AdaRpcClient, AdaRpcApi};
//! use anyhow::Result;
//! 
//! #[tokio::main]
//! async fn main() -> Result<()> {
//!     // Connect to Ogmios server
//!     let client = AdaRpcClient::new("ws://localhost:1337".to_string()).await?;
//!     
//!     // Get current blockchain tip
//!     let tip = client.get_tip().await?;
//!     println!("Current tip height: {}", tip.height);
//!     
//!     // Query UTXOs for specific addresses
//!     let addresses = vec!["addr1...".to_string()];
//!     let utxos = client.query_utxos(&addresses).await?;
//!     
//!     for utxo in utxos {
//!         println!("UTXO: {}:{} - {} ADA", 
//!                 hex::encode(utxo.tx_hash), 
//!                 utxo.output_index,
//!                 utxo.amount as f64 / 1_000_000.0);
//!         
//!         // Process native tokens
//!         for token in utxo.native_tokens {
//!             println!("  Token: {} - {} units", 
//!                     hex::encode(&token.policy_id), 
//!                     token.amount);
//!         }
//!     }
//!     
//!     Ok(())
//! }
//! ```
//! 
//! ### Chain Sync Mode
//! ```rust
//! use multichain_engine::ada::rpc::{AdaRpcClient, AdaRpcApi};
//! use ogmios_client::chain_sync::ChainSyncConfig;
//! use anyhow::Result;
//! 
//! #[tokio::main]
//! async fn main() -> Result<()> {
//!     // Create client with Chain Sync support
//!     let config = ChainSyncConfig {
//!         enable_chain_sync: true,
//!         max_stored_blocks: 1000,
//!         sync_batch_size: 10,
//!         enable_transaction_index: true,
//!         initial_sync_points: vec![],
//!     };
//!     
//!     let client = AdaRpcClient::new_with_chain_sync_config(
//!         "ws://localhost:1337".to_string(),
//!         config
//!     ).await?;
//!     
//!     // Start synchronization
//!     client.start_chain_sync().await?;
//!     
//!     // Now you can query historical blocks and transactions
//!     let block = client.get_block_by_height(12345).await?;
//!     println!("Block {} has {} transactions", 
//!             block.header.height, 
//!             block.transactions.len());
//!     
//!     // Query specific transactions
//!     let tx_hash = block.transactions[0].hash;
//!     let transaction = client.get_transaction(tx_hash).await?;
//!     
//!     Ok(())
//! }
//! ```
//! 
//! ## Architecture Overview
//! 
//! The Cardano RPC client operates in two modes:
//! 
//! ### Basic Mode (`AdaRpcClient::new`)
//! - **Use Case**: Simple operations like UTXO queries, transaction submission
//! - **Capabilities**: Current blockchain state only
//! - **Resource Usage**: Low memory, minimal connection overhead
//! - **Latency**: Low for single operations
//! 
//! ### Chain Sync Mode (`AdaRpcClient::new_with_chain_sync_config`)
//! - **Use Case**: Historical data access, real-time block monitoring
//! - **Capabilities**: Full blockchain history, block/transaction lookup
//! - **Resource Usage**: Higher memory (configurable), persistent connection
//! - **Latency**: Higher initial sync time, then real-time updates
//! 
//! ## Performance Considerations
//! 
//! ### Connection Management
//! - Basic mode creates connections on-demand
//! - Chain Sync mode maintains persistent WebSocket connection
//! - Consider connection pooling for high-throughput applications
//! 
//! ### Memory Usage
//! - Chain Sync stores blocks in memory (configurable via `max_stored_blocks`)
//! - Native token validation happens in-memory for better performance
//! - UTXO queries scale linearly with address count
//! 
//! ### Error Handling
//! ```rust
//! use multichain_engine::ada::rpc::{AdaRpcClient, AdaRpcApi};
//! use anyhow::{Result, Context};
//! 
//! async fn robust_utxo_query(client: &AdaRpcClient, addresses: Vec<String>) -> Result<()> {
//!     match client.query_utxos(&addresses).await {
//!         Ok(utxos) => {
//!             println!("Found {} UTXOs", utxos.len());
//!         }
//!         Err(e) => {
//!             // Log error with context
//!             eprintln!("UTXO query failed for {} addresses: {}", addresses.len(), e);
//!             
//!             // Implement retry logic if appropriate
//!             if e.to_string().contains("connection") {
//!                 println!("Network error detected, retrying...");
//!                 // Retry with exponential backoff
//!             }
//!             
//!             return Err(e).context("Failed to query UTXOs");
//!         }
//!     }
//!     
//!     Ok(())
//! }
//! ```
//! 
//! ## Security Considerations
//! 
//! - **Network Security**: Always use TLS/WSS in production environments
//! - **Input Validation**: All addresses and hashes are validated before use
//! - **Rate Limiting**: Implement client-side rate limiting for high-volume applications
//! - **Error Information**: Avoid logging sensitive transaction details in error messages
```

#### 6.2 Best Practices Guide

Create `docs/ada-integration-best-practices.md`:

```markdown
# Cardano Integration Best Practices

## When to Use Chain Sync vs Basic Mode

### Use Basic Mode When:
- Only need current UTXO state
- Submitting transactions occasionally
- Building simple wallet applications
- Resource constraints are important

### Use Chain Sync Mode When:
- Need historical block/transaction data
- Building blockchain explorers
- Implementing real-time monitoring
- Need guaranteed transaction ordering

## Performance Optimization

### Connection Management
```rust
// ❌ Bad: Creating new client for each operation
async fn bad_pattern() {
    for address in addresses {
        let client = AdaRpcClient::new("ws://...").await?;
        let utxos = client.query_utxos(&[address]).await?;
        // Client dropped, connection closed
    }
}

// ✅ Good: Reuse client instance
async fn good_pattern() {
    let client = AdaRpcClient::new("ws://...").await?;
    
    for address_batch in addresses.chunks(100) {
        let utxos = client.query_utxos(address_batch).await?;
        // Process batch...
    }
}
```

### Native Token Handling
```rust
// ✅ Validate tokens before processing
for utxo in utxos {
    for token in utxo.native_tokens {
        // Tokens are automatically validated, but check for business logic
        if token.policy_id == expected_policy_id {
            // Process known token type
        } else {
            // Log unknown token for investigation
            log::info!("Unknown token type: {}", token.token_id());
        }
    }
}
```

## Error Handling Patterns

### Retry Strategy
```rust
use tokio::time::{sleep, Duration};

async fn query_with_retry<T>(
    operation: impl Fn() -> Pin<Box<dyn Future<Output = Result<T>>>>,
    max_retries: usize
) -> Result<T> {
    let mut last_error = None;
    
    for attempt in 0..=max_retries {
        match operation().await {
            Ok(result) => return Ok(result),
            Err(e) => {
                last_error = Some(e);
                
                if attempt < max_retries {
                    let delay = Duration::from_millis(200 * 2_u64.pow(attempt as u32));
                    sleep(delay).await;
                }
            }
        }
    }
    
    Err(last_error.unwrap())
}
```

### Monitoring and Alerting
```rust
// Implement circuit breaker pattern
pub struct AdaCircuitBreaker {
    failure_count: AtomicUsize,
    last_failure: AtomicU64,
    threshold: usize,
    timeout: Duration,
}

impl AdaCircuitBreaker {
    pub fn should_allow_request(&self) -> bool {
        let now = SystemTime::now().duration_since(UNIX_EPOCH).unwrap();
        let last_failure = Duration::from_millis(self.last_failure.load(Ordering::Relaxed));
        
        if now.saturating_sub(last_failure) > self.timeout {
            // Reset after timeout
            self.failure_count.store(0, Ordering::Relaxed);
            true
        } else {
            self.failure_count.load(Ordering::Relaxed) < self.threshold
        }
    }
}
```
```

#### 6.3 Troubleshooting Guide

Create `docs/ada-troubleshooting.md`:

```markdown
# Cardano Integration Troubleshooting Guide

## Common Issues and Solutions

### Connection Problems

#### "Failed to connect to Ogmios at ws://localhost:1337"
**Cause**: Ogmios server is not running or not accessible
**Solutions**:
1. Verify Ogmios server is running: `curl http://localhost:1337/health`
2. Check network connectivity and firewall rules
3. Verify the endpoint URL is correct (ws:// vs wss://)
4. Check Ogmios server logs for startup errors

#### "Connection closed unexpectedly"
**Cause**: Network instability or server restart
**Solutions**:
1. Implement automatic reconnection logic
2. Use connection pooling with health checks
3. Monitor network stability
4. Set appropriate timeouts

### Chain Sync Issues

#### "Chain Sync stuck in FindingIntersection state"
**Cause**: Invalid initial sync points or network issues
**Solutions**:
1. Use `BlockPoint::origin()` as fallback sync point
2. Verify initial sync points are valid block hashes
3. Check network connectivity during sync initialization

#### "Chain Sync memory usage growing unbounded"
**Cause**: `max_stored_blocks` set too high or not enforced
**Solutions**:
1. Reduce `max_stored_blocks` to reasonable value (≤1000)
2. Enable block pruning in Chain Sync configuration
3. Monitor memory usage with metrics

### Performance Issues

#### "UTXO queries taking too long"
**Cause**: Querying too many addresses at once
**Solutions**:
1. Batch addresses in groups of 50-100
2. Implement parallel queries with bounded concurrency
3. Cache frequently accessed UTXOs
4. Use connection pooling

#### "High CPU usage during native token processing"
**Cause**: Processing many tokens without validation caching
**Solutions**:
1. Implement token validation caching
2. Process tokens in batches
3. Skip obviously invalid tokens early

### Data Validation Errors

#### "Invalid native token: Policy ID length expected 28 bytes, got X"
**Cause**: Malformed native token data from Ogmios
**Solutions**:
1. This is logged as warning and token is skipped automatically
2. Investigate if Ogmios data source is corrupted
3. Report to Ogmios maintainers if systematic issue

#### "Hash conversion failed: invalid hex string"
**Cause**: Corrupted hash data from Ogmios
**Solutions**:
1. Verify Ogmios server is synchronized properly
2. Check for data corruption in Cardano node
3. Restart Chain Sync from known good point

## Debugging Tools

### Enable Debug Logging
```rust
// Set log level for detailed debugging
env_logger::Builder::from_env(env_logger::Env::default().default_filter_or("debug")).init();

// Or use tracing for structured logs
tracing_subscriber::fmt()
    .with_max_level(tracing::Level::DEBUG)
    .with_target(false)
    .init();
```

### Monitor Key Metrics
```bash
# Check RPC request metrics
curl http://localhost:9090/metrics | grep ada_rpc_requests_total

# Monitor Chain Sync lag
curl http://localhost:9090/metrics | grep ada_chain_sync_lag_blocks

# Check connection pool status
curl http://localhost:9090/metrics | grep ada_connection_pool
```

### Health Check Endpoint
```rust
// GET /health/ada
{
    "status": "healthy",
    "rpc_connectivity": true,
    "chain_sync_status": {
        "state": "Syncing",
        "blocks_processed": 12345,
        "lag_blocks": 2
    },
    "last_request_duration_ms": 150
}
```

## Performance Benchmarking

### Load Testing
```bash
# Test UTXO query performance
cargo test --release performance::ada_performance_benchmarks

# Chain Sync stress test
cargo test --release integration::chain_sync_load_test
```

### Memory Profiling
```bash
# Run with memory profiling
RUST_LOG=debug cargo test --release test_chain_sync_memory_usage

# Monitor with system tools
top -p $(pgrep multichain-engine)
```

## Getting Help

1. **Check logs**: Enable debug logging to see detailed operation traces
2. **Review metrics**: Monitor key performance and error metrics
3. **Test isolation**: Use integration tests to isolate issues
4. **Community support**: Reach out to Ogmios community for server-side issues
```

### Files to Create/Modify
- Enhanced documentation in `multichain-engine/src/ada/rpc.rs`
- `docs/ada-integration-best-practices.md`
- `docs/ada-troubleshooting.md`
- Update `README.md` with Cardano integration section
- Add examples to `examples/ada_usage.rs`

### Acceptance Criteria
- [ ] Complete usage examples for both basic and Chain Sync modes
- [ ] Best practices guide covers performance and security
- [ ] Troubleshooting guide addresses common issues
- [ ] All public APIs have comprehensive documentation
- [ ] Examples are tested and verified to work

---

## 🚀 Implementation Timeline & Dependencies

### Phase 1: Foundation (Days 1-2)
- ✅ **P2.1 Native Token Handling** - **COMPLETED**
- **P2.2 Configuration Validation**
  - Add validation logic to `ChainSyncConfig`
  - Write comprehensive tests
  - Update constructor error handling

### Phase 2: Core Improvements (Days 3-6)
- **P2.5 Structured Logging & Monitoring** (Days 3-5)
  - Migrate to structured logging with tracing
  - Implement metrics collection
  - Add health check endpoints
- **P2.3 Performance Optimizations** (Start on Day 4)
  - Design connection pooling architecture
  - Implement request batching

### Phase 3: Testing & Optimization (Days 7-9)
- **P2.3 Performance Optimizations** (Complete)
  - Finish connection pooling implementation
  - Add response caching
  - Performance testing and tuning
- **P2.4 Integration Testing** (Days 8-9)
  - Set up Docker test environment
  - Write integration test scenarios
  - Run performance benchmarks

### Phase 4: Documentation & Polish (Days 10-11)
- **P2.6 Documentation Enhancement**
  - Write comprehensive usage examples
  - Create best practices guide
  - Develop troubleshooting documentation
- **Final validation and cleanup**

## 📋 Success Criteria

### Overall Goals
- **Reliability**: System handles network failures gracefully with proper retry logic
- **Observability**: Comprehensive logging and metrics enable effective monitoring
- **Performance**: 25% improvement in throughput with stable memory usage
- **Maintainability**: Clear documentation and testing enable confident future development

### Specific Metrics
- [ ] **P2.2**: Configuration validation prevents >95% of invalid setups
- [ ] **P2.3**: Connection pooling reduces establishment overhead by 50%
- [ ] **P2.4**: Integration tests achieve >90% code coverage for RPC operations
- [ ] **P2.5**: Health checks accurately reflect system status with <1% false positives
- [ ] **P2.6**: Documentation includes working examples for all major use cases

### Production Readiness Checklist
- [ ] All critical paths have retry logic with exponential backoff
- [ ] Comprehensive monitoring covers performance and error scenarios  
- [ ] Integration tests validate behavior against real Cardano network
- [ ] Documentation enables new developers to integrate successfully
- [ ] Performance meets requirements under expected production load
- [ ] Health checks enable automated deployment and scaling decisions

This plan provides a systematic approach to production readiness while building upon the completed native token validation improvements. Each phase delivers incremental value and can be deployed independently, allowing for continuous improvement of the Cardano integration.

