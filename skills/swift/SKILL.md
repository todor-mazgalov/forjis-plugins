---
name: swift
description: >
  Swift iOS/macOS development skill. Produces clean, safe, idiomatic modern Swift
  code following Apple platform conventions. Enforces value types, protocol-oriented
  design, Swift concurrency (async/await, actors), SwiftUI-first approach, proper
  memory management, strict error handling, and Apple Human Interface Guidelines.
  TRIGGER when: project uses Swift (Package.swift, *.xcodeproj, *.xcworkspace,
  *.swift files, Xcode project structure), or user asks to write Swift code,
  implement an iOS/macOS feature, or fix Swift bugs.
  DO NOT TRIGGER when: project is Kotlin/Android-only, Flutter, React Native
  without native Swift modules, or Objective-C-only without Swift files.
---

# Swift iOS Development

Write clean, safe, idiomatic modern Swift. Prioritize value semantics,
protocol-oriented design, Swift concurrency, and Apple platform conventions.

## Before Writing Code

1. **Read existing code first.** Search the project for existing types, extensions,
   utilities, and patterns before creating anything new. Reuse what exists.
2. **Check the Swift and deployment target versions.** Look at `Package.swift`
   (`swift-tools-version`), Xcode project settings (`SWIFT_VERSION`,
   `IPHONEOS_DEPLOYMENT_TARGET`) to know which language features and APIs
   are available.
3. **Check if SwiftUI or UIKit.** Look at the existing views, the app lifecycle
   (`@main App` vs `AppDelegate`), and project structure. Match the approach.
4. **Follow project conventions.** Match naming, file organization, dependency
   injection patterns, and architecture already in use.

---

## Swift Language Fundamentals

### Prefer value types (`struct`, `enum`) over reference types (`class`)

```swift
// CORRECT — value type, safe to copy, no shared mutable state
struct User {
    let id: UUID
    var name: String
    var email: String
}

// WRONG — class for plain data (unnecessary reference semantics)
class User {
    var id: UUID
    var name: String
    var email: String

    init(id: UUID, name: String, email: String) {
        self.id = id
        self.name = name
        self.email = email
    }
}
```

Use `class` only when you need:
- Reference semantics (identity matters, shared mutable state)
- Inheritance (prefer protocols + composition instead)
- Interop with Objective-C APIs

### Use `let` by default, `var` only when mutation is needed

```swift
// CORRECT — immutable by default
let name = "Alice"
let users: [User] = fetchUsers()

// Only var when you actually mutate
var count = 0
count += 1
```

### Use enums for closed sets with associated values

```swift
// CORRECT — rich enum modeling
enum NetworkError: Error {
    case notConnected
    case timeout(seconds: Int)
    case serverError(statusCode: Int, message: String)
    case decodingFailed(underlying: Error)
}

enum LoadingState<T> {
    case idle
    case loading
    case loaded(T)
    case failed(Error)
}
```

### Use guard for early exits, if-let for conditional binding

```swift
// CORRECT — guard for preconditions and early exit
func process(user: User?) throws -> Profile {
    guard let user else {
        throw ValidationError.missingUser
    }
    guard user.isActive else {
        throw ValidationError.inactiveUser
    }
    return Profile(name: user.name)
}

// CORRECT — if-let for optional branching
if let cachedValue = cache[key] {
    return cachedValue
}

// WRONG — nested optionals, pyramid of doom
if user != nil {
    if user!.isActive {
        if let profile = user!.profile {
            // deeply nested
        }
    }
}
```

### Never force-unwrap (`!`) outside of tests and IBOutlets

```swift
// WRONG — crashes at runtime if nil
let name = user.name!
let data = try! JSONDecoder().decode(User.self, from: json)

// CORRECT — safe unwrapping
guard let name = user.name else {
    return "Unknown"
}

do {
    let data = try JSONDecoder().decode(User.self, from: json)
} catch {
    logger.error("Failed to decode user: \(error)")
}
```

