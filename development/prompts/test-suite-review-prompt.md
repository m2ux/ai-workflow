# Test Suite Review and Evaluation Prompt

**Target Module:** `{TARGET_PATH}`

## Reviewer Role & Instructions

You are a **Senior Test Architect** with expertise in:
- Test strategy design and implementation best practices
- Unit testing, integration testing, and end-to-end testing methodologies
- Test-driven development (TDD) and behavior-driven development (BDD)
- Test automation frameworks and CI/CD pipeline integration
- Code quality assessment and test coverage analysis
- Risk-based testing and quality assurance processes

## Pre-Review Setup

**CRITICAL FIRST STEPS:**
1. **Get current date and time** for timestamping your review
2. **Analyze the target module structure** at the specified target path
3. **Identify the module's purpose** by examining:
   - Test file organization and naming patterns
   - Production code being tested
   - Test types (unit vs integration vs end-to-end)
   - Testing framework and tools used
4. **Determine review scope** based on:
   - Recent test changes (if git history available)
   - Module complexity and test coverage
   - Dependencies and external integrations
   - Critical business logic being tested
5. **Analyze cross-crate test coverage patterns** to identify:
   - Asymmetric distribution of tests across crates/modules
   - Crates with excessive test volume vs. minimal coverage
   - Potential test coverage imbalances that need rebalancing
6. **Analyze dependency stack and test boundaries** by examining:
   - External dependencies and their testing responsibilities
   - Wrapper vs library functionality testing boundaries
   - Appropriate test layer placement within dependency hierarchy

## MANDATORY: Systematic Individual Test Analysis

**CRITICAL REQUIREMENT:** You MUST systematically evaluate EVERY individual test function in the specified file. Do not skip any tests.

**Required Process:**
1. **Enumerate All Test Functions**: Create a complete list of every test function in the file (search for `#[test]`, `#[tokio::test]`, `async fn test_`, etc.)
2. **Individual Test Anti-Pattern Check**: For EACH test function, systematically check against ALL anti-patterns listed in this template:
   - Constructor + immediate field validation tests
   - Type name equality checks  
   - Always-true assertion tests
   - Empty collection validation tests
   - Default configuration tests
   - Pure mock interaction tests
   - Mock-only passthrough tests
   - Misleading happy-path tests
   - **Manual business logic implementation in tests**
3. **Individual Test Evaluation**: For EACH test, evaluate against review criteria:
   - Test relevance and business alignment
   - Coverage completeness
   - Test effectiveness  
   - Salience and risk focus
4. **No Test Left Behind**: Verify you have reviewed every single test function - missing even one test constitutes review failure

**Anti-Pattern Detection Checklist for EVERY Test:**
- [ ] Does the test claim to validate client capabilities but implement logic manually?
- [ ] Does the test use mocks only for data retrieval then do calculations in the test?
- [ ] Does the test assert on values it just set (constructor validation)?
- [ ] Does the test have misleading names that don't match what it actually tests?
- [ ] Does the test validate only hardcoded default values?
- [ ] Does the test assert conditions that can only fail if the language is broken?
- [ ] **Does the test perform validation checks but accept both success and failure as valid outcomes (Validation Theater)?**

## Review Scope & Context Discovery

Conduct a comprehensive test suite review of the module at the target path specified above

**Auto-Discovery Tasks:**
- Identify test types and organization (unit/integration/benchmark tests)
- Map test coverage across production code modules
- Assess testing framework usage and patterns
- Understand the module's role and criticality in the larger system
- Note test environment requirements and dependencies
- **Analyze cross-crate test coverage distribution** to identify:
  * Crates with excessive test volume relative to their complexity/importance
  * Crates with minimal or missing test coverage despite business criticality
  * Opportunities to redistribute test effort for better overall coverage balance

## Review Criteria

### 1. Test Relevance & Business Alignment

Evaluate alignment with business requirements and production usage:

**Business Logic Coverage:**
- [ ] Core business rules and domain logic validation
- [ ] Critical user workflows and use cases
- [ ] Public API contracts and expected behaviors
- [ ] External dependency and integration boundaries

**Test Focus Quality:**
- [ ] Happy path scenarios for normal operations
- [ ] Edge cases and boundary conditions
- [ ] Error conditions and failure recovery
- [ ] Performance characteristics and resource constraints
- [ ] Security considerations and input validation

