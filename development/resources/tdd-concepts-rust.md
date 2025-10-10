# Test-Driven Development for Embedded C - Lexicon with Rust Equivalents

**Document Purpose**: Comprehensive lexicon of TDD concepts from "Test Driven Development for Embedded C" with direct translations to Rust idioms and practices.

**Last Updated**: 2025-10-10

---

## Table of Contents
1. [Core TDD Concepts](#core-tdd-concepts)
2. [Test Harnesses and Frameworks](#test-harnesses-and-frameworks)
3. [Test Structure and Organization](#test-structure-and-organization)
4. [Test Doubles](#test-doubles)
5. [Breaking Dependencies](#breaking-dependencies)
6. [TDD Workflow and State Machine](#tdd-workflow-and-state-machine)
7. [Test Quality Principles](#test-quality-principles)
8. [Examples and Patterns](#examples-and-patterns)
9. [Legacy Code and Characterization](#legacy-code-and-characterization)
10. [Advanced Techniques](#advanced-techniques)

---

## Core TDD Concepts

### Three Laws of TDD (Bob Martin)

**Summary**: Bob Martin composed the Three Laws of TDD to provide guidance on alternating between writing test code and production code. These laws enforce a discipline where you let the code follow the tests, sticking to this discipline produces comprehensive tests and thoroughly tested production code. The laws prevent code from getting ahead of tests and ensure that every piece of production code is driven by a failing test.

**C Version (Book)**:
1. Write no production code until you have a failing unit test
2. Write only enough of a unit test to make it fail
3. Write only enough production code to make the failing test pass

**Rust Equivalent**:
```rust
// Law 1: Write test first (it won't compile yet)
#[test]
fn test_calculate_sum_returns_correct_total() {
    let result = calculate_sum(vec![1, 2, 3]);
    assert_eq!(result, 6);
}

// Law 2: Minimal test to fail meaningfully
// Law 3: Minimal implementation to pass
fn calculate_sum(numbers: Vec<i32>) -> i32 {
    numbers.iter().sum()
}
```

### Red-Green-Refactor Cycle

**Summary**: The cornerstone of TDD workflow. Once a test passes, you know you have the desired behavior, but your work isn't done yet - the code needs to be left clean. While making the test pass, it's OK to make a mess, but don't leave the mess; refactor it out. The cycle emphasizes that working software is not the end goal - working, clean software is. Tests provide the safety net that allows fearless refactoring.

**C Version**: Write failing test → Make it pass → Clean up code

**Rust Equivalent**:
- **Red**: `cargo test` fails (compilation or assertion)
- **Green**: `cargo test` passes
- **Refactor**: Improve code while keeping `cargo test` green

### TDD State Machine

**Summary**: The TDD state machine describes the mechanical steps a developer goes through when practicing TDD. You choose a test representing the next increment of behavior and express the desired outcome. Then you make the compiler happy as you design the interface, resolve link errors by creating minimal stubs, watch the test fail to confirm it can detect wrong behavior, make it pass with the simplest implementation, and finally refactor to make it right. This state machine helps developers focus on solving one problem at a time through methodical, incremental progress.

**C Version**: Compilation error → Link error → Test fails → Test passes → Refactor

**Rust Equivalent**:
```bash
# Red (compilation error)
error[E0425]: cannot find function `calculate_sum`

# Red (test fails)  
thread 'test_calculate_sum' panicked at assertion failed

# Green (test passes)
test test_calculate_sum ... ok

# Refactor while maintaining green
```

### Fake It Till You Make It

**Summary**: A valuable TDD technique where you start with the simplest possible implementation - even hard-coded return values. These simple implementations test your tests, showing that the test can detect both wrong and right results. Watching the test case fail shows that the test can detect a wrong result; hard-coding the right answer shows that the test case can detect the right result. The test is right and valuable, even though the production code is incomplete. Later, as the implementation evolves through triangulation (adding more tests), these seemingly trivial tests will test important behavior and boundary conditions.

**C Version**: Hard-code return values initially, then generalize

**Rust Equivalent**:
```rust
// Step 1: Fake it (hard-coded)
fn get_led_state(led: u8) -> bool {
    false  // Just make test pass
}

// Step 2: Triangulation with more tests
#[test]
fn test_led_on_returns_true() {
    turn_on_led(1);
    assert!(get_led_state(1));
}

// Step 3: Real implementation
fn get_led_state(led: u8) -> bool {
    (leds_state & (1 << led)) != 0
}
```

---

## Test Harnesses and Frameworks

### Unity Test Framework (C)

**C Version**:
```c
TEST_GROUP(LedDriver);

TEST_SETUP(LedDriver) {
    // Setup code
}

TEST_TEAR_DOWN(LedDriver) {
    // Cleanup code
}

TEST(LedDriver, LedsOffAfterCreate) {
    TEST_ASSERT_EQUAL_HEX16(0x0, virtualLeds);
}

TEST_GROUP_RUNNER(LedDriver) {
    RUN_TEST_CASE(LedDriver, LedsOffAfterCreate);
}
```

**Rust Equivalent**:
```rust
#[cfg(test)]
mod led_driver_tests {
    use super::*;
    
    // Test fixture setup
    fn setup() -> LedDriver {
        LedDriver::new(0xFF00)
    }
    
    #[test]
    fn leds_off_after_create() {
        let driver = setup();
        assert_eq!(driver.get_all_leds(), 0x0);
    }
    
    // Teardown happens automatically via Drop trait
}
```

### CppUTest Framework (C++)

**C Version**:
```cpp
TEST_GROUP(LedDriver) {
    void setup() {
        ledDriver = LedDriver_Create(&virtualLeds);
    }
    
    void teardown() {
        LedDriver_Destroy(ledDriver);
    }
};

TEST(LedDriver, LedsOffAfterCreate) {
    LONGS_EQUAL(0, virtualLeds);
}
```

**Rust Equivalent**:
```rust
#[cfg(test)]
mod tests {
    use super::*;
    
    struct TestFixture {
        led_driver: LedDriver,
        virtual_leds: u16,
    }
    
    impl TestFixture {
        fn new() -> Self {
            let virtual_leds = 0;
            let led_driver = LedDriver::new(&virtual_leds);
            Self { led_driver, virtual_leds }
        }
    }
    
    // Drop trait provides automatic teardown
    impl Drop for TestFixture {
        fn drop(&mut self) {
            // Cleanup if needed
        }
    }
    
    #[test]
    fn leds_off_after_create() {
        let fixture = TestFixture::new();
        assert_eq!(fixture.virtual_leds, 0);
    }
}
```

---

## Test Structure and Organization

### Four-Phase Test Pattern

**Summary**: Described by Gerard Meszaros in "xUnit Testing Patterns," the Four-Phase Test pattern creates concise, readable, and well-structured tests. Following this pattern allows test readers to quickly determine what is being tested. The four phases are: Setup (establish preconditions), Exercise (perform the operation being tested), Verify (check the results), and Teardown (return system to original state). In the Four-Phase Test pattern, the test case has the responsibility to set up and clean up the Depended-On Components (DOCs), managing all test dependencies properly.

**C Version**: Setup → Exercise → Verify → Teardown

**Rust Equivalent** (Also called Arrange-Act-Assert or AAA):
```rust
#[test]
fn test_led_driver_turns_on_led() {
    // Arrange (Setup)
    let mut driver = LedDriver::new(0xFF00);
    
    // Act (Exercise)
    driver.turn_on(7);
    
    // Assert (Verify)
    assert!(driver.is_on(7));
    
    // Cleanup happens automatically via Drop
}
```

### Test Fixtures

**Summary**: Duplication reduction is the motivation for a test fixture. A test fixture helps organize the common facilities needed by all the tests in one place. With every test added, duplication will crowd out and obscure the code that is essential to understand the test case. Test fixtures extract repeated setup and teardown code, making the actual test logic clear and focused on what's being tested rather than how to set up the test environment.

**C Version**: Shared setup/teardown in `TEST_SETUP()` and `TEST_TEAR_DOWN()`

**Rust Equivalent**:
```rust
#[cfg(test)]
mod tests {
    use super::*;
    
    // Helper function for common setup
    fn create_test_led_driver() -> LedDriver {
        LedDriver::new(0xFF00)
    }
    
    #[test]
    fn test_turn_on() {
        let mut driver = create_test_led_driver();
        driver.turn_on(5);
        assert!(driver.is_on(5));
    }
    
    #[test]
    fn test_turn_off() {
        let mut driver = create_test_led_driver();
        driver.turn_off(5);
        assert!(!driver.is_on(5));
    }
}
```

### Test Groups

**C Version**: `TEST_GROUP(GroupName)` organizes related tests

**Rust Equivalent**: Use modules
```rust
#[cfg(test)]
mod led_driver_tests {
    use super::*;
    
    mod boundary_conditions {
        use super::*;
        
        #[test]
        fn test_out_of_bounds_led_is_off() {
            let driver = LedDriver::new(0xFF00);
            assert!(!driver.is_on(0));
            assert!(!driver.is_on(17));
        }
    }
    
    mod state_transitions {
        use super::*;
        
        #[test]
        fn test_led_can_toggle() {
            let mut driver = LedDriver::new(0xFF00);
            driver.turn_on(5);
            driver.turn_off(5);
            assert!(!driver.is_on(5));
        }
    }
}
```

---

## Test Doubles

### Concept Overview

**Summary**: Test doubles are inspired by Hollywood stunt doubles - they take the place of the real actor in specific situations to keep things simple and safe. Just as a stunt double knows his part and can fall off a building really well, test doubles replace production dependencies in very specific testing situations. This helps test doubles stay simple, much simpler than the thing being replaced. Test doubles provide indirect inputs (return values) to the Code Under Test (CUT) or capture and possibly check indirect outputs (parameters) sent by the CUT to the test double. They enable testing by isolating the code under test from problematic dependencies like hardware, operating systems, or other complex modules.

**Definition**: Test doubles replace production dependencies to enable isolated testing

**Types**: Dummy, Stub, Spy, Mock, Fake

### Test Dummy

**Summary**: A test dummy is the simplest form of test double. It keeps the linker from rejecting your build by satisfying the compiler, linker, or runtime dependency, but it is never actually called during the test. It's provided purely to satisfy interface requirements when the test doesn't actually exercise that particular dependency.

**C Version**: Placeholder that satisfies compiler/linker but is never used

**Rust Equivalent**:
```rust
struct DummyLogger;

impl Logger for DummyLogger {
    fn log(&self, _message: &str) {
        // Never called, just satisfies trait bound
    }
}

#[test]
fn test_component_without_logging() {
    let dummy_logger = DummyLogger;
    let component = Component::new(dummy_logger);
    // Test component behavior
}
```

### Stub

**Summary**: Stubs provide predetermined responses to calls made during tests. They return canned answers to queries, allowing you to control the indirect inputs to the code under test. Stubs don't verify that they're called correctly - they simply provide the data needed to exercise a specific test scenario.

**C Version**: Returns predetermined values

**Rust Equivalent**:
```rust
struct StubTimeService;

impl TimeService for StubTimeService {
    fn get_time(&self) -> DateTime {
        // Always returns same fixed time
        DateTime::new(2025, 10, 10, 14, 30, 0)
    }
}

#[test]
fn test_scheduler_with_fixed_time() {
    let time_service = StubTimeService;
    let scheduler = Scheduler::new(time_service);
    // Test time-dependent behavior with predictable time
}
```

### Spy

**Summary**: A spy is a test double that records information about how it was used during the test. Unlike a stub which just returns values, a spy remembers what was called, with what parameters, and in what order. The test can then query the spy to verify that the code under test interacted with its dependency correctly. Spies are particularly useful for verifying indirect outputs - calls made by the code under test that produce side effects.

**C Version**: Records information about calls for later verification

**Rust Equivalent**:
```rust
use std::cell::RefCell;

struct SpyLightController {
    calls: RefCell<Vec<(u8, LightAction)>>,
}

impl SpyLightController {
    fn new() -> Self {
        Self {
            calls: RefCell::new(Vec::new()),
        }
    }
    
    fn verify_called_with(&self, led: u8, action: LightAction) -> bool {
        self.calls.borrow().contains(&(led, action))
    }
}

impl LightController for SpyLightController {
    fn on(&self, led: u8) {
        self.calls.borrow_mut().push((led, LightAction::On));
    }
    
    fn off(&self, led: u8) {
        self.calls.borrow_mut().push((led, LightAction::Off));
    }
}

#[test]
fn test_scheduler_controls_light() {
    let spy = SpyLightController::new();
    let mut scheduler = Scheduler::new(&spy);
    
    scheduler.schedule_turn_on(7, DateTime::new(2025, 10, 10, 18, 0, 0));
    scheduler.wake_up(DateTime::new(2025, 10, 10, 18, 0, 0));
    
    assert!(spy.verify_called_with(7, LightAction::On));
}
```

### Mock Object

**Summary**: A mock object (or simply "the mock") is a test double that allows a test case to describe the calls expected from one module to another. During test execution, the mock checks that all calls happen with the right parameters and in the right order. The mock can also be instructed to return specific values in proper sequence to the code under test. When stubs won't work for more complex interactions, mock objects provide sophisticated verification of behavior and call patterns.

**C Version**: Verifies interactions and call sequences using mock frameworks

**Rust Equivalent** (using `mockall` crate):
```rust
use mockall::*;

#[automock]
trait FlashDriver {
    fn write(&mut self, address: u32, data: u8) -> Result<(), FlashError>;
    fn read(&self, address: u32) -> Result<u8, FlashError>;
}

#[test]
fn test_flash_write_verifies_sequence() {
    let mut mock_flash = MockFlashDriver::new();
    
    // Setup expectations
    mock_flash.expect_write()
        .with(eq(0x1000), eq(0x42))
        .times(1)
        .returning(|_, _| Ok(()));
    
    mock_flash.expect_read()
        .with(eq(0x1000))
        .times(1)
        .returning(|_| Ok(0x42));
    
    // Exercise
    let mut device = Device::new(mock_flash);
    device.store_data(0x1000, 0x42).unwrap();
    
    // Verification happens automatically on drop
}
```

### Fake

**Summary**: A fake is a working implementation of an interface that takes shortcuts to make it suitable for testing but not for production. For example, an in-memory database is a fake - it implements database operations but uses RAM instead of persistent storage. Fakes allow testing of complex interactions without the overhead, complexity, or unreliability of real implementations. They're particularly useful when the real implementation would involve hardware, network calls, or other resources that are slow or unavailable in the test environment.

**C Version**: Working implementation with shortcuts (e.g., in-memory database)

**Rust Equivalent**:
```rust
use std::collections::HashMap;

struct FakeFlashDriver {
    memory: HashMap<u32, u8>,
}

impl FakeFlashDriver {
    fn new() -> Self {
        Self {
            memory: HashMap::new(),
        }
    }
}

impl FlashDriver for FakeFlashDriver {
    fn write(&mut self, address: u32, data: u8) -> Result<(), FlashError> {
        self.memory.insert(address, data);
        Ok(())
    }
    
    fn read(&self, address: u32) -> Result<u8, FlashError> {
        self.memory.get(&address)
            .copied()
            .ok_or(FlashError::InvalidAddress)
    }
}

#[test]
fn test_device_with_fake_flash() {
    let fake_flash = FakeFlashDriver::new();
    let mut device = Device::new(fake_flash);
    
    device.store_data(0x1000, 0xAB).unwrap();
    assert_eq!(device.read_data(0x1000).unwrap(), 0xAB);
}
```

### Exploding Fake

**Summary**: An exploding fake is a test double that detects and reports when it's being used incorrectly. Rather than silently accepting invalid operations, it "explodes" (panics or reports errors) to help identify programming mistakes during testing. This helps catch bugs early by making incorrect usage immediately obvious rather than allowing subtle failures to propagate.

**C Version**: Fake that reports usage errors to help identify problems

**Rust Equivalent**:
```rust
struct ExplodingFlashDriver {
    initialized: bool,
}

impl FlashDriver for ExplodingFlashDriver {
    fn write(&mut self, address: u32, data: u8) -> Result<(), FlashError> {
        if !self.initialized {
            panic!("Flash driver used before initialization!");
        }
        if address > 0xFFFF {
            panic!("Write to invalid address: 0x{:X}", address);
        }
        Ok(())
    }
}
```

---

## Breaking Dependencies

**Summary**: Real code has dependencies - one module interacts with several others to get its job done. Code resists automated tests when it interacts with the operating system, hardware devices, or sometimes other modules. To test the code under test, problematic dependencies must be broken. The essential idea is that the test case and the test doubles together form a software test fixture that surrounds the CUT, driving its inputs and monitoring and checking its outputs. All access to the execution environment must go through defined interfaces, and interface calls can be intercepted and inspected by replacing a problem Depended-Upon Component with a test double.

### Dependency Injection

**C Version**: Pass hardware addresses and dependencies as parameters

**C Example**:
```c
LedDriver * LedDriver_Create(uint16_t *address) {
    ledDriver.ledsAddress = address;
    *ledDriver.ledsAddress = 0;
    return &ledDriver;
}
```

**Rust Equivalent**:
```rust
pub struct LedDriver<'a> {
    leds_address: &'a mut u16,
}

impl<'a> LedDriver<'a> {
    pub fn new(address: &'a mut u16) -> Self {
        *address = 0;
        Self { leds_address: address }
    }
    
    pub fn turn_on(&mut self, led_number: u8) {
        *self.leds_address |= 1 << (led_number - 1);
    }
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn leds_off_after_create() {
        let mut virtual_leds: u16 = 0xFFFF;
        let driver = LedDriver::new(&mut virtual_leds);
        assert_eq!(virtual_leds, 0x0);
    }
}
```

### Link-Time Substitution

**C Version**: Replace functions at link time using weak symbols or linker flags

**Rust Equivalent**: Use trait objects or feature flags
```rust
// Production code
#[cfg(not(test))]
fn get_system_time() -> SystemTime {
    SystemTime::now()
}

// Test code
#[cfg(test)]
fn get_system_time() -> SystemTime {
    SystemTime::UNIX_EPOCH + Duration::from_secs(1_000_000)
}

// Or using traits
pub trait TimeProvider {
    fn now(&self) -> SystemTime;
}

pub struct RealTimeProvider;
impl TimeProvider for RealTimeProvider {
    fn now(&self) -> SystemTime {
        SystemTime::now()
    }
}

pub struct FakeTimeProvider {
    fixed_time: SystemTime,
}

impl TimeProvider for FakeTimeProvider {
    fn now(&self) -> SystemTime {
        self.fixed_time
    }
}
```

### Function Pointer Substitution

**C Version**: Use function pointers to swap implementations

**C Example**:
```c
typedef int (*RandomMinuteFunc)(void);
static RandomMinuteFunc randomMinute = RandomMinute_Get;

void RandomMinute_SetFunction(RandomMinuteFunc func) {
    randomMinute = func;
}
```

**Rust Equivalent**: Use trait objects or function pointers
```rust
type RandomMinuteFn = fn() -> i32;

pub struct Scheduler {
    random_minute: RandomMinuteFn,
}

impl Scheduler {
    pub fn new(random_minute: RandomMinuteFn) -> Self {
        Self { random_minute }
    }
    
    pub fn with_default() -> Self {
        Self::new(get_random_minute)
    }
}

#[test]
fn test_with_fake_random() {
    fn fake_random() -> i32 { 42 }
    let scheduler = Scheduler::new(fake_random);
    // Test with predictable random value
}

// Or using trait objects for more flexibility
trait RandomMinuteProvider {
    fn get_random_minute(&self) -> i32;
}

pub struct Scheduler<R: RandomMinuteProvider> {
    random_provider: R,
}
```

### Preprocessor Substitution

**C Version**: Use `#ifdef` to swap implementations

**C Example**:
```c
#ifdef TEST
    #include "FakeFlash.h"
#else
    #include "Flash.h"
#endif
```

**Rust Equivalent**: Use feature flags or cfg attributes
```rust
#[cfg(not(test))]
mod flash {
    pub use crate::hardware::flash::*;
}

#[cfg(test)]
mod flash {
    pub use crate::fakes::fake_flash::*;
}

// Or with cargo features
#[cfg(feature = "hardware-flash")]
pub fn get_flash_driver() -> Box<dyn FlashDriver> {
    Box::new(HardwareFlash::new())
}

#[cfg(not(feature = "hardware-flash"))]
pub fn get_flash_driver() -> Box<dyn FlashDriver> {
    Box::new(FakeFlash::new())
}
```

---

## TDD Workflow and State Machine

**Summary**: The TDD workflow guides developers through a disciplined, incremental approach to software development. The workflow prevents "Debug-Later Programming" by catching errors immediately when they're introduced. Each step of the way, new automated unit tests are written, followed immediately by code satisfying those tests. As the production code grows, so does a suite of unit tests, which is an asset as valuable as the production code itself. With every code change, the test suite runs, checking the new code's function but also checking all existing code for compatibility with the latest change. This approach may seem slower initially ("slow down to go fast"), but it leads to faster overall development by preventing bugs rather than debugging them later.

### TDD Developer State Machine

**C Version States**:
1. Choose a test
2. Write the test
3. Make the test compile
4. Make the test link
5. Make the test pass
6. Refactor (make it right)
7. All tests pass

**Rust Equivalent Workflow**:
```bash
# 1. Choose a test (plan what to implement)

# 2. Write the test
#[test]
fn test_new_feature() { ... }

# 3. Make it compile
cargo test
# error[E0425]: cannot find function `new_feature`

# Add function signature
fn new_feature() -> Result<(), Error> {
    unimplemented!()
}

# 4. Make it link (Rust handles this automatically)

# 5. Make it pass
fn new_feature() -> Result<(), Error> {
    Ok(())  // Minimal implementation
}

# 6. Refactor
fn new_feature() -> Result<(), Error> {
    // Proper implementation
    do_something_complex()?;
    Ok(())
}

# 7. All tests pass
cargo test
# test result: ok. 42 passed
```

### Incremental Development (Baby Steps)

**Summary**: TDD emphasizes taking small steps - adding one behavior at a time rather than trying to implement everything at once. Why do these small steps? They allow you to focus on solving one problem at a time. You're methodically and incrementally adding and verifying behavior. This isn't procrastination - it's disciplined development. Each small step provides immediate feedback, and if something goes wrong, you know exactly what caused it because you only changed one small thing.

**C Version**: Add one small behavior at a time

**Rust Equivalent**:
```rust
// Test 1: Basic creation
#[test]
fn circular_buffer_is_empty_after_creation() {
    let buffer = CircularBuffer::new(10);
    assert!(buffer.is_empty());
}

// Test 2: Can put one item
#[test]
fn buffer_not_empty_after_put() {
    let mut buffer = CircularBuffer::new(10);
    buffer.put(42).unwrap();
    assert!(!buffer.is_empty());
}

// Test 3: Can get item back
#[test]
fn buffer_returns_put_value() {
    let mut buffer = CircularBuffer::new(10);
    buffer.put(42).unwrap();
    assert_eq!(buffer.get().unwrap(), 42);
}

// Continue incrementally...
```

---

## Test Quality Principles

### FIRST Principles

**Summary**: The FIRST principles define the characteristics of good unit tests. Tests should be Fast (run quickly so they can be run frequently), Independent (tests don't depend on each other and can run in any order), Repeatable (produce the same results every time, with no randomness or environmental dependencies), Self-validating (have clear pass/fail criteria with no manual inspection needed), and Timely (written at the right time - ideally before the production code). Following these principles ensures tests remain valuable and maintainable throughout the project lifecycle.

**C/Rust Universal Concept**:
- **F**ast: Tests run quickly
- **I**ndependent: Tests don't depend on each other
- **R**epeatable: Same results every time
- **S**elf-validating: Clear pass/fail
- **T**imely: Written at the right time (ideally first)

**Rust Implementation**:
```rust
// Fast: No I/O, no sleeps, minimal setup
#[test]
fn fast_test() {
    let result = calculate_sum(vec![1, 2, 3]);
    assert_eq!(result, 6);  // Completes in microseconds
}

// Independent: Each test has own data
#[test]
fn independent_test_1() {
    let mut data = vec![1, 2, 3];
    process(&mut data);
    assert_eq!(data, vec![2, 4, 6]);
}

#[test]
fn independent_test_2() {
    let mut data = vec![5, 10];  // Own data, not shared
    process(&mut data);
    assert_eq!(data, vec![10, 20]);
}

// Repeatable: No random data, fixed time
#[test]
fn repeatable_test() {
    let time_provider = FakeTimeProvider::new(fixed_time());
    let scheduler = Scheduler::new(time_provider);
    // Always same result
}

// Self-validating: Assert statements give clear pass/fail
#[test]
fn self_validating_test() {
    let result = validate_input("test");
    assert!(result.is_ok());  // Clear pass/fail
}

// Timely: Write test before implementation (TDD)
```

### Test List

**Summary**: Before diving into implementation, create a test list to help organize your thoughts. You don't expect to make a perfect test list - start with whatever tests you can think of, and evolve the test list as you learn more. The test list serves as a roadmap for development, helping you stay focused and ensuring you don't forget important test scenarios. As you complete tests, check them off; as you discover new scenarios, add them to the list.

**C Version**: Write down all tests you can think of before implementing

**Rust Equivalent**:
```rust
// Comment-based test list
mod led_driver_tests {
    // Test List:
    // ✅ Leds off after create
    // ✅ Turn on LED 1
    // ✅ Turn off LED 1
    // ✅ Turn on multiple LEDs
    // ⏳ Out of bounds LEDs are always off
    // ⏳ Remember LED state
    // ⏳ All LEDs on
    // ⏳ All LEDs off
    
    // Or use #[ignore] for pending tests
    #[test]
    fn leds_off_after_create() { /* ... */ }
    
    #[test]
    #[ignore = "not yet implemented"]
    fn out_of_bounds_leds_are_always_off() {
        todo!("implement boundary checking")
    }
}
```

---

## Examples and Patterns

### LED Driver Example

**C Version**: Classic embedded example of controlling LEDs through memory-mapped I/O

**Rust Equivalent**:
```rust
pub struct LedDriver {
    leds_address: *mut u16,
}

impl LedDriver {
    pub fn new(address: *mut u16) -> Self {
        unsafe {
            *address = 0;
        }
        Self { leds_address: address }
    }
    
    pub fn turn_on(&mut self, led_number: u8) -> Result<(), LedError> {
        if !self.is_valid_led_number(led_number) {
            return Err(LedError::InvalidLedNumber);
        }
        
        unsafe {
            *self.leds_address |= self.convert_led_number_to_bit(led_number);
        }
        Ok(())
    }
    
    pub fn turn_off(&mut self, led_number: u8) -> Result<(), LedError> {
        if !self.is_valid_led_number(led_number) {
            return Err(LedError::InvalidLedNumber);
        }
        
        unsafe {
            *self.leds_address &= !self.convert_led_number_to_bit(led_number);
        }
        Ok(())
    }
    
    pub fn is_on(&self, led_number: u8) -> bool {
        if !self.is_valid_led_number(led_number) {
            return false;
        }
        
        unsafe {
            (*self.leds_address & self.convert_led_number_to_bit(led_number)) != 0
        }
    }
    
    fn is_valid_led_number(&self, led_number: u8) -> bool {
        led_number >= 1 && led_number <= 16
    }
    
    fn convert_led_number_to_bit(&self, led_number: u8) -> u16 {
        1 << (led_number - 1)
    }
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn leds_off_after_create() {
        let mut virtual_leds: u16 = 0xFFFF;
        let _driver = LedDriver::new(&mut virtual_leds as *mut u16);
        assert_eq!(virtual_leds, 0x0);
    }
    
    #[test]
    fn turn_on_led_one() {
        let mut virtual_leds: u16 = 0;
        let mut driver = LedDriver::new(&mut virtual_leds as *mut u16);
        driver.turn_on(1).unwrap();
        assert_eq!(virtual_leds, 0x1);
    }
    
    #[test]
    fn out_of_bounds_leds_are_always_off() {
        let mut virtual_leds: u16 = 0;
        let driver = LedDriver::new(&mut virtual_leds as *mut u16);
        assert!(!driver.is_on(0));
        assert!(!driver.is_on(17));
    }
}
```

### Circular Buffer Example

**C Version**: Ring buffer data structure

**Rust Equivalent**:
```rust
pub struct CircularBuffer<T> {
    buffer: Vec<Option<T>>,
    capacity: usize,
    read_index: usize,
    write_index: usize,
    count: usize,
}

impl<T> CircularBuffer<T> {
    pub fn new(capacity: usize) -> Self {
        Self {
            buffer: (0..capacity).map(|_| None).collect(),
            capacity,
            read_index: 0,
            write_index: 0,
            count: 0,
        }
    }
    
    pub fn is_empty(&self) -> bool {
        self.count == 0
    }
    
    pub fn is_full(&self) -> bool {
        self.count == self.capacity
    }
    
    pub fn put(&mut self, value: T) -> Result<(), BufferError> {
        if self.is_full() {
            return Err(BufferError::Full);
        }
        
        self.buffer[self.write_index] = Some(value);
        self.write_index = (self.write_index + 1) % self.capacity;
        self.count += 1;
        Ok(())
    }
    
    pub fn get(&mut self) -> Result<T, BufferError> {
        if self.is_empty() {
            return Err(BufferError::Empty);
        }
        
        let value = self.buffer[self.read_index].take()
            .ok_or(BufferError::Empty)?;
        self.read_index = (self.read_index + 1) % self.capacity;
        self.count -= 1;
        Ok(value)
    }
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn buffer_is_empty_after_creation() {
        let buffer: CircularBuffer<i32> = CircularBuffer::new(10);
        assert!(buffer.is_empty());
    }
    
    #[test]
    fn buffer_not_empty_after_put() {
        let mut buffer = CircularBuffer::new(10);
        buffer.put(42).unwrap();
        assert!(!buffer.is_empty());
    }
    
    #[test]
    fn buffer_returns_value_in_fifo_order() {
        let mut buffer = CircularBuffer::new(3);
        buffer.put(1).unwrap();
        buffer.put(2).unwrap();
        buffer.put(3).unwrap();
        
        assert_eq!(buffer.get().unwrap(), 1);
        assert_eq!(buffer.get().unwrap(), 2);
        assert_eq!(buffer.get().unwrap(), 3);
    }
    
    #[test]
    fn buffer_returns_error_when_full() {
        let mut buffer = CircularBuffer::new(2);
        buffer.put(1).unwrap();
        buffer.put(2).unwrap();
        
        assert!(matches!(buffer.put(3), Err(BufferError::Full)));
    }
}
```

### Flash Driver Example

**C Version**: Testing Flash memory operations with mocks

**Rust Equivalent**:
```rust
use mockall::*;

#[automock]
pub trait IoDriver {
    fn write(&mut self, address: u32, value: u8);
    fn read(&self, address: u32) -> u8;
}

pub struct FlashDriver<I: IoDriver> {
    io: I,
}

impl<I: IoDriver> FlashDriver<I> {
    pub fn new(io: I) -> Self {
        Self { io }
    }
    
    pub fn write(&mut self, address: u32, data: u8) -> Result<(), FlashError> {
        const COMMAND_REGISTER: u32 = 0x80;
        const PROGRAM_COMMAND: u8 = 0x40;
        const STATUS_REGISTER: u32 = 0x80;
        const READY_BIT: u8 = 1 << 7;
        
        self.io.write(COMMAND_REGISTER, PROGRAM_COMMAND);
        self.io.write(address, data);
        
        // Wait for ready
        while (self.io.read(STATUS_REGISTER) & READY_BIT) == 0 {
            // Spin wait
        }
        
        // Verify
        if self.io.read(address) != data {
            return Err(FlashError::VerificationFailed);
        }
        
        Ok(())
    }
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn flash_write_succeeds() {
        let mut mock_io = MockIoDriver::new();
        
        // Setup expectations
        mock_io.expect_write()
            .with(eq(0x80), eq(0x40))
            .times(1)
            .return_const(());
        
        mock_io.expect_write()
            .with(eq(0x1000), eq(0xAB))
            .times(1)
            .return_const(());
        
        mock_io.expect_read()
            .with(eq(0x80))
            .times(1)
            .returning(|_| 0x80);  // READY_BIT set
        
        mock_io.expect_read()
            .with(eq(0x1000))
            .times(1)
            .returning(|_| 0xAB);  // Verification
        
        let mut flash = FlashDriver::new(mock_io);
        flash.write(0x1000, 0xAB).unwrap();
    }
}
```

---

## Advanced Techniques

### Test-Driven Bug Fixes

**Summary**: Bug fixes need tests too. The existence of a bug often shows where prior test efforts have failed. If you can write a unit test to reveal the bug, do so. If investigation is needed to track down the bug, do the investigation and capture some of your knowledge in tests as you go. Once the bug is isolated in a test, fix it and ensure the test passes. This test then becomes part of the regression suite, preventing the bug from ever returning.

**C Version**: Write test that reveals the bug, then fix it

**Rust Equivalent**:
```rust
// Bug report: System crashes when processing empty input

#[test]
fn test_handles_empty_input_without_panic() {
    let processor = DataProcessor::new();
    let result = processor.process(vec![]);
    
    assert!(result.is_ok());  // Should not panic
    assert_eq!(result.unwrap(), ProcessResult::Empty);
}

// This test fails and reveals the bug
// Now fix the bug to make it pass

pub fn process(&self, data: Vec<u8>) -> Result<ProcessResult, Error> {
    if data.is_empty() {
        return Ok(ProcessResult::Empty);  // Bug fix
    }
    
    // Original code that assumed non-empty
    let first = data[0];
    // ...
}
```

### Learning Tests

**Summary**: Learning tests are tests written not to test your own code, but to learn how third-party code works. When working with a new library or API, write tests that exercise the features you plan to use. These tests document your understanding, verify that the library works as you expect, and will alert you if a library upgrade changes behavior you depend on. Learning tests are free (or maybe better than free!) - they provide documentation and confidence at minimal cost.

**C Version**: Write tests to understand third-party libraries

**Rust Equivalent**:
```rust
#[cfg(test)]
mod chrono_learning_tests {
    use chrono::*;
    
    #[test]
    fn learn_chrono_datetime_creation() {
        let dt = Utc.ymd(2025, 10, 10).and_hms(14, 30, 0);
        assert_eq!(dt.year(), 2025);
        assert_eq!(dt.month(), 10);
        assert_eq!(dt.day(), 10);
    }
    
    #[test]
    fn learn_chrono_duration_arithmetic() {
        let dt1 = Utc.ymd(2025, 10, 10).and_hms(14, 0, 0);
        let dt2 = dt1 + Duration::hours(2);
        assert_eq!(dt2.hour(), 16);
    }
    
    #[test]
    fn learn_chrono_formatting() {
        let dt = Utc.ymd(2025, 10, 10).and_hms(14, 30, 0);
        let formatted = dt.format("%Y-%m-%d %H:%M:%S").to_string();
        assert_eq!(formatted, "2025-10-10 14:30:00");
    }
}
```

### Crash to Pass

**Summary**: Adding the first test to legacy code is usually the hardest. Knowing what to expect and how to react can ease the process. The crash-to-pass algorithm helps you work through the challenges of getting legacy code into a test harness. You want to test some existing legacy code that is part of an interwoven mass of dependencies. The algorithm walks you through compilation errors, link errors, and runtime crashes, systematically addressing each problem by adding minimal stubs and test doubles until you have a passing test. Once the test infrastructure is in place, adding more tests becomes progressively easier.

**C Version**: Algorithm for adding tests to legacy code
1. Try to build test executable
2. If link fails, create minimal stub
3. If test crashes, add fake/stub
4. Make test pass
5. Refactor

**Rust Equivalent**:
```rust
// Step 1: Try to write test for legacy code
#[test]
fn test_legacy_process_data() {
    let result = legacy_process_data(&[1, 2, 3]);
    assert!(result.is_ok());
}

// Step 2: Compilation errors reveal dependencies
// error: no method named `legacy_process_data`

// Step 3: Create minimal interface
fn legacy_process_data(data: &[u8]) -> Result<Vec<u8>, Error> {
    todo!("extract from legacy code")
}

// Step 4: Test compiles but may panic/crash
// Add necessary fakes/stubs for dependencies

// Step 5: Extract actual implementation
fn legacy_process_data(data: &[u8]) -> Result<Vec<u8>, Error> {
    // Extracted and refactored legacy code
    Ok(data.iter().map(|&x| x * 2).collect())
}

// Step 6: Refactor while keeping tests green
```

### Characterization Tests

**Summary**: Characterization tests describe the current behavior of existing code, not necessarily the ideal behavior. When working with legacy code that you don't fully understand, characterization tests document what the code actually does. This creates a safety net for refactoring - the tests will alert you if your changes alter the behavior. As you understand the code better, you can evolve these tests to enforce correct behavior rather than just current behavior.

**C Version**: Tests that describe existing behavior of legacy code

**Rust Equivalent**:
```rust
// Documenting existing behavior, not ideal behavior
#[test]
fn characterize_legacy_quirk_returns_minus_one_for_empty() {
    // This documents that empty input returns -1
    // Not necessarily correct, but current behavior
    let legacy = LegacyComponent::new();
    let result = legacy.process_empty();
    assert_eq!(result, -1);  // Documenting existing behavior
}

#[test]
fn characterize_legacy_handles_max_value() {
    let legacy = LegacyComponent::new();
    let result = legacy.process(u32::MAX);
    // Document what actually happens, even if surprising
    assert_eq!(result, 0);  // Wraps around
}
```

---

## Assertion Macros

### C Assertions (Unity/CppUTest)

**C Version**:
```c
TEST_ASSERT_EQUAL(expected, actual);
TEST_ASSERT_EQUAL_HEX16(expected, actual);
TEST_ASSERT_TRUE(condition);
TEST_ASSERT_FALSE(condition);
TEST_ASSERT_NOT_NULL(pointer);
TEST_ASSERT_NULL(pointer);
TEST_ASSERT_EQUAL_STRING(expected, actual);
```

**Rust Equivalent**:
```rust
// Basic assertions
assert_eq!(expected, actual);
assert_ne!(not_expected, actual);
assert!(condition);
assert!(!condition);

// With custom messages
assert_eq!(expected, actual, "Values should match: expected {}, got {}", expected, actual);

// Result/Option assertions
assert!(result.is_ok());
assert!(result.is_err());
assert!(option.is_some());
assert!(option.is_none());

// Pattern matching assertions
assert!(matches!(result, Ok(42)));
assert!(matches!(error, Error::InvalidInput));

// Floating point comparisons
assert!((expected - actual).abs() < 0.0001);

// Custom assertions
fn assert_hex_eq(expected: u16, actual: u16) {
    assert_eq!(expected, actual, 
        "Expected 0x{:04X}, got 0x{:04X}", expected, actual);
}

// Panic testing
#[test]
#[should_panic(expected = "index out of bounds")]
fn test_panics_on_invalid_index() {
    let vec = vec![1, 2, 3];
    let _ = vec[10];  // Should panic
}
```

---

## Build and Test Execution

### Running Tests

**C Version**:
```bash
make
./test_runner
# or
make test
```

**Rust Equivalent**:
```bash
# Run all tests
cargo test

# Run specific test
cargo test test_led_driver

# Run tests in specific module
cargo test led_driver_tests::

# Run with output (even for passing tests)
cargo test -- --nocapture

# Run ignored tests
cargo test -- --ignored

# Run tests with specific number of threads
cargo test -- --test-threads=1

# Run integration tests only
cargo test --test integration_tests

# Run with verbose output
cargo test -- --show-output
```

### Test Organization

**C Version**: Separate test files linked into test executable

**Rust Equivalent**:
```
src/
  lib.rs              # Library code
  led_driver.rs       # LED driver implementation
  
tests/                # Integration tests (separate crate)
  led_driver_test.rs
  integration_test.rs

# Or inline tests
src/
  led_driver.rs:
    // Production code
    pub struct LedDriver { ... }
    
    // Unit tests in same file
    #[cfg(test)]
    mod tests {
        use super::*;
        
        #[test]
        fn test_something() { ... }
    }
```

---

## Dual-Targeting Strategy

### Concept

**Summary**: Dual-targeting means that from day one, your code is designed to run on at least two platforms: the final target hardware and your development system. In the LED driver example from the book, the code is ultimately intended to run on an embedded target, but first it is written and tested on the development system. The goal is not some esoteric or academic pursuit; it is a pragmatic technique to keep development going at a steady pace without depending on scarce or unreliable hardware resources.

**C Version**: Code runs on both development system and embedded target

**Rust Equivalent**: Use target-conditional compilation
```rust
// Hardware abstraction layer
pub trait HardwareAbstraction {
    fn write_register(&mut self, addr: u32, value: u32);
    fn read_register(&self, addr: u32) -> u32;
}

// Real hardware implementation
#[cfg(not(test))]
pub struct RealHardware;

#[cfg(not(test))]
impl HardwareAbstraction for RealHardware {
    fn write_register(&mut self, addr: u32, value: u32) {
        unsafe {
            core::ptr::write_volatile(addr as *mut u32, value);
        }
    }
    
    fn read_register(&self, addr: u32) -> u32 {
        unsafe {
            core::ptr::read_volatile(addr as *const u32)
        }
    }
}

// Test double for development system
#[cfg(test)]
pub struct FakeHardware {
    registers: std::collections::HashMap<u32, u32>,
}

#[cfg(test)]
impl HardwareAbstraction for FakeHardware {
    fn write_register(&mut self, addr: u32, value: u32) {
        self.registers.insert(addr, value);
    }
    
    fn read_register(&self, addr: u32) -> u32 {
        *self.registers.get(&addr).unwrap_or(&0)
    }
}

// Device code works with either
pub struct Device<H: HardwareAbstraction> {
    hardware: H,
}

impl<H: HardwareAbstraction> Device<H> {
    pub fn new(hardware: H) -> Self {
        Self { hardware }
    }
    
    pub fn initialize(&mut self) {
        self.hardware.write_register(0x1000, 0x42);
    }
}
```

---

## Code Under Test (CUT)

### Terminology

**C Version**: Code Under Test (CUT), Device Under Test (DUT), System Under Test (SUT)

**Rust Equivalent**: Same terminology applies
```rust
// The struct/function being tested is the CUT
pub struct LedDriver { }  // <-- CUT

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_led_driver() {  
        // LedDriver is the CUT in this test
        let mut cut = LedDriver::new(0xFF00);
        cut.turn_on(5);
        assert!(cut.is_on(5));
    }
}
```

### Depended-On Component (DOC)

**C Version**: Components that CUT depends on

**Rust Equivalent**:
```rust
// LedDriver depends on IoController (DOC)
pub struct LedDriver<I: IoController> {
    io_controller: I,  // <-- DOC (Depended-On Component)
}

#[cfg(test)]
mod tests {
    #[test]
    fn test_led_driver_with_fake_io() {
        let fake_io = FakeIoController::new();  // Test double for DOC
        let mut driver = LedDriver::new(fake_io);
        // Test CUT in isolation
    }
}
```

---

## Time and Scheduling

### Time Service

**C Version**: Abstract time to enable testing

**Rust Equivalent**:
```rust
use chrono::{DateTime, Utc, Duration};

pub trait TimeService {
    fn get_time(&self) -> DateTime<Utc>;
}

pub struct RealTimeService;

impl TimeService for RealTimeService {
    fn get_time(&self) -> DateTime<Utc> {
        Utc::now()
    }
}

pub struct FakeTimeService {
    current_time: DateTime<Utc>,
}

impl FakeTimeService {
    pub fn new(time: DateTime<Utc>) -> Self {
        Self { current_time: time }
    }
    
    pub fn advance(&mut self, duration: Duration) {
        self.current_time = self.current_time + duration;
    }
}

impl TimeService for FakeTimeService {
    fn get_time(&self) -> DateTime<Utc> {
        self.current_time
    }
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn scheduler_triggers_event_at_scheduled_time() {
        let mut time_service = FakeTimeService::new(
            Utc.ymd(2025, 10, 10).and_hms(14, 0, 0)
        );
        let mut scheduler = Scheduler::new(time_service);
        
        scheduler.schedule_turn_on(7, Utc.ymd(2025, 10, 10).and_hms(18, 0, 0));
        
        time_service.advance(Duration::hours(4));
        scheduler.wake_up();
        
        // Verify event triggered
    }
}
```

### Light Scheduler Example

**C Version**: Event-driven scheduler for controlling lights

**Rust Equivalent**:
```rust
pub struct Event {
    pub id: u32,
    pub led: u8,
    pub action: LightAction,
    pub scheduled_time: DateTime<Utc>,
    pub day: DayOfWeek,
}

pub enum LightAction {
    TurnOn,
    TurnOff,
}

pub enum DayOfWeek {
    Monday, Tuesday, Wednesday, Thursday, Friday, Saturday, Sunday,
}

pub struct LightScheduler<T: TimeService, L: LightController> {
    events: Vec<Event>,
    time_service: T,
    light_controller: L,
}

impl<T: TimeService, L: LightController> LightScheduler<T, L> {
    pub fn new(time_service: T, light_controller: L) -> Self {
        Self {
            events: Vec::new(),
            time_service,
            light_controller,
        }
    }
    
    pub fn schedule_turn_on(&mut self, led: u8, time: DateTime<Utc>) {
        self.events.push(Event {
            id: self.events.len() as u32,
            led,
            action: LightAction::TurnOn,
            scheduled_time: time,
            day: DayOfWeek::Monday,  // Simplified
        });
    }
    
    pub fn wake_up(&mut self) {
        let current_time = self.time_service.get_time();
        
        for event in &self.events {
            if self.should_trigger_event(event, current_time) {
                self.execute_event(event);
            }
        }
    }
    
    fn should_trigger_event(&self, event: &Event, current_time: DateTime<Utc>) -> bool {
        event.scheduled_time <= current_time
    }
    
    fn execute_event(&mut self, event: &Event) {
        match event.action {
            LightAction::TurnOn => self.light_controller.on(event.led),
            LightAction::TurnOff => self.light_controller.off(event.led),
        }
    }
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn scheduler_executes_event_at_scheduled_time() {
        let fake_time = FakeTimeService::new(
            Utc.ymd(2025, 10, 10).and_hms(18, 0, 0)
        );
        let spy_light = SpyLightController::new();
        let mut scheduler = LightScheduler::new(fake_time, spy_light);
        
        scheduler.schedule_turn_on(7, Utc.ymd(2025, 10, 10).and_hms(18, 0, 0));
        scheduler.wake_up();
        
        assert!(spy_light.was_called_with(7, LightAction::On));
    }
}
```

---

## Test Patterns and Anti-Patterns

### Don't Let Code Get Ahead of Tests

**C/Rust Principle**: Only write code required by failing tests

**Example**:
```rust
// ❌ Don't do this
#[test]
fn test_simple_addition() {
    assert_eq!(add(2, 3), 5);
}

fn add(a: i32, b: i32) -> i32 {
    // Don't implement multiplication, division, etc.
    // until tests require it!
    a + b
}

// ✅ Do this - minimal implementation
```

### Test Code Duplication

**C Version**: Use test fixtures to reduce duplication

**Rust Equivalent**:
```rust
// ❌ Duplication
#[test]
fn test_led_1() {
    let mut virtual_leds = 0;
    let mut driver = LedDriver::new(&mut virtual_leds);
    driver.turn_on(1);
    assert!(driver.is_on(1));
}

#[test]
fn test_led_2() {
    let mut virtual_leds = 0;  // Duplicated setup
    let mut driver = LedDriver::new(&mut virtual_leds);  // Duplicated
    driver.turn_on(2);
    assert!(driver.is_on(2));
}

// ✅ Extract fixture
#[test]
fn test_led_1() {
    let (mut driver, _leds) = create_led_driver();
    driver.turn_on(1);
    assert!(driver.is_on(1));
}

#[test]
fn test_led_2() {
    let (mut driver, _leds) = create_led_driver();
    driver.turn_on(2);
    assert!(driver.is_on(2));
}

fn create_led_driver() -> (LedDriver, Box<u16>) {
    let mut virtual_leds = Box::new(0u16);
    let driver = LedDriver::new(&mut *virtual_leds as *mut u16);
    (driver, virtual_leds)
}
```

### Boundary Value Testing

**Summary**: Boundary conditions are where bugs often hide. Testing boundary values means testing the edges of valid input ranges, empty collections, null/None values, maximum and minimum values, and the transitions between valid and invalid inputs. These tests catch off-by-one errors, buffer overflows, and other common programming mistakes. As simple implementations evolve through TDD, seemingly trivial boundary tests become important validators of correct behavior.

**C Version**: Test edge cases, limits, and boundary conditions

**Rust Equivalent**:
```rust
#[test]
fn test_boundary_conditions() {
    let mut buffer = CircularBuffer::new(3);
    
    // Lower boundary
    assert!(buffer.get().is_err());  // Empty
    
    // Fill to capacity
    buffer.put(1).unwrap();
    buffer.put(2).unwrap();
    buffer.put(3).unwrap();
    
    // Upper boundary
    assert!(buffer.put(4).is_err());  // Full
    
    // Valid range
    assert_eq!(buffer.get().unwrap(), 1);
}

#[test]
fn test_led_boundaries() {
    let driver = LedDriver::new(0xFF00);
    
    // Invalid boundaries
    assert!(!driver.is_on(0));   // Below range
    assert!(!driver.is_on(17));  // Above range
    
    // Valid boundaries  
    assert!(driver.turn_on(1).is_ok());   // Min valid
    assert!(driver.turn_on(16).is_ok());  // Max valid
}

#[test]
fn test_integer_boundaries() {
    assert_eq!(safe_add(i32::MAX, 1), Err(OverflowError));
    assert_eq!(safe_add(i32::MIN, -1), Err(OverflowError));
    assert_eq!(safe_add(0, 0), Ok(0));
}
```

---

## Refactoring

**Summary**: Refactoring is the practice of improving code structure without changing its behavior. With a comprehensive test suite from TDD, you can refactor with confidence - the tests will immediately tell you if you break anything. Code that works but is messy should not be left in that state. The discipline of TDD includes a refactoring step after each test passes, ensuring that both test code and production code remain clean and maintainable. Tests are your safety net for refactoring.

### Extract Method

**C Version**: Extract repeated code into functions

**C Example**:
```c
void setup() {
    virtualLeds = 0xFFFF;
    ledDriver = LedDriver_Create(&virtualLeds);
}
```

**Rust Equivalent**:
```rust
// Before refactoring
#[test]
fn test_1() {
    let mut virtual_leds = 0xFFFF;
    let mut driver = LedDriver::new(&mut virtual_leds);
    driver.turn_on(1);
    // ... test code
}

// After refactoring
fn create_initialized_driver() -> (LedDriver, u16) {
    let mut virtual_leds = 0xFFFF;
    let driver = LedDriver::new(&mut virtual_leds);
    (driver, virtual_leds)
}

#[test]
fn test_1() {
    let (mut driver, _leds) = create_initialized_driver();
    driver.turn_on(1);
    // ... test code
}
```

### Clean Code in Tests

**Summary**: Test code deserves the same care and attention as production code. Tests that are hard to understand or maintain become a burden rather than an asset. Apply the same principles of clean code to tests: use descriptive names, avoid duplication through refactoring, keep functions short and focused, and make the intent clear. Well-structured tests are easier to maintain and provide better documentation of system behavior.

**C/Rust Principle**: Test code should be as clean as production code

**Rust Example**:
```rust
// ❌ Unclear test
#[test]
fn test_1() {
    let x = 5;
    let y = process(x);
    assert_eq!(y, 10);
}

// ✅ Clear, expressive test
#[test]
fn process_doubles_input_value() {
    let input = 5;
    let expected_output = 10;
    
    let actual_output = process(input);
    
    assert_eq!(actual_output, expected_output);
}
```

---

## Hardware Abstraction Layer (HAL)

### Concept

**Summary**: To successfully use TDD in embedded development, you must isolate hardware dependencies. The more successful you are at isolating hardware dependencies, the longer the useful life you can expect from your code and your tests. If you let hardware dependencies permeate the code, hardware evolution (and obsolescence) will accelerate the aging of your code and shorten its useful life. The Hardware Abstraction Layer provides defined interfaces that separate hardware-specific code from business logic, enabling testing without physical hardware.

**C Version**: Isolate hardware-specific code behind an interface

**Rust Equivalent**:
```rust
// Hardware abstraction trait
pub trait GpioPin {
    fn set_high(&mut self);
    fn set_low(&mut self);
    fn is_high(&self) -> bool;
}

// Real hardware implementation (for embedded target)
#[cfg(target_arch = "arm")]
pub struct HardwareGpioPin {
    register: *mut u32,
    pin_mask: u32,
}

#[cfg(target_arch = "arm")]
impl GpioPin for HardwareGpioPin {
    fn set_high(&mut self) {
        unsafe {
            core::ptr::write_volatile(self.register, 
                core::ptr::read_volatile(self.register) | self.pin_mask);
        }
    }
    
    fn set_low(&mut self) {
        unsafe {
            core::ptr::write_volatile(self.register,
                core::ptr::read_volatile(self.register) & !self.pin_mask);
        }
    }
    
    fn is_high(&self) -> bool {
        unsafe {
            (core::ptr::read_volatile(self.register) & self.pin_mask) != 0
        }
    }
}

// Fake implementation for testing
pub struct FakeGpioPin {
    state: std::cell::Cell<bool>,
}

impl FakeGpioPin {
    pub fn new() -> Self {
        Self { state: std::cell::Cell::new(false) }
    }
}

impl GpioPin for FakeGpioPin {
    fn set_high(&mut self) {
        self.state.set(true);
    }
    
    fn set_low(&mut self) {
        self.state.set(false);
    }
    
    fn is_high(&self) -> bool {
        self.state.get()
    }
}

// Business logic works with abstraction
pub struct LedController<P: GpioPin> {
    pin: P,
}

impl<P: GpioPin> LedController<P> {
    pub fn new(pin: P) -> Self {
        Self { pin }
    }
    
    pub fn turn_on(&mut self) {
        self.pin.set_high();
    }
    
    pub fn is_on(&self) -> bool {
        self.pin.is_high()
    }
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn controller_turns_on_led() {
        let fake_pin = FakeGpioPin::new();
        let mut controller = LedController::new(fake_pin);
        
        controller.turn_on();
        
        assert!(controller.is_on());
    }
}
```

---

## Continuous Integration (CI)

### Concept

**C Version**: Automated build and test execution on every commit

**Rust Equivalent** (GitHub Actions example):
```yaml
# .github/workflows/ci.yml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions-rs/toolchain@v1
        with:
          toolchain: stable
      - name: Run tests
        run: cargo test --all-features
      - name: Run clippy
        run: cargo clippy -- -D warnings
      - name: Check formatting
        run: cargo fmt -- --check
```

---

## Test Execution Modes

**Summary**: Test harnesses are normally quiet except for test failures - when all tests pass, the output is minimal. In the Unix style, the test harness follows the "no news is good news" principle. However, when debugging test failures or understanding test execution, verbose modes allow you to see each test as it announces itself before running. This is particularly useful when a test crashes - the last test mentioned is the one that crashed.

### Verbose Mode

**C Version**: `./test_runner -v` shows each test as it runs

**Rust Equivalent**:
```bash
cargo test -- --nocapture    # Show println! output
cargo test -- --show-output  # Show output even for passing tests
RUST_TEST_THREADS=1 cargo test  # Run tests serially
```

### Filtering Tests

**C Version**: `-g testgroup` or `-n testname`

**Rust Equivalent**:
```bash
cargo test led_driver          # Run tests matching "led_driver"
cargo test --test integration  # Run specific integration test file
cargo test -- --ignored        # Run ignored tests only
cargo test -- --include-ignored # Run all tests including ignored
```

---

## Test Naming Conventions

### Descriptive Names

**C Version**: `TEST(LedDriver, TurnOnLedOne)` - describes what and expected outcome

**Rust Equivalent**:
```rust
// ✅ Good: Descriptive, explains behavior
#[test]
fn led_is_on_after_turn_on_called() { }

#[test]
fn led_is_off_after_turn_off_called() { }

#[test]
fn out_of_bounds_led_numbers_return_error() { }

#[test]
fn scheduler_triggers_event_at_exact_scheduled_time() { }

// ❌ Bad: Vague, unclear intent
#[test]
fn test1() { }

#[test]
fn check_led() { }

#[test]
fn it_works() { }
```

---

## Error Handling in Tests

### Testing Error Conditions

**C Version**: Test error codes and invalid inputs

**Rust Equivalent**:
```rust
#[test]
fn invalid_led_number_returns_error() {
    let mut driver = LedDriver::new(0xFF00);
    
    assert!(matches!(
        driver.turn_on(0),
        Err(LedError::InvalidLedNumber)
    ));
    
    assert!(matches!(
        driver.turn_on(17),
        Err(LedError::InvalidLedNumber)
    ));
}

#[test]
fn buffer_full_error_when_capacity_exceeded() {
    let mut buffer = CircularBuffer::new(2);
    buffer.put(1).unwrap();
    buffer.put(2).unwrap();
    
    let result = buffer.put(3);
    assert!(result.is_err());
    assert_eq!(result.unwrap_err(), BufferError::Full);
}

#[test]
fn buffer_empty_error_when_reading_empty_buffer() {
    let mut buffer: CircularBuffer<i32> = CircularBuffer::new(10);
    
    let result = buffer.get();
    assert!(matches!(result, Err(BufferError::Empty)));
}
```

---

## Design for Testability

**Summary**: A good design is a testable design. Long functions, tight coupling, and complex conditionals all lead to more complex and less testable code. TDD naturally guides design by encouraging loose coupling and high cohesion - if code is hard to test, it's probably poorly designed. By writing tests first, you're forced to think about interfaces, dependencies, and how components will interact, leading to cleaner, more modular designs. The testability problems you encounter while writing tests are signals of design problems that should be addressed.

### Tight Coupling (Anti-Pattern)

**C Version**: Direct hardware dependencies make testing difficult

**Rust Equivalent**:
```rust
// ❌ Tight coupling - hard to test
pub struct LedDriver {
    // Direct hardware access
}

impl LedDriver {
    pub fn turn_on(&mut self, led: u8) {
        unsafe {
            // Direct register access
            let register = 0xFF00 as *mut u16;
            *register |= 1 << led;
        }
    }
}

// ✅ Loose coupling - easy to test
pub struct LedDriver<H: HardwareAccess> {
    hardware: H,
}

pub trait HardwareAccess {
    fn write_leds(&mut self, value: u16);
    fn read_leds(&self) -> u16;
}

impl<H: HardwareAccess> LedDriver<H> {
    pub fn new(hardware: H) -> Self {
        Self { hardware }
    }
    
    pub fn turn_on(&mut self, led: u8) {
        let current = self.hardware.read_leds();
        self.hardware.write_leds(current | (1 << led));
    }
}

// Easy to test with fake
#[cfg(test)]
mod tests {
    #[test]
    fn test_turn_on() {
        let fake_hw = FakeHardware::new();
        let mut driver = LedDriver::new(fake_hw);
        driver.turn_on(5);
        // Test passes without real hardware
    }
}
```

---

## Integration vs Unit Tests

### Unit Tests

**C Version**: Test single module in isolation

**Rust Equivalent**:
```rust
// In src/led_driver.rs
pub struct LedDriver { }

#[cfg(test)]
mod tests {
    use super::*;
    
    // Unit test - tests only LedDriver in isolation
    #[test]
    fn led_driver_stores_state() {
        let mut driver = LedDriver::new(0xFF00);
        driver.turn_on(5);
        assert!(driver.is_on(5));
    }
}
```

### Integration Tests

**C Version**: Test multiple modules working together

**Rust Equivalent**:
```rust
// In tests/integration_test.rs
use my_crate::*;

#[test]
fn led_scheduler_integration() {
    // Integration test - multiple components together
    let time_service = RealTimeService;
    let light_controller = RealLightController::new();
    let mut scheduler = LightScheduler::new(time_service, light_controller);
    
    scheduler.schedule_turn_on(7, future_time());
    scheduler.wake_up();
    
    // Verify end-to-end behavior
}
```

---

## Testability Metrics

### Code Coverage

**C/Rust Tool**: Measure which code is exercised by tests

**Rust Equivalent**:
```bash
# Using tarpaulin for coverage
cargo install cargo-tarpaulin
cargo tarpaulin --out Html

# Using llvm-cov
cargo install cargo-llvm-cov
cargo llvm-cov --html

# View coverage report
# Note: TDD typically achieves high coverage naturally
```

### Coverage is Not the Goal

**Principle**: TDD provides right coverage as by-product, not the goal itself

**Focus on**:
- Behavior coverage (all scenarios tested)
- Branch coverage (all paths tested)  
- Boundary coverage (edge cases tested)

---

## Documentation Through Tests

### Living Documentation

**C/Rust Concept**: Tests serve as executable specifications

**Rust Equivalent**:
```rust
/// Circular buffer implementation with fixed capacity
pub struct CircularBuffer<T> { }

#[cfg(test)]
mod tests {
    use super::*;
    
    /// Documents that buffer starts empty
    #[test]
    fn new_buffer_is_empty() {
        let buffer: CircularBuffer<i32> = CircularBuffer::new(10);
        assert!(buffer.is_empty());
    }
    
    /// Documents FIFO behavior
    #[test]
    fn buffer_returns_items_in_fifo_order() {
        let mut buffer = CircularBuffer::new(3);
        buffer.put(1).unwrap();
        buffer.put(2).unwrap();
        buffer.put(3).unwrap();
        
        assert_eq!(buffer.get().unwrap(), 1);  // First in
        assert_eq!(buffer.get().unwrap(), 2);  // ...
        assert_eq!(buffer.get().unwrap(), 3);  // Last in
    }
    
    /// Documents error behavior when full
    #[test]
    fn buffer_returns_error_when_full() {
        let mut buffer = CircularBuffer::new(1);
        buffer.put(42).unwrap();
        
        assert!(matches!(buffer.put(99), Err(BufferError::Full)));
    }
}
```

---

## State Machine Testing

**C Version**: Test state transitions and guards

**Rust Equivalent**:
```rust
#[derive(Debug, PartialEq)]
pub enum LedState {
    Off,
    On,
    Blinking,
}

pub struct StatefulLed {
    state: LedState,
}

impl StatefulLed {
    pub fn new() -> Self {
        Self { state: LedState::Off }
    }
    
    pub fn turn_on(&mut self) -> Result<(), Error> {
        match self.state {
            LedState::Off => {
                self.state = LedState::On;
                Ok(())
            }
            LedState::On => Ok(()),  // Already on
            LedState::Blinking => Err(Error::InvalidStateTransition),
        }
    }
    
    pub fn start_blinking(&mut self) -> Result<(), Error> {
        match self.state {
            LedState::On => {
                self.state = LedState::Blinking;
                Ok(())
            }
            _ => Err(Error::MustBeOnToStartBlinking),
        }
    }
    
    pub fn get_state(&self) -> &LedState {
        &self.state
    }
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn led_starts_in_off_state() {
        let led = StatefulLed::new();
        assert_eq!(led.get_state(), &LedState::Off);
    }
    
    #[test]
    fn led_transitions_from_off_to_on() {
        let mut led = StatefulLed::new();
        led.turn_on().unwrap();
        assert_eq!(led.get_state(), &LedState::On);
    }
    
    #[test]
    fn led_cannot_blink_unless_on() {
        let mut led = StatefulLed::new();
        assert!(led.start_blinking().is_err());
    }
    
    #[test]
    fn led_can_blink_when_on() {
        let mut led = StatefulLed::new();
        led.turn_on().unwrap();
        led.start_blinking().unwrap();
        assert_eq!(led.get_state(), &LedState::Blinking);
    }
}
```

---

## Test-Only APIs

**C Version**: Functions or data available only in test builds

**Rust Equivalent**:
```rust
pub struct Component {
    internal_state: u32,
}

impl Component {
    pub fn new() -> Self {
        Self { internal_state: 0 }
    }
    
    pub fn do_something(&mut self) {
        self.internal_state += 1;
    }
    
    // Test-only API
    #[cfg(test)]
    pub fn get_internal_state(&self) -> u32 {
        self.internal_state
    }
    
    // Or using doc(hidden) for public but discouraged use
    #[doc(hidden)]
    pub fn __test_get_state(&self) -> u32 {
        self.internal_state
    }
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn internal_state_increments() {
        let mut component = Component::new();
        component.do_something();
        
        // Can access test-only API
        assert_eq!(component.get_internal_state(), 1);
    }
}
```

---

## Benefits of TDD

### Well-Structured Tests as Documentation

**Summary**: Well-structured tests become a form of executable and unambiguous documentation. A working example is worth 1,000 words. Unlike written documentation that can become outdated, tests must stay current or they fail. They document not just what the code should do, but exactly how to use it, what inputs are valid, what errors can occur, and what the expected outputs are. Tests serve as living specifications that are always up-to-date and verified.

**Principle**: Tests document behavior better than comments

**Rust Example**:
```rust
/// CircularBuffer wraps around when full
#[cfg(test)]
mod buffer_wrapping_behavior {
    use super::*;
    
    /// When buffer is full and circular, oldest item is overwritten
    #[test]
    fn buffer_with_wrap_overwrites_oldest() {
        let mut buffer = CircularBuffer::with_wrapping(3);
        
        buffer.put(1).unwrap();
        buffer.put(2).unwrap();
        buffer.put(3).unwrap();
        buffer.put(4).unwrap();  // Overwrites 1
        
        assert_eq!(buffer.get().unwrap(), 2);
        assert_eq!(buffer.get().unwrap(), 3);
        assert_eq!(buffer.get().unwrap(), 4);
    }
}
```

### Improved Design

**Principle**: Testable code is better designed (loose coupling, high cohesion)

**Example**:
```rust
// TDD naturally leads to this design
pub struct Scheduler<T, L> 
where
    T: TimeService,
    L: LightController,
{
    time_service: T,      // Dependency injected
    light_controller: L,  // Dependency injected
    events: Vec<Event>,
}

// Rather than this
pub struct Scheduler {
    // Hard-coded dependencies - difficult to test
    // internal SystemTime::now() calls
    // internal GPIO register access
}
```

### Peace of Mind

**Principle**: Comprehensive test suite provides confidence

**Rust Benefits**:
- `cargo test` before commit
- Automated CI runs tests
- Regression suite catches breakage
- Refactoring with confidence

---

## Smell: Long Functions

**C/Rust Anti-Pattern**: Functions longer than ~20 lines are hard to test

**Refactoring**:
```rust
// ❌ Long, hard to test
pub fn process_data(data: &[u8]) -> Result<Vec<u8>, Error> {
    // 50 lines of complex logic
    // Multiple responsibilities
    // Hard to test each path
}

// ✅ Extracted, easy to test
pub fn process_data(data: &[u8]) -> Result<Vec<u8>, Error> {
    let validated = validate_input(data)?;
    let transformed = transform_data(validated)?;
    let finalized = finalize_data(transformed)?;
    Ok(finalized)
}

fn validate_input(data: &[u8]) -> Result<&[u8], Error> {
    // Testable in isolation
}

fn transform_data(data: &[u8]) -> Result<Vec<u8>, Error> {
    // Testable in isolation
}

fn finalize_data(data: Vec<u8>) -> Result<Vec<u8>, Error> {
    // Testable in isolation
}

#[cfg(test)]
mod tests {
    // Can now test each function independently
    #[test]
    fn validate_input_rejects_empty() { }
    
    #[test]
    fn transform_doubles_values() { }
    
    #[test]
    fn finalize_adds_checksum() { }
}
```

---

## Property-Based Testing

**C Version**: Not commonly used in embedded C

**Rust Enhancement** (using `proptest`):
```rust
use proptest::prelude::*;

proptest! {
    #[test]
    fn circular_buffer_never_exceeds_capacity(
        values in prop::collection::vec(any::<i32>(), 0..100),
        capacity in 1usize..50
    ) {
        let mut buffer = CircularBuffer::new(capacity);
        
        for value in values {
            let _ = buffer.put(value);  // May fail when full
            
            // Invariant: count never exceeds capacity
            prop_assert!(buffer.count() <= capacity);
        }
    }
    
    #[test]
    fn led_state_is_consistent(
        led_number in 1u8..=16,
        operations in prop::collection::vec(any::<bool>(), 0..100)
    ) {
        let mut driver = LedDriver::new(0xFF00);
        
        for &should_turn_on in &operations {
            if should_turn_on {
                driver.turn_on(led_number).unwrap();
            } else {
                driver.turn_off(led_number).unwrap();
            }
        }
        
        // Verify final state matches last operation
        let expected = operations.last().copied().unwrap_or(false);
        prop_assert_eq!(driver.is_on(led_number), expected);
    }
}
```

---

## Slow Down to Go Fast

**Summary**: Using Test-Driven Development for embedded software has its challenges, and in spite of those challenges, putting TDD to work in the embedded environment is worth the effort. When you test after (Debug-Later Programming), you might find many of your mistakes, but some will escape detection where TDD would have found them immediately. TDD is more rigorous and provides better test coverage - the right test coverage. When you write tests after, you also have to spend valuable time hunting down the root cause of test failures, while in TDD the root cause is usually obvious (it's the code you just wrote). The initial investment in TDD discipline pays dividends through reduced debugging time, fewer production bugs, and more maintainable code.

**Principle**: TDD may feel slower initially but leads to faster overall development

**Benefits**:
1. **Fewer bugs**: Caught immediately rather than in QA/production
2. **Easier debugging**: Problem is obvious (just-written code)
3. **Better design**: Forces thinking about interfaces
4. **Fearless refactoring**: Tests catch regressions
5. **Better documentation**: Tests show usage examples

**Rust-Specific Benefits**:
- Compiler catches many errors TDD would in other languages
- TDD + Rust's type system = extremely robust code
- Tests document lifetime and ownership expectations

---

## Test Count Expectations

### 0-1-N Pattern

**Summary**: When testing collections or repeating behavior, follow the 0-1-N pattern. First test the zero case (nothing in the collection, no events scheduled), then test the one case (single item, single event), and finally test the N case (multiple items, multiple events). This pattern ensures you handle empty states correctly, get the basic behavior right with one item, and properly handle multiple items including any interactions between them.

**C Concept**: Test zero cases, one case, then many cases

**Rust Example**:
```rust
mod scheduler_events {
    // 0 case
    #[test]
    fn no_events_scheduled_does_nothing() {
        let scheduler = Scheduler::new();
        scheduler.wake_up();
        // Nothing happens, no panics
    }
    
    // 1 case
    #[test]
    fn one_event_triggers_correctly() {
        let mut scheduler = Scheduler::new();
        scheduler.schedule_turn_on(7, time());
        scheduler.wake_up();
        // Verify event triggered
    }
    
    // N cases
    #[test]
    fn multiple_events_all_trigger() {
        let mut scheduler = Scheduler::new();
        scheduler.schedule_turn_on(1, time());
        scheduler.schedule_turn_on(2, time());
        scheduler.schedule_turn_on(3, time());
        scheduler.wake_up();
        // Verify all events triggered
    }
}
```

---

## Refactoring Safety Net

**Concept**: Tests enable safe refactoring

**Rust Example**:
```rust
// Original implementation
pub fn calculate_total(items: &[Item]) -> f64 {
    let mut total = 0.0;
    for item in items {
        total += item.price * item.quantity as f64;
    }
    total
}

#[test]
fn test_calculate_total() {
    let items = vec![
        Item { price: 10.0, quantity: 2 },
        Item { price: 5.0, quantity: 3 },
    ];
    assert_eq!(calculate_total(&items), 35.0);
}

// Refactor with confidence - test will catch breakage
pub fn calculate_total(items: &[Item]) -> f64 {
    items.iter()
        .map(|item| item.price * item.quantity as f64)
        .sum()
}

// Run test - still passes, refactoring successful!
```

---

## Conclusion and Best Practices

### Key Takeaways for Rust TDD

1. **Use Cargo Test Framework**: Built-in, powerful, zero setup
2. **Leverage Type System**: Many C tests unnecessary due to Rust's type safety
3. **Use Traits for Abstraction**: Better than C function pointers
4. **Embrace Result<T, E>**: Test both Ok and Err cases
5. **Use Feature Flags**: Better than C preprocessor for platform-specific code
6. **Mock with mockall**: Powerful mocking for complex interactions
7. **Property-Based Testing**: Rust has excellent support via proptest
8. **Integration Tests**: Use `tests/` directory for end-to-end tests
9. **Test Documentation**: Use `cargo test --doc` for doc tests
10. **CI/CD**: Automate testing with GitHub Actions or similar

### Rust-Specific Advantages

- **Ownership & Lifetimes**: Compiler catches many bugs tests would catch in C
- **Option<T> & Result<T,E>**: Explicit error handling reduces defensive testing
- **Traits**: More flexible than C interfaces
- **No UB in Safe Code**: Fewer edge cases to test
- **Package Manager**: Easy to add test dependencies (mockall, proptest, etc.)
- **Built-in Benchmarking**: Performance testing integrated

### The TDD Mindset

**Universal to C and Rust**:
1. Write test first
2. Watch it fail
3. Make it pass
4. Refactor
5. Repeat

**Red → Green → Refactor**

---

## Quick Reference

### C to Rust Test Translation Table

| C Concept | C Syntax | Rust Equivalent |
|-----------|----------|-----------------|
| Test Group | `TEST_GROUP(Name)` | `mod name_tests { }` |
| Test Case | `TEST(Group, Name)` | `#[test] fn name()` |
| Setup | `TEST_SETUP(Group)` | Helper function or fixture struct |
| Teardown | `TEST_TEAR_DOWN(Group)` | `Drop` trait implementation |
| Assert Equal | `TEST_ASSERT_EQUAL(e, a)` | `assert_eq!(a, e)` |
| Assert True | `TEST_ASSERT_TRUE(c)` | `assert!(c)` |
| Assert Null | `TEST_ASSERT_NULL(p)` | `assert!(opt.is_none())` |
| Mock Object | CppUMock | `mockall` crate |
| Function Pointer | `typedef int (*Fn)()` | `type Fn = fn() -> i32` or trait |
| Preprocessor | `#ifdef TEST` | `#[cfg(test)]` or features |
| Link Substitution | Linker flags | Trait objects or `#[cfg]` |
| Test Runner | Custom main | `cargo test` |

---

**End of Lexicon**

This lexicon provides a comprehensive mapping of TDD concepts from C to Rust, enabling developers familiar with "Test Driven Development for Embedded C" to apply the same principles effectively in Rust projects.