### Use trailing closure syntax and implicit returns

```swift
// CORRECT — clean closure syntax
let names = users.map { $0.name }
let active = users.filter { $0.isActive }
let sorted = users.sorted { $0.name < $1.name }

// CORRECT — implicit return for single-expression closures and computed properties
var fullName: String {
    "\(firstName) \(lastName)"
}

// WRONG — verbose
let names = users.map({ (user: User) -> String in
    return user.name
})
```

---

## Protocol-Oriented Design

### Define behavior through protocols, not inheritance

```swift
// CORRECT — protocol defines the contract
protocol UserRepository {
    func findById(_ id: UUID) async throws -> User?
    func save(_ user: User) async throws
    func delete(_ id: UUID) async throws
}

// Concrete implementations
struct CoreDataUserRepository: UserRepository {
    func findById(_ id: UUID) async throws -> User? {
        // Core Data implementation
    }
    // ...
}

struct InMemoryUserRepository: UserRepository {
    // Test implementation
}
```

### Use protocol extensions for default implementations

```swift
protocol Identifiable {
    var id: UUID { get }
}

extension Identifiable {
    var shortId: String {
        id.uuidString.prefix(8).lowercased()
    }
}
```

### Constrain generics with protocols

```swift
// CORRECT — generic with protocol constraint
func save<T: Encodable>(_ item: T, to url: URL) throws {
    let data = try JSONEncoder().encode(item)
    try data.write(to: url)
}

// CORRECT — protocol composition for multiple constraints
func display<T: Identifiable & CustomStringConvertible>(_ item: T) {
    print("\(item.id): \(item.description)")
}

// Use `some` for opaque return types
func makeRepository() -> some UserRepository {
    CoreDataUserRepository()
}
```

### Use `any` vs `some` correctly (Swift 5.7+)

```swift
// `some` — specific but hidden type (preferred when possible)
func makeView() -> some View {
    Text("Hello")
}

// `any` — existential, type-erased (when you need heterogeneous collections)
var repositories: [any UserRepository] = []

// WRONG — `any` when `some` would work
func getRepo() -> any UserRepository { ... } // use `some` here
```

---

## Swift Concurrency

### Use `async`/`await` for all asynchronous code

```swift
// CORRECT — structured concurrency
func fetchUserProfile(id: UUID) async throws -> UserProfile {
    let user = try await userService.findById(id)
    let preferences = try await preferencesService.get(for: id)
    return UserProfile(user: user, preferences: preferences)
}

// WRONG — completion handler callback
func fetchUserProfile(id: UUID, completion: @escaping (Result<UserProfile, Error>) -> Void) {
    userService.findById(id) { result in
        switch result {
        case .success(let user):
            // nested callback hell
        case .failure(let error):
            completion(.failure(error))
        }
    }
}
```

### Use `async let` for concurrent independent operations

```swift
// CORRECT — concurrent execution
func fetchDashboard(userId: UUID) async throws -> Dashboard {
    async let user = userService.findById(userId)
    async let orders = orderService.recent(for: userId)
    async let notifications = notificationService.unread(for: userId)

    return try await Dashboard(
        user: user,
        orders: orders,
        notifications: notifications
    )
}

// WRONG — sequential when operations are independent
let user = try await userService.findById(userId)
let orders = try await orderService.recent(for: userId)   // waits for user
let notifications = try await notificationService.unread(for: userId) // waits for orders
```

### Use `TaskGroup` for dynamic concurrency

```swift
func fetchAllUsers(ids: [UUID]) async throws -> [User] {
    try await withThrowingTaskGroup(of: User.self) { group in
        for id in ids {
            group.addTask {
                try await userService.findById(id)
            }
        }

        var users: [User] = []
        for try await user in group {
            users.append(user)
        }
        return users
    }
}
```

### Use actors for shared mutable state