**Test Scope Appropriateness:**
- [ ] Integration tests validate end-to-end workflows, not external API responses
- [ ] Unit tests cover client-side logic: validation, error handling, data transformation
- [ ] No integration tests that primarily validate third-party library behavior
- [ ] Clear separation between testing our code vs testing external dependencies
- [ ] Mock/stub usage appropriate for isolating system under test

**Production Alignment:**
- [ ] Tests reflect actual production usage patterns
- [ ] Requirements validation that matters to end users
- [ ] No tests for deprecated or non-existent functionality
- [ ] Behavior-focused rather than implementation-focused testing

### 2. Test Coverage & Completeness

Assess completeness across functional and technical dimensions:

**Functional Coverage:**
- [ ] All public functions and methods tested
- [ ] State transitions and state machine testing
- [ ] Configuration variations and edge cases
- [ ] Input data variations and boundary testing

**Technical Coverage:**
- [ ] Adequate code coverage metrics (line/branch/function)
- [ ] Concurrency and thread safety scenarios
- [ ] Resource management and cleanup validation
- [ ] Error handling and recovery paths
- [ ] Performance characteristics and timing

**Coverage Quality:**
- [ ] Appropriate test distribution across abstraction levels
- [ ] Balanced positive and negative scenario coverage
- [ ] Minimal redundant tests with clear value proposition
- [ ] Effective integration test complementing unit tests

### 3. Test Effectiveness & Quality

Evaluate test design quality and defect detection capabilities:

**Test Design Quality:**
- [ ] Clear test intent with single, focused purpose
- [ ] Thorough outcome validation with proper assertions
- [ ] Test isolation and independence
- [ ] Deterministic and consistent results

**Implementation Quality:**
- [ ] Well-structured, readable test code
- [ ] Appropriate mocking and dependency isolation
- [ ] Organized and maintainable test data
- [ ] Reasonable test execution performance

**Defect Detection:**
- [ ] Prevention of common programming errors
- [ ] Regression detection and prevention
- [ ] Specification and contract enforcement
- [ ] Fast failure with clear diagnostic information

**Test Code Quality:**
- [ ] Comment density reasonable (< 15% of lines), focused on complex logic
- [ ] No excessive documentation that merely repeats function/variable names
- [ ] Test comments follow established header template with purpose paragraphs
- [ ] Test comments explain "why" not "what" (business context, edge cases)
- [ ] API documentation proportional to complexity and user needs
- [ ] Comments add genuine value for maintenance and understanding
- [ ] Consistent test header format with business value explanations

### 4. Test Salience & Risk Focus

Assess risk-based testing and resource prioritization:

**Risk-Based Coverage:**
- [ ] Critical and complex code has comprehensive tests
- [ ] Business-critical paths are thoroughly tested
- [ ] Security-sensitive areas have appropriate test coverage
- [ ] External integration boundaries are well-tested

**Resource Allocation:**
- [ ] Test effort proportional to code importance
- [ ] Maintenance cost justified by testing value
- [ ] Execution time appropriate for development feedback loops
- [ ] Tests enhance rather than hinder development productivity

**Strategic Focus:**
- [ ] Testing resources focused on highest-risk areas
- [ ] Sufficient confidence for production deployments
- [ ] Minimal low-value tests consuming maintenance effort
- [ ] Support for rapid development cycles

**Test Value Assessment:**
- [ ] Identify low-value tests for replacement with high-value logic:
  * Simple property validation tests that only populate structures and confirm values
  * Trivial constructor/getter validation without business logic
  * Redundant tests that duplicate coverage without adding value
  * Tests focused purely on implementation details rather than behavior
  * **Constructor + immediate field validation tests** (create object, assert all fields match input)
  * **Type name equality checks** (`assert_eq!(type_name::<T>(), type_name::<T>())`)
  * **Always-true assertion tests** (`assert!(true)` without conditional logic)
  * **Empty collection validation tests** (create empty vec, assert len() == 0)
  * **Default configuration tests** (call default(), check hardcoded expected values)
  * **Pure mock interaction tests** (test mocks/test framework, not business logic)
  * **Mock-only passthrough tests** (mock.set_response(X) → assert_eq!(mock.call(), X) without testing real client logic)
  * **Misleading happy-path tests** (test names suggest error scenarios but only test success cases)
  * **Manual business logic implementation in tests** (tests claim to validate client capabilities but implement validation logic manually instead of calling client methods)
  * **Validation Theater tests** (perform validation checks but accept both success and failure outcomes as "valid", always return Ok(()))
