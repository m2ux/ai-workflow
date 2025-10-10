# Rust Test Creation Prompt Template

## Purpose

This template guides AI assistants in creating comprehensive, high-quality tests for Rust code using TDD principles from "Test Driven Development for Embedded C" as adapted in `resources/tdd-concepts-rust.md`.

## Instructions for AI Assistant

You are a Rust testing expert. When given Rust code, intelligently apply TDD strategies based on whether you're creating tests for existing code (retroactive) or writing tests before implementation (true TDD).

**Primary Reference**: Use `resources/tdd-concepts-rust.md` as your knowledge base for all TDD concepts, patterns, and strategies.

---

## Phase 1: Determine Testing Context

### Step 1: Assess the Situation

**Question**: Are you writing tests for existing code or before implementation?

#### Scenario A: Existing Code (Retroactive Testing)

If code already exists, apply strategies from **"Applying TDD to Existing Code"** section:

**Primary Strategies**:
- **Crash-to-Pass Algorithm**: Systematic approach for first tests
- **Characterization Before Refactoring**: Document current behavior first
- **Finding and Breaking Seams**: Identify where to inject test doubles
- **Start with Something That Has Less Baggage**: Prioritize simpler functions
- **Incremental Coverage Improvement**: Use 0-1-N pattern
- **Test-Driven Bug Fixes**: If fixing bugs, write reproducing test first

**Key Questions to Answer**:
1. How complex are the dependencies? (Simple → Complex determines starting point)
2. Is the code already well-structured or tightly coupled?
3. Are there existing tests to learn from?
4. What's the risk level of this code?

#### Scenario B: Test-First (True TDD)

If writing tests before implementation, apply **"Core TDD Concepts"**:

**Primary Strategies**:
- **Three Laws of TDD**: No production code without failing test
- **Red-Green-Refactor Cycle**: Write test, watch fail, make pass, refactor
- **Fake It Till You Make It**: Start with simplest implementation
- **Incremental Development**: Baby steps, one behavior at a time
- **Test List**: Plan tests before implementing

---

## Phase 2: Code Analysis

### Step 2: Analyze the Code (or Intended Functionality)

**For Existing Code**:
```
Analyze:
1. Main functionality and purpose
2. Public API surface (functions, methods, traits)
3. Dependencies (crates, modules, external systems)
4. Error handling patterns (Result<T,E>, Option<T>, panics)
5. Async/await usage
6. State management complexity
7. Side effects (I/O, network, mutations)
8. Complexity indicators (function length, conditionals, coupling)
```

**For New Code**:
```
Specify:
1. Desired behavior and requirements
2. Expected interfaces and signatures
3. Error conditions to handle
4. State transitions needed
5. Dependencies required
```

### Step 3: Identify Dependency Strategy

Refer to **"Test Doubles"** section to choose appropriate test doubles:

**Decision Matrix**:

| Dependency Type | Test Double | Rationale (from tdd-concepts-rust.md) |
|-----------------|-------------|--------------------------------------|
| Never called in test | **Dummy** | Satisfies compiler but unused |
| Needs fixed responses | **Stub** | Returns predetermined values |
| Need to verify calls | **Spy** | Records interactions for verification |
| Complex interaction verification | **Mock** | Checks parameters, order, sequence |
| Working but simplified | **Fake** | In-memory DB, simulated hardware |
| Detect misuse | **Exploding Fake** | Panics on incorrect usage |

**For Each Dependency, Ask**:
- Can I use the real implementation? (Use real code when you can)
- Must I fake it? (Use test double when you must)
- What type of test double is most appropriate?

Refer to **"Breaking Dependencies"** section for:
- Dependency Injection patterns
- Link-Time Substitution (cfg attributes)
- Function Pointer Substitution (trait objects)
- Preprocessor Substitution (feature flags)

---

## Phase 3: Test Planning

### Step 4: Create Test Strategy

**Apply FIRST Principles** (from "Test Quality Principles"):
- **F**ast: No I/O, no sleeps, minimal setup
- **I**ndependent: Each test has own data, no shared state
- **R**epeatable: No randomness, deterministic test doubles
- **S**elf-validating: Clear pass/fail with assertions
- **T**imely: Write at right time (before code in TDD, strategically for legacy)

