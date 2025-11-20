# Advanced Swift Patterns for 2026

> Production-ready patterns for modern Swift development

## 🎯 Result Builders

Result builders enable DSL-like syntax for constructing complex values.

### SwiftUI-Style Builders

```swift
@resultBuilder
struct HTMLBuilder {
    static func buildBlock(_ components: String...) -> String {
        components.joined()
    }
    
    static func buildOptional(_ component: String?) -> String {
        component ?? ""
    }
    
    static func buildEither(first component: String) -> String {
        component
    }
    
    static func buildEither(second component: String) -> String {
        component
    }
}

func html(@HTMLBuilder content: () -> String) -> String {
    "<html>\(content())</html>"
}

// Usage
let page = html {
    "<head><title>My Page</title></head>"
    "<body>"
    "<h1>Welcome</h1>"
    "</body>"
}
```

### Custom View Builder

```swift
@resultBuilder
struct ViewBuilder {
    static func buildBlock<Content: View>(_ content: Content) -> Content {
        content
    }
    
    static func buildBlock<C0: View, C1: View>(_ c0: C0, _ c1: C1) -> TupleView<(C0, C1)> {
        TupleView((c0, c1))
    }
}

struct CustomContainer<Content: View>: View {
    let content: Content
    
    init(@ViewBuilder content: () -> Content) {
        self.content = content()
    }
    
    var body: some View {
        content
    }
}
```

## 🔄 Property Wrappers

### Thread-Safe Property Wrapper

```swift
@propertyWrapper
struct Atomic<Value> {
    private var value: Value
    private let lock = NSLock()
    
    var wrappedValue: Value {
        get {
            lock.lock()
            defer { lock.unlock() }
            return value
        }
        set {
            lock.lock()
            defer { lock.unlock() }
            value = newValue
        }
    }
    
    init(wrappedValue: Value) {
        self.value = wrappedValue
    }
}

// Usage
class Counter {
    @Atomic var count = 0
    
    func increment() {
        count += 1  // Thread-safe
    }
}
```

### UserDefaults Property Wrapper

```swift
@propertyWrapper
struct UserDefault<T> {
    let key: String
    let defaultValue: T
    let storage: UserDefaults
    
    var wrappedValue: T {
        get {
            storage.object(forKey: key) as? T ?? defaultValue
        }
        set {
            storage.set(newValue, forKey: key)
        }
    }
    
    var projectedValue: Binding<T> {
        Binding(
            get: { wrappedValue },
            set: { wrappedValue = $0 }
        )
    }
    
    init(wrappedValue: T, _ key: String, storage: UserDefaults = .standard) {
        self.key = key
        self.defaultValue = wrappedValue
        self.storage = storage
    }
}

// Usage
struct Settings {
    @UserDefault("username", storage: .standard)
    var username: String = "Guest"
    
    @UserDefault("isDarkMode", storage: .standard)
    var isDarkMode: Bool = false
}
```

## 🎭 Type Erasure

### AnyPublisher Pattern

```swift
protocol DataProvider {
    associatedtype Output
    func fetch() -> Output
}

// Type erasure wrapper
struct AnyDataProvider<Output>: DataProvider {
    private let _fetch: () -> Output
    
    init<P: DataProvider>(_ provider: P) where P.Output == Output {
        _fetch = provider.fetch
    }
    
    func fetch() -> Output {
        _fetch()
    }
}

// Usage
struct UserProvider: DataProvider {
    func fetch() -> User {
        User(id: "1", name: "Alice")
    }
}

let provider: AnyDataProvider<User> = AnyDataProvider(UserProvider())
```

## 🔐 Phantom Types

Phantom types add compile-time safety without runtime overhead.