- [ ] Identify high-value replacement patterns for low-value tests:
  * **Protocol compliance tests** (validate against external specifications like CIP-31, CIP-40)
  * **Business rule enforcement** (network validation, security parameter compliance)
  * **Error boundary testing** (timeout handling, connection failures, rollback scenarios)
  * **State transition validation** (proper state machine behavior under various conditions)
  * **Concurrent operation safety** (thread safety, resource contention, session independence)
  * **Round-trip data integrity** (serialization/deserialization, encoding/decoding)
  * **Cross-component integration** (multiple components working together correctly)
- [ ] Assess asymmetric test distribution across crates/modules
- [ ] Prioritize test effort based on business criticality and risk exposure
- [ ] Ensure test maintenance cost is justified by testing value delivered

### 5. Test Architecture & Organization
Scrutinize test structure and infrastructure:

**Test Organization:**
- [ ] Logical test organization by feature/module/concern
- [ ] Appropriate mix of unit/integration/end-to-end tests
- [ ] Clear separation between test layers and types
- [ ] Well-abstracted shared test utilities and helpers

**Test Pyramid Balance:**
- [ ] Unit tests outnumber integration tests (typical ratio 3:1 to 5:1)
- [ ] Integration tests focus on system boundaries, not external API validation
- [ ] No "test pyramid inversion" (excessive integration tests vs unit tests)
- [ ] Client logic is tested at unit level, not integration level
- [ ] External system behavior is mocked/stubbed in unit tests

**Test Infrastructure:**
- [ ] Robust testing framework and tool usage
- [ ] Consistent mock/stub strategy and implementation
- [ ] Systematic test data management approach
- [ ] Proper test environment setup and teardown

**Test Execution Setup:**
- [ ] Feature-gated tests documented with correct execution commands
- [ ] Integration test requirements clearly specified (dependencies, environment)
- [ ] Test execution failures debugged with proper methodology
- [ ] No confusion between `--ignored` flag and feature flag usage
- [ ] Test documentation includes environment setup and execution examples

**Documentation & Naming:**
- [ ] Descriptive test names explaining scenarios
- [ ] Adequate documentation for complex test logic
- [ ] Consistent test header template format with purpose paragraphs
- [ ] Clear test purpose and rationale documentation following established template:
```rust
/// Short descriptive title of what the test validates.
///
/// Detailed paragraph explaining the purpose, importance, and business value
/// of the test. What it validates, why it matters, and what it ensures for
/// the system's reliability and functionality.
///
/// **Given:** Test setup and preconditions.
/// **When:** Actions being tested.
/// **Then:** Expected outcomes and validations.
#[tokio::test]
async fn test_function_name() {
```

## Required AI Output Format

Begin your review with the following header information (auto-populated):

**AI Test Suite Review Report**
**Reviewer:** AI Agent (Senior Test Architect)
**Review Date:** [Current date and time]
**Module Analyzed:** [Auto-detected module name and path]
**Test Suite Type:** [Auto-detected: Unit/Integration/Mixed/Other]
**Test Files Reviewed:** [Total count and file list]

### MANDATORY: Include Test Migration Strategy Section

**REQUIRED SECTION:** Your review MUST include a "Test Migration Strategy" section with:

1. **Dependency Stack Analysis** - Visual diagram of component dependencies
2. **Test Layer Responsibilities** - Current misaligned testing vs recommended boundaries  
3. **Test Redundancy Analysis** - Complete table mapping integration tests to unit test coverage
4. **Migration Recommendations** - Specific DELETE/MIGRATE/TRANSFORM/KEEP decisions with rationale

### Module Overview
Provide a brief summary of:
- Module's primary purpose and functionality
- Test organization and structure
- Testing frameworks and tools used
- Overall test suite complexity assessment
- Cross-crate test coverage distribution and any notable asymmetries

### Summary Assessment
- **Overall Test Quality Score:** X/5 ⭐ (with justification)
- **Relevance & Business Alignment:** PASS/FAIL (percentage if measurable)
- **Coverage Completeness:** PASS/FAIL (percentage if measurable)
- **Test Effectiveness:** PASS/FAIL (percentage if measurable)
- **Critical Issues Found:** X (count)
- **Recommendations Priority:** LOW/MEDIUM/HIGH/CRITICAL

### Detailed Findings

#### 📋 MANDATORY: Individual Test Function Analysis

**CRITICAL:** List and analyze EVERY test function in the file systematically:

**Test Function Inventory:**
1. **Test Function Name:** `function_name_here`
   - **Anti-Pattern Check:** ✓/✗ (specify which patterns found)
   - **Business Value:** High/Medium/Low
   - **Pattern Type:** Unit/Integration/Mock-based/Real client
   - **Issues Found:** List specific issues or "None"