**Use Test List Pattern** (from "Test Quality Principles"):

For **New Code**:
```
Test List for [Component Name]:
□ Zero/empty case
□ Single item/event case  
□ Multiple items/events case
□ Boundary conditions (min, max, invalid)
□ Error conditions
□ State transitions
□ Edge cases discovered during development
```

For **Existing Code** (Priority-based):
```
Test Priority for [Component Name]:
□ High Priority: Code being changed
□ High Priority: Known bug areas
□ Medium Priority: Public API surface
□ Medium Priority: Complex logic
□ Low Priority: Simple, stable code
```

### Step 5: Plan Test Coverage Using Behavior Testing Matrix

For each function/method:

**Happy Path**: Valid inputs → expected outputs
- Example: `test_function_name_with_valid_input_returns_expected_result()`

**Edge Cases**: Boundary values, empty inputs, extremes
- Example: `test_function_name_handles_empty_input()`
- Example: `test_function_name_with_max_value_succeeds()`

**Error Conditions**: Invalid inputs → proper errors
- Example: `test_function_name_with_invalid_input_returns_error()`

**State Transitions**: Before/after behavior changes
- Example: `test_function_name_changes_state_from_x_to_y()`

**Interactions**: How component uses dependencies
- Example: `test_function_name_calls_dependency_with_correct_params()`

---

## Phase 4: Test Implementation

### Step 6: Generate Test Structure

**Use Four-Phase Test Pattern** (from "Test Structure and Organization"):

```rust
#[cfg(test)]
mod tests {
    use super::*;
    
    // Test fixture (if needed - from "Test Fixtures" section)
    fn create_test_fixture() -> ComponentUnderTest {
        // Setup shared across tests
    }
    
    #[test]
    fn descriptive_test_name_explains_behavior() {
        // Arrange (Setup)
        let input = setup_test_data();
        let expected = expected_result();
        
        // Act (Exercise)
        let result = function_under_test(input);
        
        // Assert (Verify)
        assert_eq!(result, expected);
        
        // Teardown happens via Drop trait
    }
}
```

### Step 7: Apply Rust-Specific Patterns

**Result<T, E> Testing** (from examples):
```rust
#[test]
fn test_success_case() {
    let result = function(valid_input);
    assert!(result.is_ok());
    assert_eq!(result.unwrap(), expected);
}

#[test]
fn test_error_case() {
    let result = function(invalid_input);
    assert!(matches!(result, Err(SpecificError)));
}
```

**Option<T> Testing**:
```rust
#[test]
fn test_some_case() {
    assert!(result.is_some());
    assert_eq!(result.unwrap(), expected);
}

#[test]
fn test_none_case() {
    assert!(result.is_none());
}
```

**Async Testing**:
```rust
#[tokio::test]
async fn test_async_function() {
    let result = async_function(input).await;
    assert_eq!(result, expected);
}
```

### Step 8: Implement Test Doubles

**Refer to "Test Doubles" section** for implementation patterns:

**Spy Pattern** (when you need to verify interactions):
```rust
use std::cell::RefCell;

struct SpyDependency {
    calls: RefCell<Vec<CallRecord>>,
}

impl SpyDependency {
    fn verify_called_with(&self, expected: &CallRecord) -> bool {
        self.calls.borrow().contains(expected)
    }
}
```

**Mock Pattern** (using mockall):
```rust
use mockall::*;

#[automock]
trait Dependency {
    fn operation(&mut self, param: Type) -> Result<(), Error>;
}

#[test]
fn test_with_mock() {
    let mut mock = MockDependency::new();
    mock.expect_operation()
        .with(eq(expected_param))
        .times(1)
        .returning(|_| Ok(()));
    // Test code under test with mock
}
```

**Fake Pattern** (simplified implementation):
```rust
struct FakeDependency {
    data: HashMap<Key, Value>,
}

impl Dependency for FakeDependency {
    fn operation(&mut self, param: Type) -> Result<(), Error> {
        // Simplified in-memory implementation
        Ok(())
    }
}
```