```swift
// CORRECT — actor provides data-race safety
actor ImageCache {
    private var cache: [URL: UIImage] = [:]

    func image(for url: URL) -> UIImage? {
        cache[url]
    }

    func store(_ image: UIImage, for url: URL) {
        cache[url] = image
    }

    func clear() {
        cache.removeAll()
    }
}

// WRONG — class with manual locking
class ImageCache {
    private let lock = NSLock()
    private var cache: [URL: UIImage] = [:]

    func image(for url: URL) -> UIImage? {
        lock.lock()
        defer { lock.unlock() }
        return cache[url]
    }
}
```

### Use `@MainActor` for UI updates

```swift
// CORRECT — ensures UI updates on main thread
@MainActor
final class UserViewModel: ObservableObject {
    @Published var user: User?
    @Published var isLoading = false
    @Published var error: String?

    func loadUser(id: UUID) async {
        isLoading = true
        defer { isLoading = false }

        do {
            user = try await userService.findById(id)
        } catch {
            self.error = error.localizedDescription
        }
    }
}
```

### Task cancellation

```swift
func fetchData() async throws -> Data {
    // Check cancellation before expensive work
    try Task.checkCancellation()

    let data = try await networkClient.fetch(url)

    // Check again between steps
    try Task.checkCancellation()

    return try await process(data)
}

// Cancel tasks when no longer needed
let task = Task {
    try await fetchData()
}
// Later:
task.cancel()
```

---

## SwiftUI

### Use `@Observable` (iOS 17+) or `ObservableObject`

```swift
// PREFERRED — @Observable macro (iOS 17+, simpler, more performant)
@Observable
final class UserViewModel {
    var user: User?
    var isLoading = false
    var error: String?

    func load(id: UUID) async {
        isLoading = true
        defer { isLoading = false }
        do {
            user = try await userService.findById(id)
        } catch {
            self.error = error.localizedDescription
        }
    }
}

// In the view
struct UserView: View {
    let viewModel: UserViewModel

    var body: some View {
        // Automatically tracks which properties are read
        if viewModel.isLoading {
            ProgressView()
        } else if let user = viewModel.user {
            Text(user.name)
        }
    }
}
```

```swift
// FALLBACK — ObservableObject (iOS 13+)
final class UserViewModel: ObservableObject {
    @Published var user: User?
    @Published var isLoading = false
}

struct UserView: View {
    @StateObject private var viewModel = UserViewModel()
    // ...
}
```

### Keep views small — extract subviews

```swift
// CORRECT — small, focused views
struct UserProfileView: View {
    let user: User

    var body: some View {
        ScrollView {
            AvatarSection(user: user)
            InfoSection(user: user)
            ActionSection(user: user)
        }
    }
}

private struct AvatarSection: View {
    let user: User

    var body: some View {
        AsyncImage(url: user.avatarURL) { image in
            image.resizable().scaledToFill()
        } placeholder: {
            ProgressView()
        }
        .frame(width: 100, height: 100)
        .clipShape(Circle())
    }
}
```

### Use the right property wrapper

| Wrapper | When to use |
|---------|-------------|
| `@State` | Simple value owned by the view |
| `@Binding` | Two-way reference to parent's state |
| `@StateObject` | Create and own an ObservableObject |
| `@ObservedObject` | Receive an ObservableObject from parent |
| `@EnvironmentObject` | Shared object from ancestor via environment |
| `@Environment` | System environment values (colorScheme, locale) |
| `@AppStorage` | UserDefaults-backed value |

### Use `.task` for async loading

```swift
struct UserView: View {
    let userId: UUID
    @State private var user: User?

    var body: some View {
        Group {
            if let user {
                Text(user.name)
            } else {
                ProgressView()
            }
        }
        .task {
            // Automatically cancelled when view disappears
            user = try? await userService.findById(userId)
        }
    }
}
```

### Proper list performance

```swift
// CORRECT — identified data, lazy rendering
List(users) { user in  // User must conform to Identifiable
    UserRow(user: user)
}

// For custom identity
List(users, id: \.email) { user in
    UserRow(user: user)
}

// WRONG — ForEach without stable identity
ForEach(0..<users.count, id: \.self) { index in
    UserRow(user: users[index]) // breaks animations and diffing
}
```