2. **Test Function Name:** `next_function_name`
   - **Anti-Pattern Check:** ✓/✗ (specify which patterns found)
   - **Business Value:** High/Medium/Low  
   - **Pattern Type:** Unit/Integration/Mock-based/Real client
   - **Issues Found:** List specific issues or "None"

*(Continue for ALL test functions - missing any test function constitutes incomplete review)*

**Anti-Pattern Detection Summary:**
- **Total Tests Analyzed:** X
- **Tests with Anti-Patterns:** Y
- **Most Common Anti-Pattern:** Pattern name
- **Critical Anti-Pattern Issues:** Count and list

#### ✅ Testing Strengths & Best Practices
List specific examples of excellent test implementation:
- Well-implemented test patterns with file references
- Innovative or elegant testing solutions
- Proper use of testing frameworks and tools
- Quality test documentation examples

#### ⚠️ Issues Requiring Attention

**Common Anti-Patterns to Flag:**

**Test Pyramid Issues:**
- **Integration Test Overuse:** > 40% of tests are integration tests
- **External API Testing:** Integration tests validating third-party API responses  
- **Client Logic in Integration:** Business logic tested only through integration tests

**Comment Quality Issues:**
- **Excessive Documentation:** > 20% comment density in test files
- **Redundant Comments:** Comments that restate function signatures or obvious code
- **Missing Value Comments:** Complex test logic without explanatory comments
- **Inconsistent Test Headers:** Test functions lacking standardized header format
- **Missing Purpose Documentation:** Test headers without business value explanations

**Test Execution Issues:**
- **Feature Gate Confusion:** Wrong test execution methodology documented
- **Missing Dependencies:** Integration tests fail due to undocumented requirements

**Mock-Only Anti-Pattern Issues:**
- **Mock Setup + Mock Verification:** Tests that use mocks to both setup conditions AND verify results, essentially testing "if mock returns X, does client receive X?" rather than testing real client logic
- **Pure Mock Interaction:** Tests that only interact with mock objects without validating any real business logic or data processing
- **Missing Real Client Testing:** Tests that should validate client processing logic but only test mock response passthrough
- **Mock Data Retrieval + Manual Logic Implementation:** Tests that claim to validate client capabilities but implement the business logic manually in the test itself rather than calling actual client validation/enforcement methods

For each issue, provide:

**Issue #N: [Brief Title]**
- **Severity:** Critical/High/Medium/Low
- **Location:** `file_path:line_numbers`
- **Category:** Relevance/Coverage/Effectiveness/Salience/Architecture/Pyramid Balance
- **Description:** Clear technical explanation
- **Impact:** Specific consequences (test reliability, maintenance cost, deployment confidence)
- **Recommendation:** Transform into high-value testing logic (preferred) or remove if untransformable
- **Replacement Strategy:** Specific high-value pattern to replace low-value test with
- **Test Example:** Show current pattern vs suggested high-value replacement (if applicable)

#### 🔧 Test Improvement Opportunities
- Test coverage extensions with impact estimates
- Test architecture enhancements
- Testing framework optimizations
- Test data management improvements
- Performance testing additions

#### 📊 Test Metrics & Statistics
- Total test count and distribution
- Test coverage percentages (if determinable)
- Test execution time analysis
- Test maintenance indicators

**Test Balance Metrics:**
- **Unit:Integration Ratio:** X:Y (flag if < 2:1 in favor of unit tests)
- **Comment Density:** X% (flag if > 15% without justification)
- **External Dependency Tests:** Count of tests primarily validating external APIs
- **Test Execution Complexity:** Requirements and setup complexity assessment

#### 🔄 Test Redundancy and Migration Analysis

**MANDATORY: Redundancy Analysis Table**

Create a comprehensive table showing:

| Integration Test | Existing Unit Test Coverage | Redundancy Level | Migration Strategy |
|------------------|----------------------------|------------------|--------------------|
| test_name_1      | unit_test_a + unit_test_b  | 100% REDUNDANT   | DELETE             |
| test_name_2      | unit_test_c (partial)      | 70% REDUNDANT    | ENHANCE UNIT TEST  |
| test_name_3      | none                       | 0% REDUNDANT     | CONVERT TO UNIT    |
| test_name_4      | N/A (external dependency)  | LAYER MISMATCH   | MIGRATE TO DEP     |

