# Swift Testing Framework

> Apple's modern testing framework replacing XCTest with better syntax and performance

## 🧪 Introduction to Swift Testing

Swift Testing is Apple's next-generation testing framework that provides:
- **Cleaner syntax** with `@Test` attributes
- **Better error reporting** with detailed diagnostics
- **Improved performance** with parallel execution
- **Enhanced debugging** capabilities

## 🚀 Getting Started

### Basic Test Structure
```swift
import Testing

@Test("Basic arithmetic operations")
func testArithmetic() {
    let result = 2 + 2
    #expect(result == 4)
}

@Test("String manipulation")
func testStringOperations() {
    let text = "Hello, World!"
    #expect(text.contains("World"))
    #expect(text.count == 13)
}
```

### Parameterized Tests
```swift
@Test("Factorial calculation", arguments: [
    (0, 1),
    (1, 1),
    (5, 120),
    (10, 3628800)
])
func testFactorial(input: Int, expected: Int) {
    let result = factorial(input)
    #expect(result == expected)
}
```

## 🔧 Advanced Features

### Async Testing
```swift
@Test("Async network request")
func testNetworkRequest() async throws {
    let url = URL(string: "https://api.example.com/data")!
    let (data, response) = try await URLSession.shared.data(from: url)
    
    #expect(data.count > 0)
    #expect((response as? HTTPURLResponse)?.statusCode == 200)
}
```

### Error Testing
```swift
@Test("Error handling")
func testErrorHandling() {
    #expect(throws: ValidationError.self) {
        try validateEmail("invalid-email")
    }
    
    #expect(throws: Never.self) {
        try validateEmail("valid@example.com")
    }
}
```

### Conditional Tests
```swift
@Test("iOS specific feature", .enabled(if: ProcessInfo.processInfo.isiOSAppOnMac == false))
func testIOSFeature() {
    // Test only runs on actual iOS devices
    let feature = IOSSpecificFeature()
    #expect(feature.isAvailable == true)
}
```

## 📱 SwiftUI Testing

### View Testing
```swift
import Testing
import SwiftUI

@Test("ContentView displays correctly")
@MainActor
func testContentView() {
    let view = ContentView()
    let hosting = UIHostingController(rootView: view)
    
    #expect(hosting.view != nil)
    // Additional view testing logic
}
```

### State Testing
```swift
@Test("State management")
@MainActor
func testStateChanges() {
    @State var counter = 0
    
    let view = CounterView(counter: $counter)
    
    // Simulate user interaction
    counter += 1
    
    #expect(counter == 1)
}
```

## 🎯 Testing Patterns

### Dependency Injection Testing
```swift
protocol NetworkService {
    func fetchData() async throws -> Data
}

class MockNetworkService: NetworkService {
    var shouldFail = false
    
    func fetchData() async throws -> Data {
        if shouldFail {
            throw NetworkError.connectionFailed
        }
        return Data("mock data".utf8)
    }
}

@Test("Service with dependency injection")
func testServiceWithMocking() async throws {
    let mockService = MockNetworkService()
    let dataManager = DataManager(networkService: mockService)
    
    let data = try await dataManager.loadData()
    #expect(data.count > 0)
    
    // Test error case
    mockService.shouldFail = true
    #expect(throws: NetworkError.self) {
        try await dataManager.loadData()
    }
}
```

### Core Data Testing
```swift
@Test("Core Data operations")
func testCoreDataOperations() throws {
    let container = NSPersistentContainer.inMemory()
    let context = container.viewContext
    
    // Create test entity
    let entity = TestEntity(context: context)
    entity.name = "Test"
    
    try context.save()
    
    // Verify entity was saved
    let request: NSFetchRequest<TestEntity> = TestEntity.fetchRequest()
    let results = try context.fetch(request)
    
    #expect(results.count == 1)
    #expect(results.first?.name == "Test")
}
```

## 🔍 Test Organization

### Test Suites
```swift
@Suite("User Authentication Tests")
struct AuthenticationTests {
    
    @Test("Valid login")
    func testValidLogin() async throws {
        let auth = AuthenticationService()
        let result = try await auth.login(email: "test@example.com", password: "password")
        #expect(result.isSuccess == true)
    }
    
    @Test("Invalid credentials")
    func testInvalidLogin() async throws {
        let auth = AuthenticationService()
        #expect(throws: AuthError.invalidCredentials) {
            try await auth.login(email: "test@example.com", password: "wrong")
        }
    }
}
```