---

## Error Handling

### Use typed `throws` (Swift 6+) or specific error types

```swift
// CORRECT — typed throws (Swift 6+)
func findUser(id: UUID) throws(DatabaseError) -> User {
    guard let user = try database.fetch(id) else {
        throw .notFound(id: id)
    }
    return user
}

// CORRECT — untyped throws with documented error types
/// - Throws: `NetworkError.notConnected` if offline,
///   `NetworkError.serverError` for 5xx responses
func fetchData(from url: URL) async throws -> Data {
    // ...
}
```

### Use `Result` for error values passed around

```swift
// When you need to store or pass error state
enum FetchResult<T> {
    case success(T)
    case failure(AppError)
}

// Swift's built-in Result type
func validate(input: String) -> Result<ValidatedInput, ValidationError> {
    guard !input.isEmpty else {
        return .failure(.empty)
    }
    return .success(ValidatedInput(value: input))
}
```

### Map errors to user-facing messages at the presentation layer

```swift
extension AppError {
    var userMessage: String {
        switch self {
        case .networkUnavailable:
            return String(localized: "error.network.unavailable")
        case .serverError:
            return String(localized: "error.server.generic")
        case .notFound:
            return String(localized: "error.notFound")
        }
    }
}
```

---

## Memory Management

### Understand ARC and avoid retain cycles

```swift
// CORRECT — weak self in escaping closures that capture self
class UserViewModel {
    func startPolling() {
        timer = Timer.scheduledTimer(withTimeInterval: 30, repeats: true) { [weak self] _ in
            guard let self else { return }
            Task { await self.refresh() }
        }
    }
}

// CORRECT — unowned when you guarantee lifetime
class Parent {
    let child: Child

    init() {
        child = Child(parent: self)
    }
}

class Child {
    unowned let parent: Parent // parent always outlives child

    init(parent: Parent) {
        self.parent = parent
    }
}
```

### Use `[weak self]` in closures that outlive the current scope

```swift
// CORRECT — weak capture prevents retain cycle
URLSession.shared.dataTask(with: url) { [weak self] data, response, error in
    guard let self else { return }
    self.handleResponse(data, error)
}

// In Swift concurrency, Task does NOT create a retain cycle
// (Task captures self strongly but is not retained by self)
func load() {
    Task {
        await self.fetchData() // OK — no retain cycle
    }
}
```

### Use `deinit` to verify cleanup in debug builds

```swift
class SomeViewController: UIViewController {
    deinit {
        #if DEBUG
        print("\(Self.self) deallocated")
        #endif
    }
}
```

---

## Networking

### Use URLSession with async/await and Codable

```swift
struct APIClient {
    private let session: URLSession
    private let decoder: JSONDecoder
    private let baseURL: URL

    func fetch<T: Decodable>(_ type: T.Type, from path: String) async throws -> T {
        let url = baseURL.appendingPathComponent(path)
        let (data, response) = try await session.data(from: url)

        guard let httpResponse = response as? HTTPURLResponse else {
            throw NetworkError.invalidResponse
        }

        guard (200...299).contains(httpResponse.statusCode) else {
            throw NetworkError.serverError(statusCode: httpResponse.statusCode)
        }

        return try decoder.decode(T.self, from: data)
    }
}
```

### Make Codable models match API exactly, transform in mapping layer

```swift
// API model — matches JSON keys exactly
struct UserDTO: Decodable {
    let user_id: String
    let full_name: String
    let created_at: String
}

// Domain model — clean Swift naming
struct User {
    let id: UUID
    let name: String
    let createdAt: Date
}

// Mapping
extension User {
    init(dto: UserDTO) throws {
        guard let id = UUID(uuidString: dto.user_id) else {
            throw MappingError.invalidId(dto.user_id)
        }
        self.id = id
        self.name = dto.full_name
        self.createdAt = try DateFormatter.iso8601.parse(dto.created_at)
    }
}
```