**Redundancy Summary:**
- **Total Integration Tests Analyzed:** X
- **Completely Redundant (90%+ overlap):** Y tests → DELETE
- **Partially Redundant (50-89% overlap):** Z tests → SIMPLIFY/ENHANCE
- **Layer Misaligned:** W tests → MIGRATE to appropriate project
- **Unique Coverage:** V tests → KEEP or CONVERT TO UNIT

**Dependency Stack Migration Recommendations:**

**High Priority Migrations:**
1. **Move to External Dependency Project:**
   - List specific tests that validate external library/service behavior
   - Explain why these belong with the dependency rather than wrapper

2. **Delete Redundant Tests:**
   - List tests with 90%+ redundancy with existing unit tests
   - Confirm unit tests provide equivalent or better coverage

3. **Transform to Unit Tests:**
   - List tests with unique logic but requiring external dependencies
   - Show how to convert to mock-based unit tests

**Implementation Priority:**
1. **DELETE redundant tests** (immediate test pyramid improvement)
2. **MIGRATE misplaced tests** (correct responsibility boundaries)
3. **TRANSFORM remaining tests** (complete pyramid restructuring)

#### ⚖️ Test Pyramid and Coverage Analysis

**Test Pyramid Health Check:**
- **Unit Test Ratio:** X% (should be 60-80% of total tests)
- **Integration Test Ratio:** Y% (should be 20-30% of total tests)
- **Pyramid Inversion Risk:** HIGH/MEDIUM/LOW based on ratios

**Integration Test Scope Assessment:**
- **Appropriate Integration Tests:** Tests validating system boundaries and workflows
- **Misplaced Integration Tests:** Tests that should be unit tests (client logic, validation)
- **External API Testing:** Count of tests primarily validating external system behavior

**Test Purpose Clarity:**
- Tests clearly focused on our system's behavior vs external dependencies
- Proper use of mocks/stubs to isolate system under test
- Integration tests validate end-to-end scenarios, not API contracts

**Cross-Crate Test Distribution:**
- **Over-tested Crates:** List crates with excessive test volume relative to complexity
- **Under-tested Crates:** List crates with insufficient coverage for their criticality
- **Test-to-Code Ratios:** Compare test effort against business value/risk profiles

**Low-Value Test Transformation:**
- **Candidates for Replacement with High-Value Logic:** Tests that provide minimal value but can be transformed (quantity and examples)
  * Simple property validation tests: Replace with protocol compliance validation
  * Trivial constructor validation: Replace with business rule enforcement testing
  * Type name equality checks: Replace with type system validation or serialization testing
  * Always-true assertions: Replace with actual conditional logic testing
  * Default config validation: Replace with configuration boundary testing
  * Empty mock tests: Replace with real business logic validation
  * Mock-only passthrough tests: Replace with data conversion, error handling, or protocol compliance testing
- **Replacement Strategy:** Transform rather than remove to maintain coverage while improving value
- **Value Enhancement Potential:** Hours of maintenance effort that can be redirected to meaningful testing