```swift
enum Validated {}
enum Unvalidated {}

struct Email<State> {
    let value: String
    
    private init(_ value: String) {
        self.value = value
    }
}

extension Email where State == Unvalidated {
    init(raw: String) {
        self.init(raw)
    }
    
    func validated() -> Email<Validated>? {
        guard value.contains("@"), value.contains(".") else {
            return nil
        }
        return Email<Validated>(value)
    }
}

extension Email where State == Validated {
    func send(message: String) {
        print("Sending to \(value): \(message)")
    }
}

// Usage
let email = Email<Unvalidated>(raw: "test@example.com")
if let validated = email.validated() {
    validated.send(message: "Hello!")  // ✅ Type-safe
}

// let invalid = Email<Unvalidated>(raw: "invalid")
// invalid.send(message: "Hi")  // ❌ Compile error - can't send unvalidated
```

## 🎯 KeyPath Magic

### Dynamic Member Lookup

```swift
@dynamicMemberLookup
struct Settings {
    private var storage: [String: Any] = [:]
    
    subscript<T>(dynamicMember key: String) -> T? {
        get { storage[key] as? T }
        set { storage[key] = newValue }
    }
}

var settings = Settings()
settings.apiKey = "abc123"
settings.timeout = 30
let key: String? = settings.apiKey
```

### KeyPath Sorting

```swift
extension Sequence {
    func sorted<T: Comparable>(by keyPath: KeyPath<Element, T>) -> [Element] {
        sorted { $0[keyPath: keyPath] < $1[keyPath: keyPath] }
    }
}

struct User {
    let name: String
    let age: Int
}

let users = [
    User(name: "Alice", age: 30),
    User(name: "Bob", age: 25)
]

let sortedByAge = users.sorted(by: \.age)
let sortedByName = users.sorted(by: \.name)
```

## 🚀 Async Sequences

### Custom AsyncSequence

```swift
struct CountdownSequence: AsyncSequence {
    typealias Element = Int
    
    let start: Int
    let delay: Duration
    
    struct AsyncIterator: AsyncIteratorProtocol {
        var current: Int
        let delay: Duration
        
        mutating func next() async -> Int? {
            guard current > 0 else { return nil }
            try? await Task.sleep(for: delay)
            defer { current -= 1 }
            return current
        }
    }
    
    func makeAsyncIterator() -> AsyncIterator {
        AsyncIterator(current: start, delay: delay)
    }
}

// Usage
for await count in CountdownSequence(start: 5, delay: .seconds(1)) {
    print(count)  // 5, 4, 3, 2, 1
}
```

## 🎨 Protocol Witnesses

Replace protocols with concrete types for better performance.

```swift
// Traditional protocol
protocol Validator {
    func validate(_ value: String) -> Bool
}

// Protocol witness (faster)
struct Validator<T> {
    let validate: (String) -> Bool
}

// Concrete validators
extension Validator {
    static var email: Validator<String> {
        Validator { $0.contains("@") && $0.contains(".") }
    }
    
    static var notEmpty: Validator<String> {
        Validator { !$0.isEmpty }
    }
}

// Usage
let emailValidator = Validator<String>.email
emailValidator.validate("test@example.com")  // true
```

## 🔥 Opaque Types

```swift
// Return opaque type instead of protocol
func makeView() -> some View {
    VStack {
        Text("Hello")
        Text("World")
    }
}

// Generic opaque return
func makePublisher<T>() -> some Publisher<T, Never> {
    Just(value)
        .delay(for: .seconds(1), scheduler: DispatchQueue.main)
}
```

## 📦 Existential Types (Swift 5.7+)

```swift
// Old way
protocol Animal {
    func makeSound() -> String
}

let animals: [Animal] = [Dog(), Cat()]  // Implicit existential

// New explicit syntax
let animals: [any Animal] = [Dog(), Cat()]

// Constrained existential
func feed(_ animal: any Animal & Hashable) {
    // animal must conform to both Animal and Hashable
}
```

## 🎯 Practice Challenges

### Challenge 1: Build a Type-Safe Builder

Create a SQL query builder using result builders that prevents invalid queries at compile time.

### Challenge 2: Thread-Safe Cache

Implement a generic cache with property wrappers that's thread-safe and supports expiration.

### Challenge 3: Phantom Type State Machine

Create a state machine using phantom types where invalid state transitions are compile errors.

---

**Next**: [Concurrency Patterns →](./concurrency-patterns.md)