Or use `CodingKeys` for simple remapping:

```swift
struct User: Decodable {
    let id: UUID
    let name: String
    let createdAt: Date

    enum CodingKeys: String, CodingKey {
        case id = "user_id"
        case name = "full_name"
        case createdAt = "created_at"
    }
}
```

---

## Security

### Use Keychain for sensitive data

```swift
// WRONG — UserDefaults is unencrypted
UserDefaults.standard.set(token, forKey: "authToken")

// CORRECT — Keychain is encrypted at rest
import Security

func saveToKeychain(key: String, data: Data) throws {
    let query: [String: Any] = [
        kSecClass as String: kSecClassGenericPassword,
        kSecAttrAccount as String: key,
        kSecValueData as String: data,
        kSecAttrAccessible as String: kSecAttrAccessibleWhenUnlockedThisDeviceOnly,
    ]
    let status = SecItemAdd(query as CFDictionary, nil)
    guard status == errSecSuccess else {
        throw KeychainError.saveFailed(status)
    }
}
```

### Use App Transport Security — never disable globally

```xml
<!-- WRONG — disables ATS for all domains -->
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsArbitraryLoads</key>
    <true/>
</dict>

<!-- CORRECT — exception for specific domain if needed -->
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSExceptionDomains</key>
    <dict>
        <key>legacy-api.example.com</key>
        <dict>
            <key>NSTemporaryExceptionAllowsInsecureHTTPLoads</key>
            <true/>
        </dict>
    </dict>
</dict>
```

### Validate inputs, especially from deep links and URL schemes

```swift
func handle(url: URL) -> Bool {
    guard let components = URLComponents(url: url, resolvingAgainstBaseURL: false),
          let host = components.host,
          allowedHosts.contains(host) else {
        return false
    }

    // Validate and sanitize all query parameters
    guard let params = components.queryItems,
          let id = params.first(where: { $0.name == "id" })?.value,
          UUID(uuidString: id) != nil else {
        return false
    }

    return true
}
```

---

## Project Structure

Follow existing project structure. For new projects:

```
Sources/
├── App/
│   ├── MyApp.swift           # @main entry point
│   └── AppDelegate.swift     # If using UIKit lifecycle
├── Features/
│   └── User/
│       ├── Views/
│       │   ├── UserListView.swift
│       │   └── UserDetailView.swift
│       ├── ViewModels/
│       │   └── UserViewModel.swift
│       ├── Models/
│       │   └── User.swift
│       └── Services/
│           └── UserService.swift
├── Core/
│   ├── Networking/
│   │   └── APIClient.swift
│   ├── Storage/
│   │   └── KeychainManager.swift
│   ├── Extensions/
│   │   └── Date+Formatting.swift
│   └── Errors/
│       └── AppError.swift
└── Resources/
    ├── Assets.xcassets
    └── Localizable.xcstrings

Tests/
├── UserViewModelTests.swift
└── APIClientTests.swift
```

### Naming conventions

| Element | Convention | Example |
|---------|-----------|---------|
| Types (struct, class, enum, protocol) | PascalCase | `UserService`, `NetworkError` |
| Properties, methods, variables | camelCase | `userName`, `fetchData()` |
| Constants | camelCase (not UPPER_SNAKE) | `let maxRetryCount = 3` |
| Protocols describing capability | `-able`, `-ible` suffix | `Identifiable`, `Codable` |
| Protocols describing role | Noun | `UserRepository`, `Logger` |
| Boolean properties | `is`, `has`, `should` prefix | `isActive`, `hasPermission` |
| Factory methods | `make` prefix | `makeUserService()` |
| File names | Match primary type | `UserService.swift` |
| Extensions | `Type+Feature.swift` | `String+Validation.swift` |

---

## Testing

### Use Swift Testing framework (Xcode 16+) or XCTest