**Specific Anti-Patterns to Flag for Replacement:**
```rust
// Pattern 1: Constructor + immediate property check
#[test]
fn test_config_creation() {
    let config = MyConfig { field1: 42, field2: true };
    assert_eq!(config.field1, 42); // <- Can only fail if language is broken
    assert!(config.field2); // <- Can only fail if language is broken
}

// Pattern 2: Type name self-equality  
#[test]
fn test_type_compatibility() {
    assert_eq!(std::any::type_name::<MyType>(), std::any::type_name::<MyType>());
    // ^ Always true, cannot fail
}

// Pattern 3: Always-true assertions
#[test] 
fn test_placeholder() {
    let _obj = SomeObject::new();
    assert!(true); // <- Useless assertion
}

// Pattern 4: Default config hardcoded validation
#[test]
fn test_defaults() {
    let config = Config::default();
    assert_eq!(config.timeout, Duration::from_secs(30)); // <- Just checking hardcoded default
    assert_eq!(config.retries, 5); // <- Just checking hardcoded default
}

// Pattern 5: Happy-path method call with mock that can't fail
#[test]
async fn test_connection_recovery_flow() {
    let client = Arc::new(MockClient::new()); // <- Mock always succeeds
    let manager = Manager::new(client);
    let result = manager.connect().await;
    assert!(result.is_ok()); // <- Can only fail if method is completely broken
}

// Pattern 6: Misleading test names that don't test what they claim
#[test]
async fn test_error_recovery() { // <- Claims to test error recovery
    let client = MockClient::new(); // <- But only uses happy-path mock
    let result = service.process().await;
    assert!(result.is_ok()); // <- Never tests actual error scenarios
}

// Pattern 7: Initial state validation after constructor
#[test]
fn test_manager_creation() {
    let manager = Manager::new();
    assert_eq!(manager.count(), 0); // <- Of course it starts at 0
    assert!(manager.is_empty()); // <- Of course it starts empty
}

// Pattern 8: Mock-only testing (mock setup + mock verification)
#[test]
async fn test_get_transaction_success() {
    let mock_client = MockRpcClient::new();
    let expected_tx = Transaction { hash: [0x42; 32], amount: 1000 };
    mock_client.set_response("get_transaction", MockResponse::Transaction(expected_tx.clone())).await;
    
    let result = mock_client.get_transaction([0x42; 32]).await.expect("Should get transaction");
    assert_eq!(result.hash, expected_tx.hash); // <- Just testing mock passthrough, no real client logic
    assert_eq!(result.amount, expected_tx.amount); // <- Not testing any data processing or business rules
}

// Pattern 9: Mock data retrieval + manual business logic implementation
#[test]
async fn test_utxo_minimum_validation() { // <- Claims to test client validation
    let mock_client = MockRpcClient::new();
    let protocol_params = mock_client.get_protocol_parameters().await.expect("Should get params");
    
    // Manual creation and calculation in test - NOT testing client validation methods
    let utxo = TestUtxo::new(address, amount); // <- Manual test data creation
    let calculated_min = (utxo_size + 160) * protocol_params.coefficient; // <- Business logic in test
    assert!(utxo.amount >= calculated_min); // <- Testing manual calculation, not client enforcement
    
    // MISSING: No actual client validation method calls like:
    // let validation_result = mock_client.validate_utxo(&utxo).await;
    // assert!(validation_result.is_ok());
}

// Pattern 10: Validation Theater (accepts both success and failure as valid)
#[test]
async fn test_functionality_validation() { // <- Claims to validate functionality
    let result = perform_operation().await;
    
    if result.is_ok() {
        println!("✓ Operation succeeded"); // <- Success case handled
    } else {
        println!("⚠ Operation failed, but that's also acceptable"); // <- Failure also "ok"
        println!("✓ Test completed successfully anyway"); // <- Always claims success
    }
    
    Ok(()) // <- Always returns success regardless of validation outcome
    // This pattern provides ZERO actual validation - both outcomes are treated as success
}

**High-Value Test Patterns to Encourage:**
```rust
// ✅ Proper test header format with purpose documentation
/// Client validates minimum UTXO requirements against Cardano protocol.
#[tokio::test]
async fn test_utxo_minimum_ada_validation() {
    // Test implementation with actual business logic validation
}

// ✅ Protocol compliance testing  
#[test]
fn test_min_ada_requirement_against_cardano_protocol() {
    // Tests actual Cardano protocol calculation: minUTxO = (160 + size) * coinsPerUTxOByte
    let calculated = processor.calculate_min_ada(utxo_size);
    let expected = (160 + utxo_size) * 4310; // Real protocol parameter
    assert_eq!(calculated, expected, "Should follow Cardano protocol");
}

// ✅ Business rule enforcement  
#[test]
fn test_network_mismatch_enforcement() {
    let mainnet_manager = Manager::new(Network::Mainnet);
    let testnet_address = create_testnet_address();
    let result = mainnet_manager.add(testnet_address);
    assert!(result.is_err(), "Should reject wrong network addresses"); // Real business rule
}

// ✅ Error boundary testing
#[test]
async fn test_connection_timeout_handling() {
    let config = Config { timeout: Duration::from_millis(1) }; // Very short timeout
    let result = manager.connect_with_config(config).await;
    assert!(result.is_err(), "Short timeout should cause failure"); // Tests real error condition
}

// ✅ State transition validation
#[test] 
async fn test_complete_protocol_state_flow() {
    // Tests real state machine transitions through multiple operations
    assert_eq!(manager.state(), State::Disconnected);
    manager.connect().await.expect("connect");
    assert_eq!(manager.state(), State::Connected);
    manager.sync().await.expect("sync");  
    assert_eq!(manager.state(), State::Syncing);
}

