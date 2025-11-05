# Swift 6.0 Features

> **Complete data race safety and modern language features**

## 🎯 Key Features

### 1. Complete Concurrency

Swift 6 enforces **data race safety** at compile time.

```swift
actor DataStore {
    private var cache: [String: Data] = [:]
    
    func store(_ data: Data, forKey key: String) {
        cache[key] = data
    }
    
    func retrieve(forKey key: String) -> Data? {
        cache[key]
    }
}

// Usage
let store = DataStore()
await store.store(imageData, forKey: "profile")
let data = await store.retrieve(forKey: "profile")
```

### 2. Typed Throws

Specify exact error types for better error handling.

```swift
enum NetworkError: Error {
    case invalidURL
    case timeout
    case serverError(Int)
}

func fetchData() throws(NetworkError) -> Data {
    guard let url = URL(string: apiEndpoint) else {
        throw .invalidURL
    }
    // Implementation
}

// Usage
do {
    let data = try fetchData()
} catch let error as NetworkError {
    switch error {
    case .invalidURL:
        print("Invalid URL")
    case .timeout:
        print("Request timed out")
    case .serverError(let code):
        print("Server error: \(code)")
    }
}
```

### 3. Noncopyable Types

Zero-copy performance for resource management.

```swift
struct FileHandle: ~Copyable {
    private let descriptor: Int32
    
    init(path: String) throws {
        descriptor = open(path, O_RDONLY)
        guard descriptor >= 0 else {
            throw FileError.cannotOpen
        }
    }
    
    consuming func close() {
        Darwin.close(descriptor)
    }
    
    deinit {
        Darwin.close(descriptor)
    }
}

// Usage - no accidental copies
let handle = try FileHandle(path: "/tmp/data.txt")
handle.close() // consuming - handle is now invalid
```

### 4. Parameter Packs

Generic programming with variadic generics.

```swift
func combine<each T>(_ values: repeat each T) -> (repeat each T) {
    return (repeat each values)
}

let result = combine(42, "Hello", true, 3.14)
// result: (Int, String, Bool, Double)
```

### 5. Macros

Compile-time code generation.

```swift
@attached(member, names: named(init))
public macro AddInit() = #externalMacro(
    module: "MyMacros",
    type: "AddInitMacro"
)

@AddInit
struct User {
    let id: UUID
    let name: String
    let email: String
}

// Generates:
// init(id: UUID, name: String, email: String) {
//     self.id = id
//     self.name = name
//     self.email = email
// }
```

## 🔒 Sendable & Data Race Safety

### Sendable Protocol

```swift
struct User: Sendable {
    let id: UUID
    let name: String
    var email: String // Error: mutable stored property
}

// Fix: Make immutable or use actor
actor UserManager {
    var users: [UUID: User] = [:]
    
    func addUser(_ user: User) {
        users[user.id] = user
    }
}
```

### @MainActor

```swift
@MainActor
class ViewModel: ObservableObject {
    @Published var items: [Item] = []
    
    func loadItems() async {
        // Automatically on main thread
        items = await fetchItems()
    }
}
```

## 🚀 Performance Improvements

### Ownership

```swift
// Consuming ownership
func process(consuming data: LargeData) {
    // data is moved, not copied
    heavyOperation(data)
}

// Borrowing ownership
func analyze(borrowing data: LargeData) {
    // data is borrowed, no copy
    lightOperation(data)
}
```

### Embedded Swift

```swift
// For embedded systems
@_silgen_name("main")
func main() -> Never {
    while true {
        processInput()
    }
}
```

## 📊 Migration Guide

### From Swift 5.x

```swift
// Swift 5
class DataManager {
    var data: [String] = []
    
    func update() {
        DispatchQueue.global().async {
            self.data.append("new") // Potential data race
        }
    }
}

// Swift 6
actor DataManager {
    private var data: [String] = []
    
    func update() async {
        data.append("new") // Safe!
    }
}
```

### Gradual Migration

```swift
// Enable strict concurrency checking
// Build Settings -> Swift Compiler - Language
// Strict Concurrency Checking: Complete

// Or per-file
#if compiler(>=6.0)
@preconcurrency import OldFramework
#endif
```

## 🎯 Best Practices

### 1. Use Actors for Mutable State

```swift
actor Cache<Key: Hashable, Value> {
    private var storage: [Key: Value] = [:]
    
    func get(_ key: Key) -> Value? {
        storage[key]
    }
    
    func set(_ value: Value, forKey key: Key) {
        storage[key] = value
    }
}
```

### 2. Leverage Sendable

```swift
struct APIResponse: Sendable, Codable {
    let id: Int
    let data: String
    let timestamp: Date
}

func fetchData() async throws -> APIResponse {
    // Safe to send across concurrency boundaries
}
```

### 3. Typed Throws for APIs

```swift
enum ValidationError: Error {
    case emptyField(String)
    case invalidFormat(String)
}

func validate(email: String) throws(ValidationError) {
    guard !email.isEmpty else {
        throw .emptyField("email")
    }
    guard email.contains("@") else {
        throw .invalidFormat("email")
    }
}
```

## 📚 Resources

- **Swift Evolution**: [SE-0414](https://github.com/apple/swift-evolution/blob/main/proposals/0414-region-based-isolation.md)
- **WWDC 2025**: "What's New in Swift"
- **Documentation**: [Swift.org](https://swift.org/documentation/)
- **Migration Guide**: [Swift 6 Migration](https://swift.org/migration/)

## 🔗 Next Steps

- [Concurrency & Actors →](./concurrency.md)
- [Macros & Metaprogramming →](./macros.md)
- [Memory Management →](./memory.md)

---

**Source**: Apple Swift Documentation, Swift Evolution Proposals, WWDC 2025