```swift
// PREFERRED — Swift Testing (modern, expressive)
import Testing

@Suite("UserService")
struct UserServiceTests {
    let sut: UserService
    let mockRepo: InMemoryUserRepository

    init() {
        mockRepo = InMemoryUserRepository()
        sut = UserService(repository: mockRepo)
    }

    @Test("creates a user with valid input")
    func createUser() async throws {
        let user = try await sut.create(name: "Alice", email: "alice@test.com")

        #expect(user.name == "Alice")
        #expect(user.email == "alice@test.com")
        #expect(user.id != UUID())
    }

    @Test("throws validation error for empty name")
    func createUserEmptyName() async {
        await #expect(throws: ValidationError.self) {
            try await sut.create(name: "", email: "alice@test.com")
        }
    }
}
```

```swift
// FALLBACK — XCTest (iOS 13+)
import XCTest

final class UserServiceTests: XCTestCase {
    func testCreateUser() async throws {
        let sut = UserService(repository: InMemoryUserRepository())
        let user = try await sut.create(name: "Alice", email: "alice@test.com")

        XCTAssertEqual(user.name, "Alice")
        XCTAssertEqual(user.email, "alice@test.com")
    }
}
```

### Use protocol-based dependency injection for testability

```swift
// Production
let viewModel = UserViewModel(
    userService: LiveUserService(apiClient: URLSessionAPIClient()),
    analytics: FirebaseAnalytics()
)

// Test
let viewModel = UserViewModel(
    userService: MockUserService(stubbedUsers: [testUser]),
    analytics: NoOpAnalytics()
)
```

---

## Common Mistakes to Avoid

| Mistake | Why it's wrong | Fix |
|---------|---------------|-----|
| Force unwrap (`!`) | Runtime crash if nil | `guard let`, `if let`, `??` |
| `class` for data models | Unnecessary reference semantics, shared mutable state | Use `struct` |
| Deep inheritance | Fragile, hard to change | Protocols + composition |
| Completion handlers | Callback hell, hard to reason about | `async`/`await` |
| Manual thread management | Data races, deadlocks | Actors, `@MainActor` |
| `DispatchQueue.main.async` | Mixing concurrency models | `@MainActor`, `.task` |
| Massive view controllers | Untestable, unmaintainable | Extract ViewModel + subviews |
| `UserDefaults` for secrets | Unencrypted plist on disk | Keychain |
| Stringly-typed APIs | No compiler checks | Enums, protocols, phantom types |
| Ignoring `Result` in callbacks | Silent failures | Handle both cases |
| `try!` / `try?` everywhere | Hides errors or crashes | `do/catch` with proper handling |
| Capturing `self` strongly | Retain cycles, memory leaks | `[weak self]` in escaping closures |
| God objects | Hard to test, modify, understand | Single responsibility principle |
| Mutable global state | Race conditions, unpredictable behavior | Dependency injection |

---

## Quick Reference

| Aspect | Recommended | Avoid |
|--------|-------------|-------|
| Types | `struct`, `enum`, value semantics | `class` for data, deep inheritance |
| Optionals | `guard let`, `if let`, `??` | Force unwrap `!`, `try!` |
| Concurrency | `async`/`await`, actors, `@MainActor` | Completion handlers, `DispatchQueue`, locks |
| Protocols | Protocol-oriented, composition, `some`/`any` | Deep inheritance, base classes |
| UI | SwiftUI with `@Observable`, `.task` | Massive `UIViewController`, manual threading |
| Errors | Typed throws, specific error enums | Generic `Error`, silent `try?` |
| Memory | `[weak self]`, value types, ARC awareness | Strong retain cycles, force unwrap |
| Testing | Swift Testing, protocol DI, mock via protocols | Testing implementation details, `XCTAssert` only |
| Security | Keychain, ATS, input validation | `UserDefaults` for tokens, disabled ATS |
| Naming | Swift API Design Guidelines, clarity at call site | Abbreviations, Hungarian notation |
