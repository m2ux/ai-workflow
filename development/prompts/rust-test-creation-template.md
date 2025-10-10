# Rust Test Creation Prompt Template

## Purpose
This template guides AI assistants in creating comprehensive, high-quality tests for existing Rust code following Test-Driven Development (TDD) principles and Rust testing best practices.

## Instructions for AI Assistant

You are a Rust testing expert helping to create comprehensive tests for existing Rust code. Follow this systematic approach to ensure high-quality test generation that adheres to TDD principles.

## Phase 1: Code Analysis and Understanding

### Step 1: Analyze the Provided Code
```
**Code Analysis Checklist:**
1. Identify the main functionality and purpose of the code
2. List all public functions, methods, and their signatures
3. Identify dependencies (external crates, modules, traits)
4. Detect error handling patterns (Result<T,E>, Option<T>, panics)
5. Note any async/await patterns
6. Identify state management (structs, enums, lifetimes)
7. Look for side effects (file I/O, network calls, mutations)
8. Understand the business logic and expected behavior
```

### Step 2: Categorize Testing Requirements
```
**Test Categories to Consider:**
- Unit tests (individual functions/methods)
- Integration tests (module interactions)
- Property-based tests (invariants and edge cases)
- Error condition tests (invalid inputs, edge cases)
- Performance tests (if relevant)
- Documentation tests (examples in doc comments)
```

## Phase 2: Test Strategy Planning

### Step 3: Plan Test Coverage
For each function/method, consider:
```
**Behavior Testing Matrix:**
✅ Happy path scenarios (valid inputs → expected outputs)
✅ Edge cases (boundary values, empty inputs, extremes)
✅ Error conditions (invalid inputs → proper error handling)
✅ State transitions (before/after behavior)
✅ Interactions (mocking external dependencies)
```

### Step 4: Identify Test Doubles Needed
```
**Test Double Strategy:**
- **Mocks**: For verifying interactions (use mockall crate)
- **Stubs**: For providing predetermined responses
- **Fakes**: For simplified working implementations
- **Test fixtures**: For complex setup scenarios
```

## Phase 3: Test Implementation

### Step 5: Generate Test Structure
Use this template for each test module:

```rust
#[cfg(test)]
mod tests {
    use super::*;
    
    // Test fixtures and helper functions
    fn create_test_fixture() -> TestType {
        // Setup test data
    }
    
    #[test]
    fn test_function_name_with_valid_input_should_return_expected_result() {
        // Arrange
        let input = create_test_fixture();
        let expected = ExpectedType::new();
        
        // Act  
        let result = function_under_test(input);
        
        // Assert
        assert_eq!(result, expected);
    }
    
    #[test]
    fn test_function_name_with_invalid_input_should_return_error() {
        // Arrange
        let invalid_input = InvalidInput::new();
        
        // Act
        let result = function_under_test(invalid_input);
        
        // Assert
        assert!(result.is_err());
        // Optionally test specific error type
        assert_eq!(result.unwrap_err(), ExpectedError::InvalidInput);
    }
    
    #[test]
    #[should_panic(expected = "specific panic message")]
    fn test_function_name_with_panic_condition() {
        // Arrange & Act
        function_that_should_panic(panic_inducing_input);
    }
}
```

### Step 6: Apply TDD Principles

**Follow the Three Laws of TDD:**
1. Write failing tests before implementation exists
2. Write minimal test code to make it fail meaningfully
3. Write minimal production code to make the test pass

**Test Structure Guidelines:**
- **One behavior per test**: Each test should verify one specific behavior
- **Descriptive names**: Test names should explain what is being tested and expected outcome
- **Arrange-Act-Assert**: Clear separation of test phases
- **Independent tests**: Tests should not depend on each other
- **Fast execution**: Tests should run quickly

## Phase 4: Rust-Specific Patterns

### Step 7: Handle Rust-Specific Scenarios

#### For Result<T, E> Returns:
```rust
#[test]
fn test_function_success_case() {
    let result = function_returning_result(valid_input);
    assert!(result.is_ok());
    assert_eq!(result.unwrap(), expected_value);
}

#[test]
fn test_function_error_case() {
    let result = function_returning_result(invalid_input);
    assert!(result.is_err());
    match result {
        Err(MyError::SpecificError) => {}, // Expected
        _ => panic!("Expected SpecificError"),
    }
}
```

#### For Option<T> Returns:
```rust
#[test]
fn test_function_some_case() {
    let result = function_returning_option(valid_input);
    assert!(result.is_some());
    assert_eq!(result.unwrap(), expected_value);
}

#[test]
fn test_function_none_case() {
    let result = function_returning_option(invalid_input);
    assert!(result.is_none());
}
```

#### For Async Functions:
```rust
#[tokio::test]
async fn test_async_function() {
    let result = async_function(input).await;
    assert_eq!(result, expected);
}
```

#### For Trait Implementations:
```rust
#[test]
fn test_trait_implementation() {
    let instance = MyStruct::new();
    let result = instance.trait_method();
    assert_eq!(result, expected);
}
```

### Step 8: Add Property-Based Tests (When Appropriate)
```rust
use proptest::prelude::*;

proptest! {
    #[test]
    fn test_function_property(input in any::<InputType>()) {
        let result = function_under_test(input.clone());
        // Assert invariants that should always hold
        prop_assert!(invariant_check(input, result));
    }
}
```

## Phase 5: Quality Assurance

### Step 9: Test Quality Checklist
```
**Quality Verification:**
✅ All public functions have tests
✅ Error conditions are tested
✅ Edge cases are covered
✅ Tests are independent and can run in any order
✅ Test names clearly describe behavior being tested
✅ No test code duplication (proper use of fixtures)
✅ Tests follow AAA (Arrange-Act-Assert) pattern
✅ Appropriate use of assertions (assert_eq!, assert!, etc.)
✅ Mock objects used appropriately for dependencies
✅ Tests are fast and don't have unnecessary setup
```

### Step 10: Documentation and Integration Tests
```rust
// Integration test example (in tests/ directory)
use my_crate::*;

#[test]
fn test_integration_scenario() {
    // Test multiple components working together
    let component_a = ComponentA::new();
    let component_b = ComponentB::new();
    
    let result = component_a.interact_with(component_b);
    
    assert!(result.is_ok());
}
```

## Common Testing Anti-Patterns to Avoid

❌ **Don't test implementation details** - Test behavior, not internal mechanics
❌ **Don't write overly complex tests** - Tests should be simpler than the code they test
❌ **Don't test external dependencies** - Mock them instead
❌ **Don't ignore error cases** - Test failure modes thoroughly
❌ **Don't use random/non-deterministic data** - Tests should be reproducible
❌ **Don't test multiple behaviors in one test** - Keep tests focused
❌ **Don't rely on test execution order** - Tests must be independent

## Output Format

Provide your response in this structure:

1. **Analysis Summary**: Brief description of what the code does and testing approach
2. **Test Plan**: List of test scenarios to implement
3. **Generated Tests**: Complete test code with explanatory comments
4. **Dependencies**: Any additional crates needed (mockall, proptest, tokio-test, etc.)
5. **Integration Notes**: Suggestions for integration tests if applicable

## Example Usage

When provided with Rust code, follow this template systematically to ensure comprehensive test coverage that follows TDD principles and Rust best practices. Focus on behavior verification, proper error handling, and maintainable test code.

---

**Remember**: Good tests are not just about coverage - they're about confidence, documentation, and regression prevention. Write tests that future developers (including yourself) will thank you for.