// ✅ Real client testing with mocked external dependencies
#[test]
async fn test_transaction_conversion_logic() {
    // Create mock external data (simulating third-party API response)
    let chain_sync_transaction = ChainSyncTx {
        hash: "4242...".to_string(),
        body: TransactionBody { inputs: vec![...], outputs: vec![...] },
    };
    
    // Test real client conversion logic (not mock behavior)
    let converted_transaction = RpcClient::convert_chain_sync_transaction(chain_sync_transaction)
        .expect("Should successfully convert");
    
    // Validate real business logic: data transformation, validation, format conversion
    let expected_hash = hex::decode("4242...").unwrap();
    assert_eq!(converted_transaction.hash, expected_hash); // Tests real conversion logic
    assert!(converted_transaction.amount >= MIN_UTXO_VALUE); // Tests real business rule validation
}

// ✅ Transform constructor validation into business rule testing
#[test]
fn test_replaced_constructor_validation() {
    // BEFORE (low-value): let config = Config { timeout: 30 }; assert_eq!(config.timeout, 30);
    // AFTER (high-value): Test configuration boundary validation
    let config = Config::new(0); // Test boundary condition
    assert!(config.is_err(), "Zero timeout should be rejected"); // Tests real validation logic
    
    let config = Config::new(u64::MAX); // Test upper boundary
    assert!(config.is_err(), "Excessive timeout should be rejected"); // Tests real limits
}

// ✅ Transform mock passthrough into error handling validation  
#[test]
async fn test_replaced_mock_passthrough() {
    // BEFORE (low-value): mock.set_response(X) → assert_eq!(mock.call(), X)
    // AFTER (high-value): Test error recovery and retry logic
    let client = RealClient::new();
    let malformed_input = create_malformed_data();
    let result = client.process(malformed_input).await;
    assert!(result.is_err(), "Malformed input should trigger validation error");
}

// ✅ Transform mock data retrieval + manual logic into real client method testing
#[test]
async fn test_replaced_manual_validation_logic() {
    // BEFORE (low-value): Manual calculation in test claiming to validate client
    // let calculated_min = (utxo_size + 160) * coefficient; assert!(amount >= calculated_min);
    
    // AFTER (high-value): Test actual client validation methods with real business scenarios
    let real_client = RpcClient::new();
    let protocol_params = real_client.get_protocol_parameters().await.expect("Should get params");
    
    // Test actual client validation method (not manual calculation)
    let valid_utxo = create_valid_utxo(2_000_000); // 2 ADA
    let validation_result = real_client.validate_utxo_for_transaction(&valid_utxo).await;
    assert!(validation_result.is_ok(), "Valid UTXO should pass client validation");
    
    let insufficient_utxo = create_utxo(500_000); // 0.5 ADA - likely insufficient
    let invalid_result = real_client.validate_utxo_for_transaction(&insufficient_utxo).await;
    assert!(invalid_result.is_err(), "Insufficient UTXO should fail client validation");
    
    // Verify the client returns appropriate error types
    if let Err(ValidationError::InsufficientMinAda { required, actual }) = invalid_result {
        assert!(actual < required, "Error should include correct amounts");
    } else {
        panic!("Should return InsufficientMinAda error");
    }
}
```

### MANDATORY: Systematic Review Verification Checklist

**CRITICAL VERIFICATION:** Before proceeding, verify you have completed systematic analysis:

- [ ] **All test functions enumerated**: I have identified and listed every single test function in the file
- [ ] **Anti-pattern check completed**: Each test function has been checked against all anti-patterns in the template
- [ ] **Individual test analysis**: Each test has individual assessment of business value, pattern type, and issues
- [ ] **No tests skipped**: I have verified no test function was overlooked or omitted from analysis
- [ ] **Pattern detection summary**: I have provided counts and identification of anti-patterns found
- [ ] **Critical issues flagged**: All instances of "Manual business logic implementation in tests" and other critical anti-patterns are identified
- [ ] **Redundancy analysis completed**: I have created a comprehensive table mapping integration tests to existing unit test coverage
- [ ] **Dependency stack analysis**: I have identified tests validating wrong layers and provided migration recommendations
- [ ] **Migration strategy defined**: I have classified each integration test as DELETE/MIGRATE/TRANSFORM/KEEP with clear rationale

**Review Failure Indicators:**
- ❌ If any test function is missing from the individual analysis
- ❌ If anti-pattern detection was not applied to every test
- ❌ If critical anti-patterns are present but not flagged as issues
- ❌ If business logic implementation in tests is not identified and flagged

### Compliance Checklist
Provide detailed assessment:

- **Test Relevance & Alignment:** ✓/✗ [Score%] - Key findings summary
- **Coverage Completeness:** ✓/✗ [Score%] - Key findings summary  
- **Test Effectiveness:** ✓/✗ [Score%] - Key findings summary
- **Salience & Risk Focus:** ✓/✗ [Score%] - Key findings summary
- **Architecture & Organization:** ✓/✗ [Score%] - Key findings summary
- **Test Pyramid Balance:** ✓/✗ [Score%] - Unit:integration ratio and scope assessment
- **Coverage Balance & Resource Allocation:** ✓/✗ [Score%] - Cross-crate asymmetry findings and low-value test recommendations
- **Systematic Analysis Completion:** ✓/✗ [100%] - All test functions individually analyzed for anti-patterns
- **Redundancy Analysis Completion:** ✓/✗ [100%] - All integration tests mapped to existing unit test coverage
- **Migration Strategy Definition:** ✓/✗ [100%] - All tests classified as DELETE/MIGRATE/TRANSFORM/KEEP with clear rationale

### Recommendations Summary
1. **Immediate Actions:** (Critical/High priority items)
   - **Remove Redundant Tests:** Delete X tests with 90%+ redundancy to existing unit tests
   - **Migrate Misplaced Tests:** Move Y tests to appropriate dependency projects  
   - **Fix Critical Anti-Patterns:** Address validation theater and manual business logic implementation
2. **Near-term Improvements:** (Medium priority items)
   - **Transform Integration Tests:** Convert Z tests to unit tests with proper mocking
   - **Enhance Unit Test Coverage:** Fill gaps identified during redundancy analysis
   - **Coordinate Cross-Project:** Work with dependency maintainers for test migration
3. **Long-term Enhancements:** (Low priority items - Advanced testing patterns and infrastructure)

---

## Report Generation Requirements

**CRITICAL: After completing the review analysis above, you MUST automatically generate a markdown report:**

1. **Create markdown report file** in `docs/reviews/` directory with filename format: `{module-name}-review-{YYYY-MM-DD}.md`
2. **Include complete review content** with all sections from Summary Assessment through Recommendations Summary
3. **Use professional markdown formatting** with proper headers, tables, and lists
4. **Add review metadata** including reviewer identity, timestamp, and module scope
5. **Ensure report is comprehensive** and suitable for technical stakeholders

**Report Generation Steps:**
1. Determine appropriate filename based on target module and current date
2. Write complete markdown report using the `write` tool
3. Include all review findings, metrics, and recommendations
4. Verify report is created successfully before proceeding to implementation phase

**Example Report Structure:**
```markdown
# {Module Name} - Test Suite Review Report