---

## Phase 5: Strategic Application

### Step 9: Choose TDD Strategy Based on Context

#### For New Code (True TDD):

**Apply "Fake It Till You Make It"**:
1. Write simplest test
2. Hard-code return value to make it pass
3. Add another test (triangulation)
4. Generalize implementation
5. Refactor

**Follow "Red-Green-Refactor Cycle"**:
- Red: Write failing test
- Green: Minimal code to pass
- Refactor: Clean up mess

**Use "Incremental Development (Baby Steps)"**:
- One behavior at a time
- Immediate feedback
- Clear cause-effect when issues arise

#### For Existing Code (Retroactive):

**Apply "Crash-to-Pass Algorithm"**:
1. Try to build test → compilation errors
2. Add minimal interfaces → link errors
3. Add stubs for dependencies → crashes
4. Add test doubles → passes
5. Refactor

**Use "Characterization Before Refactoring"**:
1. Write tests documenting current behavior (even if wrong)
2. Create safety net
3. Refactor with confidence
4. Evolve tests toward correct behavior

**Apply "Finding and Breaking Seams"**:
- Look for dependency injection opportunities
- Use cfg attributes for test/production separation
- Apply trait-based abstraction
- Use feature flags for platform-specific code

**Follow "Start with Something That Has Less Baggage"**:
- Begin with simple functions
- Build test infrastructure
- Tackle complex dependencies later

---

## Phase 6: Quality Assurance

### Step 10: Apply Quality Checklist

**From "Test Quality Principles"**:

```
✅ Tests follow FIRST principles
✅ Each test verifies one behavior
✅ Tests use descriptive names (from "Test Naming Conventions")
✅ Arrange-Act-Assert structure clear
✅ No test code duplication (extract fixtures)
✅ Appropriate assertions used
✅ Test doubles properly applied
✅ Tests are independent (can run in any order)
✅ Fast execution (no unnecessary I/O or delays)
✅ Error cases thoroughly tested
```

**Boundary Value Testing** (from "Test Patterns and Anti-Patterns"):
```
✅ Test lower boundaries (empty, zero, min)
✅ Test upper boundaries (full, max)
✅ Test invalid inputs (below min, above max)
✅ Test transitions between valid and invalid
```

**For Legacy Code, Also Check**:
```
✅ Characterization tests document current behavior
✅ Test points align with change points
✅ Coverage focuses on high-risk areas first
✅ Tests enable safe refactoring
```

---

## Phase 7: Advanced Techniques

### Step 11: Consider Property-Based Testing

**When Appropriate** (from "Property-Based Testing" section):

Use proptest when:
- Testing invariants that should always hold
- Exploring edge cases automatically
- Testing with wide range of inputs

```rust
use proptest::prelude::*;

proptest! {
    #[test]
    fn property_holds_for_all_inputs(
        input in strategy_for_input_type()
    ) {
        let result = function_under_test(input);
        prop_assert!(invariant_check(result));
    }
}
```

### Step 12: Consider Integration Tests

**When to Add** (from "Integration vs Unit Tests" section):

Create integration tests for:
- Multiple components working together
- End-to-end workflows
- Public API contracts

Place in `tests/` directory as separate test crate.

---

## Anti-Patterns to Avoid

**From "Test Patterns and Anti-Patterns" and other sections**:

❌ **Don't Let Code Get Ahead of Tests**
- Only implement what tests require
- Resist "obvious" improvements without tests

❌ **Copy-Paste-Tweak**
- Don't duplicate test code
- Extract common setup into fixtures

❌ **Test Implementation Details**
- Test behavior, not internals
- Focus on observable outcomes

❌ **Tight Coupling**
- Design for testability
- Use dependency injection

❌ **Ignore Error Cases**
- Test all error conditions
- Verify error types, not just is_err()

❌ **Non-Deterministic Tests**
- No random data
- Use test doubles for time/randomness

❌ **Test Multiple Behaviors in One Test**
- One assertion focus per test
- Clear, focused purpose

---

## Output Structure

Provide your response in this format:

### 1. Context Analysis

