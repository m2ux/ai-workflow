# Yaci DevKit Integration Documentation

> **🆕 Updated August 2025 - Working Implementation**  
> This documentation reflects the **actual working implementation** using the official Yaci DevKit v0.10.6 distribution scripts. Previous custom implementations were replaced with the official approach for reliability.

## 🎯 **Overview**

The Cardano integration testing environment uses [Yaci DevKit v0.10.6](https://github.com/bloxbean/yaci-devkit), a comprehensive Cardano development toolkit that provides instant local devnet creation. This setup enables rapid development and testing with full control over the blockchain environment.

## ⚡ **Key Features**

- **⚡ Fast Startup**: 30-60 seconds to full readiness
- **🎛️ Configurable Blocks**: 1-second block times for rapid testing
- **🔒 Local Control**: Fully isolated devnet environment
- **📊 Rich APIs**: Blockfrost-compatible + Ogmios WebSocket
- **🔍 Built-in Explorer**: Web-based blockchain viewer
- **💰 Pre-funded Addresses**: Ready-to-use test accounts
- **🔄 Quick Resets**: Instant devnet state management

## 📁 **Project Structure**

### **Core Yaci DevKit Setup**
```
tests/yaci-devkit/
├── yaci-devkit-0.10.6/             # Official Yaci DevKit v0.10.6 distribution
│   ├── config/                     # Official Yaci DevKit configuration
│   │   ├── env                     # Environment variables
│   │   ├── version                 # Version info  
│   │   └── node.properties         # Node configuration
│   ├── scripts/                    # Official Yaci DevKit scripts
│   │   ├── start.sh                # ✅ Start all services
│   │   ├── stop.sh                 # Stop all services
│   │   └── docker-compose.yml      # Official Docker setup
│   └── bin/                        # Yaci DevKit binaries
├── yaci-devkit-0.10.6.zip         # Original distribution archive
└── README.md                       # Setup documentation
```

### **Build and Test Infrastructure**
```
tests/
├── Makefile.yaci                   # New test orchestration commands
├── integration/
│   └── mod.rs                      # Updated for Yaci Store + Ogmios
└── ada_integration_tests.rs        # Integration test runner
```

### **CI/CD Pipeline**
```
.github/workflows/
└── ada-yaci-integration-tests.yml  # GitHub Actions for Yaci DevKit
```

### **Documentation**
```
docs/development/
└── yaci-devkit-transition-summary.md  # This documentation
```

## 🔧 **Technical Architecture**

### **1. Service Components**
Our Yaci DevKit v0.10.6 setup uses the **official Yaci DevKit scripts** which manage two Docker services:
- **yaci-cli** (`bloxbean/yaci-cli:0.10.6`): All-in-one container with Cardano Node, Ogmios, Yaci Store, Submit API, and Cluster API
- **yaci-viewer** (`bloxbean/yaci-viewer:0.10.6`): Web-based blockchain explorer

### **2. API Endpoints**
| Service | Port | Purpose | Status |
|---------|------|---------|---------|
| **Yaci Store API** | 8080 | Blockfrost-compatible transaction and query API | ✅ Primary |
| **Yaci Viewer** | 5173 | Web-based blockchain explorer | ✅ Active |
| **Ogmios** | 1337 | Chain Sync and query WebSocket/HTTP API | ✅ Active |
| **Submit API** | 8090 | Transaction submission | ✅ Available |
| **Cluster API** | 10000 | Yaci DevKit cluster management | ✅ Available |
| **Cardano Node N2N** | 3001 | Node-to-node communication | ✅ Available |
| **Cardano Node N2C** | 3333 | Node-to-client communication (via socat) | ✅ Available |
| **Kupo** | 1442 | Chain indexer | ⚠️ Disabled by default |

### **3. Setup Method**
- **✅ Uses Official Scripts**: `./yaci-devkit-0.10.6/scripts/start.sh`
- **🐳 Official Docker Setup**: Uses the included `docker-compose.yml` from v0.10.6
- **⚙️ Native Configuration**: Automatically loads from `config/env` and `config/version`
- **🚀 Startup Time**: ~2-5 minutes for full initialization
- **💾 Data Persistence**: Uses Docker volumes for blockchain data

### **4. Test Environment Features**
- `TestEnvironment.yaci_store_endpoint` for Blockfrost-compatible API access
- `wait_for_yaci_store_ready()` function for service health validation
- Pre-configured test fixtures optimized for Yaci DevKit
- Environment-aware timeouts for CI and local development

## 🚀 **Quick Start Guide**

### **1. Start Yaci DevKit**
```bash
cd multichain-engine/tests
make -f Makefile.yaci setup-yaci
```

### **2. Run Integration Tests**
```bash
make -f Makefile.yaci test-yaci
```

### **3. Access Services**
- **🔍 Blockchain Explorer**: http://localhost:5173
- **📚 API Documentation**: http://localhost:8080/swagger-ui.html
- **🔗 Yaci Store API**: http://localhost:8080/api/v1/
- **🔌 Ogmios WebSocket**: ws://localhost:1337

> **⏱️ Note**: Services take 2-5 minutes to fully initialize. Yaci Store API may not be immediately available.

## 🎛️ **Available Commands**

### **Environment Management**
```bash
make -f Makefile.yaci setup-yaci      # Start environment
make -f Makefile.yaci check-yaci      # Check service health
make -f Makefile.yaci logs-yaci       # View logs
make -f Makefile.yaci restart-yaci    # Restart services
make -f Makefile.yaci clean-yaci      # Clean up
```

### **Test Execution**
```bash
make -f Makefile.yaci test-yaci              # All tests
make -f Makefile.yaci test-yaci-basic        # Basic RPC tests
make -f Makefile.yaci test-yaci-chain-sync   # Chain synchronization
make -f Makefile.yaci test-yaci-performance  # Performance benchmarks
```

## 🔧 **CI/CD Configuration**

### **GitHub Actions**
The project includes automated testing via `.github/workflows/ada-yaci-integration-tests.yml` with:
- Automated Yaci DevKit setup and health checks
- Parallel execution of all test suites
- Extended timeouts for CI environments
- Comprehensive logging on failures

### **Environment Variables**

#### **Test Environment Variables**
Used by integration tests:
- `YACI_STORE_ENDPOINT=http://localhost:8080`
- `ADA_WS_ENDPOINT=ws://localhost:1337`
- `ADA_HTTP_ENDPOINT=http://localhost:1337`
- `CARDANO_TESTNET_MAGIC=1`

#### **Yaci DevKit Configuration** (from `config/env`)
Automatically loaded by official scripts:
- `yaci_store_enabled=true` - Enables Yaci Store Blockfrost-compatible API
- `ogmios_enabled=true` - Enables Ogmios WebSocket API  
- `kupo_enabled=false` - Disables Kupo indexer
- `yaci_cli_mode=native` - Uses native CLI mode
- `yaci_store_mode=native` - Uses native Store mode

#### **CI/CD Variables**
- `CI=true` - Enables extended timeouts (300s for Yaci Store startup)

## 🧪 **Testing Capabilities**

### **Integration Test Types**
1. **Basic RPC Tests** (`ada_rpc_integration`)
   - Network info queries via Yaci Store API
   - UTXO queries and address validation
   - Protocol parameter retrieval

2. **Chain Sync Tests** (`chain_sync_integration`)
   - Real-time block synchronization via Ogmios
   - Transaction monitoring and indexing
   - Chain state management

3. **Performance Tests** (`ada_performance`)
   - API response time benchmarks
   - Concurrent request handling
   - Throughput measurements

### **Test Data**
- **Pre-funded Addresses**: Configurable via `topup_addresses` environment variable
- **Network State**: Local devnet environment managed by Yaci DevKit
- **Block Generation**: Configurable block times for development iteration
- **Test Fixtures**: Pre-configured addresses and scenarios for integration tests

## 🔍 **Troubleshooting**

### **Common Issues & Solutions**

#### **Services Won't Start**
```bash
# Check Docker daemon
docker info

# Check port conflicts (all Yaci DevKit ports)
netstat -tulpn | grep -E "(8080|1337|5173|8090|10000|3001|3333|1442)"

# View startup logs using official scripts
cd tests/yaci-devkit/yaci-devkit-0.10.6/scripts
docker-compose logs -f
```

#### **Tests Fail with Connection Errors**
```bash
# Verify service health
make -f Makefile.yaci check-yaci

# Test endpoints manually
curl -f http://localhost:8080/api/v1/network/info  # Should return JSON
curl -f http://localhost:1337/health               # Should return "OK" 
curl -f http://localhost:5173                      # Should return HTML
```

#### **Services Taking Too Long to Start**
```bash
# Check if services are still initializing (normal: 2-5 minutes)
docker ps | grep node1                             # Should show 2 running containers

# Monitor Yaci Store initialization
curl -s http://localhost:8080/api/v1/network/info || echo "Still starting..."

# Check container logs for progress
cd tests/yaci-devkit/yaci-devkit-0.10.6/scripts
docker-compose logs yaci-cli | tail -20
```

#### **Reset Environment**
```bash
# Full restart (recommended)
make -f Makefile.yaci restart-yaci

# Manual cleanup using official scripts
cd tests/yaci-devkit/yaci-devkit-0.10.6/scripts
./stop.sh && ./start.sh
```

## 📈 **Performance Characteristics**

### **Startup Performance**
- **Yaci DevKit**: ~2-5 minutes to full readiness (official scripts handle all initialization)
- **Container Startup**: ~30 seconds for Docker containers to start
- **Service Initialization**: Progressive startup - Yaci Viewer first, then Yaci Store + Ogmios
- **CI Environment**: Extended timeouts (300s for Yaci Store, 120s for Ogmios)

### **Runtime Performance**
- **Basic RPC Tests**: 10-30 seconds execution
- **Chain Sync Tests**: 30-60 seconds execution
- **Performance Tests**: 60-120 seconds with benchmarks
- **Service Response**: Yaci Store API responds within 1-2 seconds after initialization

### **Resource Efficiency**
- **Memory Usage**: ~2GB total
- **Disk Space**: ~500MB for complete devnet
- **CPU Usage**: Minimal during idle, moderate during sync

## 🔮 **Future Enhancements**

### **Potential Improvements**
1. **Multi-network Support**: Easy switching between mainnet/testnet simulation
2. **Custom Genesis**: Pre-configured tokens and smart contracts
3. **Advanced Rollback Testing**: Multi-node cluster scenarios
4. **Performance Tuning**: Even faster block times (200ms) for rapid iteration

### **Integration Opportunities**
1. **MeshJS Integration**: Frontend development support
2. **Cardano Client Lib**: Java backend integration
3. **Custom dApp Testing**: Smart contract deployment and testing

## 🔑 **Key Implementation Insights**

### **✅ What Works**
- **Official Yaci DevKit v0.10.6 scripts**: Use `./start.sh` and `./stop.sh` from the distribution
- **Automatic configuration**: The official scripts load environment variables from `config/env`
- **Extended timeouts**: Allow 2-5 minutes for full service initialization
- **Docker networking**: The official docker-compose.yml handles all port mappings correctly

### **❌ What Doesn't Work**
- **Custom docker-compose.yml**: Environment variable conflicts cause startup failures
- **Manual service orchestration**: The official scripts handle complex initialization sequences
- **Short timeouts**: Yaci Store requires significant time to initialize the blockchain environment

## 🎉 **Summary**

The Yaci DevKit v0.10.6 integration provides a powerful Cardano development environment with:

✅ **Official Scripts Integration** - Uses the tested v0.10.6 distribution directly  
✅ **Complete Service Stack** - Cardano Node + Ogmios + Yaci Store + Explorer  
✅ **Robust Configuration** - Automatically loads official configuration files  
✅ **Rich APIs** - Blockfrost-compatible + Ogmios WebSocket + Submit API  
✅ **Comprehensive tooling** - Built-in explorer and API documentation  
✅ **Automated Setup** - Makefile commands handle all complexity  

This setup enables efficient development and testing of Cardano integrations with a fully functional local blockchain environment and comprehensive API access.

---

**Resources:**
- **[Yaci DevKit Repository](https://github.com/bloxbean/yaci-devkit)**
- **[Setup Documentation](../multichain-engine/tests/yaci-devkit/README.md)**
- **[Official Yaci DevKit Docs](https://devkit.yaci.xyz/)**