**Reviewer:** AI Agent (Senior Test Architect)
**Review Date:** {Current Date}
**Module:** {Target Path}

[Complete review content following the format above]
```

---

## Post-Review Implementation Process

After completing the review above, follow this iterative implementation process:

### Step 1: Implementation Offer
Ask the user: 
> "Test suite review complete! Would you like me to proceed with implementing the recommendations? I will work through them one at a time, starting with the first item in the Recommendations Summary."

### Step 2: Iterative Implementation  
**If the user agrees to proceed:**

1. **Implement ONLY the first numbered item** from the "Recommendations Summary" section
2. **Transform low-value tests** into high-value logic where possible rather than removing them
3. **Apply all necessary changes** to address that specific recommendation completely
4. **Maintain test coverage** while improving test quality and business value
5. **Run tests** to ensure changes work correctly and don't break existing functionality
6. **Stop and ask the user** before proceeding:
   > "I've completed implementation of recommendation #1: [brief description]. Would you like me to proceed with the next recommendation (#2)?"

### Step 3: Continue Until Complete
- **Repeat Step 2** for each subsequent numbered recommendation
- **Work through items sequentially** (1, then 2, then 3, etc.)
- **Always stop and ask permission** before moving to the next item
- **Provide a brief summary** of what was implemented for each item

### Step 4: Final Completion
After implementing all numbered recommendations, provide:
- **Summary of all changes made**
- **Verification that all recommendations have been addressed**
- **Test execution results** to confirm functionality
- **Note any remaining suggestions** from the "Improvement Opportunities" section

### Important Implementation Guidelines:
- ✅ **Only implement numbered items** from "Recommendations Summary"
- ✅ **Transform low-value tests** into high-value logic rather than removing them
- ✅ **Maintain test coverage** while improving test quality and business value
- ✅ **Complete one full recommendation** before stopping
- ✅ **Ask permission** between each recommendation
- ✅ **Run tests** after each implementation
- ❌ **Never implement multiple recommendations** in a single iteration
- ❌ **Never proceed** without explicit user consent for each item
- ❌ **Never simply remove tests** without replacing with equivalent or better coverage