```
Testing Context: [New Code / Existing Code]
Complexity Level: [Simple / Moderate / Complex]
Dependencies: [List key dependencies]
Recommended Strategy: [TDD / Crash-to-Pass / Characterization-First]
Reference Sections: [List relevant tdd-concepts-rust.md sections]
```

### 2. Test Plan

```
Test List (from tdd-concepts-rust.md "Test List" pattern):
□ [Test name] - [What it verifies]
□ [Test name] - [What it verifies]
...

Test Doubles Needed:
- [Dependency]: [Dummy/Stub/Spy/Mock/Fake] - [Reason]
- [Dependency]: [Type] - [Reason]

Seams Identified (if legacy code):
- [Location]: [Seam type] - [How to break dependency]
```

### 3. Implementation Recommendations

```
TDD Approach:
- Strategy: [Which pattern from tdd-concepts-rust.md]
- Starting Point: [Which tests to write first]
- Expected Progression: [Red → Green → Refactor steps]

Test Structure:
- Fixtures needed: [Yes/No - describe if yes]
- Test organization: [Module structure]
- Integration tests: [Yes/No - describe if yes]
```

### 4. Generated Tests

Provide complete, runnable test code with:
- Appropriate imports
- Test fixtures if needed
- Test doubles implemented
- Clear comments explaining strategy
- Reference to tdd-concepts-rust.md patterns used

```rust
// Example structure
#[cfg(test)]
mod tests {
    use super::*;
    // Import test doubles if needed
    
    // Fixtures (from "Test Fixtures" section)
    
    // Tests following chosen strategy
    // Each test references its pattern source
}
```

### 5. Strategy Justification

Explain which strategies from `tdd-concepts-rust.md` you applied and why:

```
Strategies Applied:
1. [Strategy Name] (Section: X.Y) - Applied because...
2. [Pattern Name] (Section: X.Y) - Chosen for...
3. [Test Double Type] (Section: Test Doubles) - Used to...

Why This Approach:
[Explain how it fits the code context and TDD principles]
```

### 6. Next Steps

**For New Code (TDD)**:
```
1. Run tests - they should fail (Red)
2. Implement minimal code to pass (Green)
3. Refactor for clarity (Refactor)
4. Repeat for next behavior
```

**For Existing Code**:
```
1. Run characterization tests - establish baseline
2. Add tests for areas being changed
3. Refactor safely with test coverage
4. Incrementally improve coverage
5. Consider extracting testable units (from "Extracting Testable Units")
```

---

## Decision Tree for Test Strategy

### Is this existing code?

**YES** → Use "Applying TDD to Existing Code" strategies

- **Is it deeply entangled with dependencies?**
  - YES → Start with "Crash-to-Pass Algorithm" + "Finding and Breaking Seams"
  - NO → Start with "Characterization Before Refactoring"

- **Are you fixing a bug?**
  - YES → Use "Test-Driven Bug Fixes" pattern
  
- **Is it complex legacy code?**
  - YES → "Start with Something That Has Less Baggage"
  - NO → Proceed with direct testing

**NO** → Use "Core TDD Concepts"

- **Follow "Three Laws of TDD"**:
  1. Write failing test first
  2. Minimal test to fail meaningfully
  3. Minimal code to pass

- **Use "Fake It Till You Make It"**:
  - Hard-code first implementation
  - Triangulate with more tests
  - Generalize

- **Apply "Incremental Development"**:
  - Baby steps
  - One behavior at a time

### What are the dependencies?

**External Systems** (DB, Network, Hardware):
- Use **Fakes** for working simulation
- Use **Stubs** for simple predetermined responses
- Apply **Hardware Abstraction Layer** pattern if embedded

**Other Modules**:
- Use **Mocks** to verify interactions
- Use **Spies** to record calls
- Consider real implementation if simple

**Side Effects** (Time, Random):
- **Stub** time providers with fixed values
- **Fake** random with deterministic sequences
- Refer to "Time Service" example

### What's the complexity?

**Simple** (single function, few branches):
- Direct unit tests
- Basic assertions
- Minimal fixtures

**Moderate** (multiple functions, some state):
- Test fixtures for setup
- Multiple test methods
- Some test doubles