### Setup and Teardown
```swift
@Suite("Database Tests")
struct DatabaseTests {
    let database: TestDatabase
    
    init() throws {
        database = try TestDatabase.createInMemory()
    }
    
    deinit {
        database.cleanup()
    }
    
    @Test("Insert operation")
    func testInsert() throws {
        try database.insert(TestRecord(id: 1, name: "Test"))
        let record = try database.fetch(id: 1)
        #expect(record?.name == "Test")
    }
}
```

## 📊 Performance Testing

### Timing Tests
```swift
@Test("Performance benchmark")
func testPerformance() {
    let startTime = CFAbsoluteTimeGetCurrent()
    
    // Perform operation
    let result = heavyComputation()
    
    let timeElapsed = CFAbsoluteTimeGetCurrent() - startTime
    
    #expect(timeElapsed < 1.0) // Should complete within 1 second
    #expect(result.isValid == true)
}
```

### Memory Testing
```swift
@Test("Memory usage")
func testMemoryUsage() {
    let initialMemory = getMemoryUsage()
    
    // Perform memory-intensive operation
    let largeArray = Array(0..<1_000_000)
    
    let peakMemory = getMemoryUsage()
    
    // Clean up
    _ = largeArray // Keep reference until measurement
    
    #expect(peakMemory - initialMemory < 50_000_000) // Less than 50MB increase
}
```

## 🛠 Integration with Xcode

### Test Plans
```json
{
  "configurations": [
    {
      "name": "Unit Tests",
      "testTargets": [
        {
          "target": {
            "containerPath": "MyApp.xcodeproj",
            "identifier": "MyAppTests",
            "name": "MyAppTests"
          }
        }
      ]
    }
  ],
  "defaultOptions": {
    "testTimeoutsEnabled": true,
    "maximumTestExecutionTimeAllowance": 60
  }
}
```

### Continuous Integration
```yaml
# GitHub Actions example
name: Swift Testing
on: [push, pull_request]

jobs:
  test:
    runs-on: macos-latest
    steps:
    - uses: actions/checkout@v4
    - name: Run Swift Tests
      run: |
        xcodebuild test \
          -scheme MyApp \
          -destination 'platform=iOS Simulator,name=iPhone 15 Pro' \
          -testPlan UnitTests
```

## 🎓 Best Practices

### 1. Descriptive Test Names
```swift
@Test("User can successfully create account with valid email and strong password")
func testUserAccountCreationWithValidCredentials() {
    // Test implementation
}
```

### 2. Arrange-Act-Assert Pattern
```swift
@Test("Shopping cart calculates total correctly")
func testShoppingCartTotal() {
    // Arrange
    let cart = ShoppingCart()
    let item1 = Product(name: "Item 1", price: 10.00)
    let item2 = Product(name: "Item 2", price: 15.50)
    
    // Act
    cart.add(item1)
    cart.add(item2)
    let total = cart.calculateTotal()
    
    // Assert
    #expect(total == 25.50)
}
```

### 3. Test Data Builders
```swift
struct UserBuilder {
    private var user = User()
    
    func withName(_ name: String) -> UserBuilder {
        var builder = self
        builder.user.name = name
        return builder
    }
    
    func withEmail(_ email: String) -> UserBuilder {
        var builder = self
        builder.user.email = email
        return builder
    }
    
    func build() -> User {
        return user
    }
}

@Test("User validation")
func testUserValidation() {
    let user = UserBuilder()
        .withName("John Doe")
        .withEmail("john@example.com")
        .build()
    
    #expect(user.isValid == true)
}
```

## 📚 Migration from XCTest

### XCTest vs Swift Testing
```swift
// XCTest (old)
class MyTests: XCTestCase {
    func testExample() {
        XCTAssertEqual(2 + 2, 4)
    }
}

// Swift Testing (new)
@Test("Example test")
func testExample() {
    #expect(2 + 2 == 4)
}
```

### Assertion Migration
```swift
// XCTest assertions -> Swift Testing expectations
XCTAssertEqual(a, b)           -> #expect(a == b)
XCTAssertTrue(condition)       -> #expect(condition)
XCTAssertNil(value)           -> #expect(value == nil)
XCTAssertThrowsError(try f()) -> #expect(throws: Error.self) { try f() }
```

---

*Swift Testing provides a modern, efficient way to test your Swift applications with improved syntax and better tooling integration.*