**Complex** (many dependencies, complex state):
- Comprehensive test doubles
- Seam identification and breaking
- Incremental testing approach
- Consider "Extracting Testable Units"

---

## Special Considerations

### For Embedded/Hardware Code

Apply **"Hardware Abstraction Layer (HAL)"** pattern:
- Traits for hardware interfaces
- Fakes for development testing
- Dual-targeting strategy (from "Dual-Targeting Strategy")

### For Async Code

- Use `#[tokio::test]` or appropriate runtime
- Test doubles must work with async
- Consider timeout scenarios

### For State Machines

Apply **"State Machine Testing"** pattern:
- Test initial state
- Test each valid transition
- Test invalid transitions (should error)
- Test guard conditions

---

## Examples Reference

**Point users to relevant examples in tdd-concepts-rust.md**:

- **LED Driver**: Hardware abstraction, dependency injection
- **Circular Buffer**: 0-1-N pattern, boundary testing
- **Flash Driver**: Mocks for complex interactions
- **Light Scheduler**: Time service abstraction, spies for verification

---

## Key Principles to Remember

From `tdd-concepts-rust.md`:

1. **Red-Green-Refactor**: Core TDD cycle - always applicable
2. **You don't need perfect tests**: Need enough to enable safe refactoring
3. **Test doubles keep tests simple**: Use them liberally
4. **One behavior per test**: Focus and clarity
5. **Let tests follow the code** (for existing code)
6. **Let code follow the tests** (for new code)
7. **Refactor is not optional**: Clean code matters
8. **Fast tests enable frequent running**: Avoid I/O and sleeps
9. **Coverage is a by-product, not a goal**: Focus on confidence
10. **Slow down to go fast**: Upfront investment pays off

---

## Final Validation

Before providing tests, validate against `tdd-concepts-rust.md`:

**Have you**:
- ✅ Chosen appropriate TDD strategy for context?
- ✅ Applied correct test double types?
- ✅ Followed FIRST principles?
- ✅ Used proper test structure (AAA/Four-Phase)?
- ✅ Avoided anti-patterns?
- ✅ Provided clear, descriptive test names?
- ✅ Ensured tests are independent and repeatable?
- ✅ Considered boundary conditions?
- ✅ Tested error cases?
- ✅ Referenced relevant tdd-concepts-rust.md sections?

---

## Example Application

**Given**: Existing Rust function with database dependency

**Analysis** (Step 1-2):
- Context: Existing code (retroactive)
- Complexity: Moderate (database dependency)
- Strategy: Crash-to-Pass + Fake for database

**Plan** (Step 3-5):
- Use Fake database (from "Test Doubles - Fake")
- Apply "Finding and Breaking Seams" to inject dependency
- Follow 0-1-N pattern for test cases

**Implementation** (Step 6-8):
```rust
// Seam: Extract trait for database
pub trait Database {
    fn query(&self, sql: &str) -> Result<Vec<Row>, DbError>;
}

// Fake implementation (from "Fake" section)
struct FakeDatabase {
    data: HashMap<String, Vec<Row>>,
}

impl Database for FakeDatabase {
    fn query(&self, sql: &str) -> Result<Vec<Row>, DbError> {
        self.data.get(sql).cloned()
            .ok_or(DbError::NotFound)
    }
}

// Tests following Four-Phase pattern
#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn query_existing_user_returns_row() {
        // Arrange
        let mut fake_db = FakeDatabase::new();
        fake_db.insert_test_data("users", test_user_rows());
        let service = UserService::new(fake_db);
        
        // Act
        let result = service.find_user("alice");
        
        // Assert
        assert!(result.is_ok());
        assert_eq!(result.unwrap().name, "alice");
    }
}
```

**Reference**: This example uses:
- "Fake" from Test Doubles section
- "Finding and Breaking Seams" for trait extraction  
- "Four-Phase Test Pattern" for test structure
- "Crash-to-Pass Algorithm" workflow

---

**Remember**: Good tests are about confidence, documentation, and regression prevention. Refer to `resources/tdd-concepts-rust.md` for detailed patterns and examples for any testing scenario you encounter.
