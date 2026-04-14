---
name: java
description: >
  Java development skill. Produces clean, readable, idiomatic modern Java code.
  Enforces use of latest language features (records, sealed classes, pattern matching,
  switch expressions, text blocks, var), avoids code repetition, reuses existing
  project code, and strongly prefers JDK built-in APIs over third-party libraries.
  TRIGGER when: project uses Java (pom.xml, build.gradle, *.java files), or user
  asks to write Java code, implement a Java feature, or fix Java bugs.
  DO NOT TRIGGER when: project is Kotlin-only, Scala-only, or another JVM language
  without Java source files.
---

# Java Development

Write clean, readable, idiomatic modern Java. Prioritize code reuse, clarity,
and built-in JDK capabilities.

## Before Writing Code

1. **Read existing code first.** Search the project for existing classes, utilities,
   services, and shared modules before creating anything new. Reuse what exists.
2. **Check the Java version.** Look at `pom.xml` (`<maven.compiler.source>`),
   `build.gradle` (`sourceCompatibility`), or `.java-version` to know which
   language features are available. Use the newest features the project supports.
3. **Follow project conventions.** Look at naming, package structure, dependency
   injection style, error handling patterns, and testing approach already in use.
   Match them.

## Modern Language Features

Use the latest Java features available in the project. They exist to make code
shorter, safer, and more expressive — not using them means writing more code
that does less.

### Records instead of data-carrier classes

```java
// WRONG — boilerplate class for plain data
public class UserDto {
    private final String name;
    private final String email;

    public UserDto(String name, String email) {
        this.name = name;
        this.email = email;
    }

    public String getName() { return name; }
    public String getEmail() { return email; }

    @Override public boolean equals(Object o) { /* ... */ }
    @Override public int hashCode() { /* ... */ }
    @Override public String toString() { /* ... */ }
}

// CORRECT — record
public record UserDto(String name, String email) {}
```

Records give you the constructor, accessors, `equals`, `hashCode`, and `toString`
for free. Use them for DTOs, value objects, method return types, and any class
whose identity is defined by its data.

Add compact constructors for validation:

```java
public record PositiveAmount(double value) {
    public PositiveAmount {
        if (value <= 0) throw new IllegalArgumentException("Amount must be positive");
    }
}
```

### Sealed classes for closed type hierarchies

```java
// When you have a fixed set of subtypes, seal the hierarchy
public sealed interface Shape permits Circle, Rectangle, Triangle {}
public record Circle(double radius) implements Shape {}
public record Rectangle(double width, double height) implements Shape {}
public record Triangle(double base, double height) implements Shape {}
```

Sealed types let the compiler verify exhaustiveness in switch expressions —
if you add a new subtype, every switch that handles the sealed type will fail
to compile until updated. This turns runtime bugs into compile-time errors.

### Pattern matching in switch

```java
// WRONG — instanceof chains
if (shape instanceof Circle c) {
    return Math.PI * c.radius() * c.radius();
} else if (shape instanceof Rectangle r) {
    return r.width() * r.height();
} else if (shape instanceof Triangle t) {
    return 0.5 * t.base() * t.height();
}

// CORRECT — pattern matching switch expression
return switch (shape) {
    case Circle c    -> Math.PI * c.radius() * c.radius();
    case Rectangle r -> r.width() * r.height();
    case Triangle t  -> 0.5 * t.base() * t.height();
};
```

Use guarded patterns when you need conditions:

```java
return switch (response) {
    case Success s when s.body().isEmpty() -> handleEmpty();
    case Success s                        -> handleBody(s.body());
    case Error e                          -> handleError(e.code());
};
```

### Switch expressions for all multi-branch logic

```java
// WRONG — switch statement with break
String label;
switch (status) {
    case ACTIVE:
        label = "Active";
        break;
    case INACTIVE:
        label = "Inactive";
        break;
    default:
        label = "Unknown";
        break;
}

// CORRECT — switch expression
var label = switch (status) {
    case ACTIVE   -> "Active";
    case INACTIVE -> "Inactive";
    default       -> "Unknown";
};
```

Switch expressions return a value, cover all cases (compiler-enforced with
sealed types), and eliminate `break`/fall-through bugs.

### `var` for local type inference

```java
// WRONG — redundant type declaration
Map<String, List<UserDto>> groupedUsers = userService.groupByDepartment();
HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());

// CORRECT — var when the type is obvious from context
var groupedUsers = userService.groupByDepartment();
var response = client.send(request, HttpResponse.BodyHandlers.ofString());
```

Use `var` when the right-hand side makes the type clear. Keep explicit types
for literals and ambiguous cases (`var count = 0` is fine, `var result = process()`
is fine when `process` has a clear name, but prefer explicit types when it
genuinely aids comprehension).

### Text blocks for multi-line strings

```java
// WRONG — concatenation
String query = "SELECT u.name, u.email\n" +
               "FROM users u\n" +
               "WHERE u.active = true\n" +
               "ORDER BY u.name";

// CORRECT — text block
var query = """
        SELECT u.name, u.email
        FROM users u
        WHERE u.active = true
        ORDER BY u.name
        """;
```

Use text blocks for SQL, JSON, HTML, log messages, and any multi-line string.
The closing `"""` position controls indentation stripping.

### Other modern features to prefer

| Feature | Use for | Since |
|---------|---------|-------|
| `List.of()`, `Map.of()`, `Set.of()` | Immutable collection literals | 9 |
| `Optional` | Nullable return types | 8 |
| `Stream.toList()` | Collecting to unmodifiable list | 16 |
| `String::formatted` | String formatting | 15 |
| `instanceof` pattern | Type-check + cast in one step | 16 |
| `record` | Immutable data carriers | 16 |
| `sealed` | Closed type hierarchies | 17 |
| Pattern matching `switch` | Multi-branch dispatch | 21 |
| String templates | Interpolation (preview) | 21+ |
| Unnamed variables `_` | Ignoring unused bindings | 22 |
| Gatherers | Custom intermediate stream ops | 22 |

## Prefer JDK Built-ins Over Third-Party Libraries

The JDK is large and capable. Reaching for a library when the JDK already has
the functionality adds dependency weight, version conflicts, and security
surface for no benefit.

### HTTP — `java.net.http.HttpClient`

```java
// WRONG — adding OkHttp or Apache HttpClient for simple requests
// CORRECT — built-in HttpClient (since Java 11)
var client = HttpClient.newHttpClient();
var request = HttpRequest.newBuilder()
        .uri(URI.create("https://api.example.com/users"))
        .header("Accept", "application/json")
        .GET()
        .build();
var response = client.send(request, HttpResponse.BodyHandlers.ofString());
```

### JSON — consider built-in alternatives

If the project already uses Jackson or Gson, follow that convention. If not available pick Gson.
But for new code with simple needs, consider:
- Records + a lightweight parser already in the project
- `java.io` serialization for internal data

Only add a JSON library when you genuinely need complex mapping, streaming, or
schema validation.

### Collections and data manipulation — Streams API

```java
// WRONG — adding Guava just for collection utilities
// CORRECT — JDK Streams + Collections utility methods
var activeEmails = users.stream()
        .filter(User::isActive)
        .map(User::email)
        .toList();

var grouped = orders.stream()
        .collect(Collectors.groupingBy(Order::category));

var lookup = items.stream()
        .collect(Collectors.toMap(Item::id, Function.identity()));
```

### Concurrency — `java.util.concurrent`

```java
// Prefer virtual threads (Java 21+)
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    var futures = tasks.stream()
            .map(task -> executor.submit(task::execute))
            .toList();
    // ...
}

// Structured concurrency (Java 21+ preview)
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    var userFuture = scope.fork(() -> fetchUser(id));
    var ordersFuture = scope.fork(() -> fetchOrders(id));
    scope.join().throwIfFailed();
    return new UserProfile(userFuture.get(), ordersFuture.get());
}
```

### Date/time — `java.time`

```java
// WRONG — Joda-Time (was needed before Java 8, not anymore)
// CORRECT — java.time
var now = Instant.now();
var date = LocalDate.of(2024, Month.MARCH, 15);
var duration = Duration.between(start, end);
```

### When third-party libraries ARE justified

- **Logging facade**: SLF4J + Logback/Log4j2 — JDK logging is clunky
- **Testing**: JUnit 5, AssertJ, Mockito — no JDK alternative
- **Database access**: JDBC is built-in but an ORM may be warranted for complex domains
- **JSON**: Gson when you need annotation-driven mapping at scale
- **Build tools**: Maven/Gradle plugins — no alternative (prefer Gradle)

Before adding any dependency, check if the JDK already handles the use case.
If it does, use it.

## Code Reuse

### Search before creating

Before writing a new class or method, search the codebase:
- `Glob` for `**/*.java` to find existing classes
- `Grep` for class names, method signatures, similar patterns

If something similar exists, extend or compose it instead of duplicating.

### Extract shared logic

```java
// WRONG — same validation scattered across services
public class OrderService {
    public void create(Order order) {
        if (order.amount() <= 0) throw new IllegalArgumentException("...");
        // ...
    }
}
public class RefundService {
    public void process(Order order) {
        if (order.amount() <= 0) throw new IllegalArgumentException("...");
        // ...
    }
}

// CORRECT — shared validation
public record Order(String id, double amount) {
    public Order {
        if (amount <= 0) throw new IllegalArgumentException("Amount must be positive");
    }
}
```

### Utility methods belong in the type they operate on

```java
// WRONG — a StringUtils class with one method
public class StringUtils {
    public static String truncate(String s, int maxLen) { /* ... */ }
}

// CORRECT — if you control the type, add the behavior there
// If you don't (e.g., java.lang.String), a focused utility is OK,
// but keep it in the module that needs it, not a global "utils" package
```

### Compose, don't inherit

Favor composition and interfaces over deep inheritance hierarchies.
One level of inheritance is fine; three is a code smell.

## Readability Standards

1. **Descriptive names.** `OrderValidator`, not `OV`. `calculateDiscount`, not `calc`.
2. **Small methods.** ~30-40 lines max. Single responsibility. If a method needs
   a comment explaining what a section does, extract that section into a named method.
3. **Early returns** for guard clauses and error states — avoid deep nesting.
4. **Flat control flow.** Avoid nesting beyond 3 levels. Extract into methods.
5. **No magic numbers or strings.** Use named constants or enums.
6. **Group related fields and methods.** Keep collaborating code adjacent.
7. **One public class per file.** Package-private helpers can share a file only
   if they're small and tightly coupled.

### Naming conventions

| Element | Convention | Example |
|---------|-----------|---------|
| Classes, records, enums | PascalCase | `OrderService` |
| Methods, variables | camelCase | `calculateTotal` |
| Constants | UPPER_SNAKE | `MAX_RETRY_COUNT` |
| Packages | lowercase, dot-separated | `com.example.orders` |
| Type parameters | single uppercase letter or short name | `T`, `E`, `K` |

## Error Handling

### Use specific exception types

```java
// WRONG — generic exceptions
throw new Exception("Order not found");
throw new RuntimeException("Invalid state");

// CORRECT — specific, meaningful exceptions
throw new OrderNotFoundException(orderId);
throw new IllegalStateException("Cannot cancel a shipped order");
```

### Don't catch and ignore

```java
// WRONG
try { riskyOperation(); } catch (Exception e) { /* swallow */ }

// CORRECT — handle, rethrow, or log with context
try {
    riskyOperation();
} catch (IOException e) {
    throw new ServiceException("Failed to process order " + orderId, e);
}
```

### Prefer unchecked exceptions for programming errors

Use checked exceptions only when the caller can meaningfully recover.
Use unchecked (runtime) exceptions for bugs and invalid states.

## Common Mistakes to Avoid

| Mistake | Why it's wrong | Fix |
|---------|---------------|-----|
| Data class instead of record | Unnecessary boilerplate | Use `record` |
| `if/else` chains on type | Verbose, not exhaustive | Pattern matching `switch` |
| `switch` statement with `break` | Fall-through bugs, no return value | Switch expression with `->` |
| String concatenation in loops | O(n^2) performance | `StringBuilder` or `String.join` |
| Catching `Exception` | Hides bugs, catches everything | Catch specific types |
| Returning `null` | NullPointerException risk | Return `Optional` |
| Mutable DTOs with getters/setters | Harder to reason about | Immutable records |
| Adding Guava for `ImmutableList` | JDK has `List.of()` since Java 9 | Use `List.of()` |
| Adding Apache Commons for `StringUtils.isBlank` | JDK has `String::isBlank` since Java 11 | Use `str.isBlank()` |
| Deep inheritance hierarchies | Fragile, hard to change | Composition + interfaces |
| God classes with 500+ lines | Unreadable, untestable | Split by responsibility |
| `var` everywhere including API boundaries | Hides intent on public APIs | `var` for locals, explicit types on signatures |
| API resources not registered in SERVER runtime | Resources will be ignored due to constraint configuration problem | Implement proper resources registration |

## Project Structure Conventions

Follow whatever structure the project already uses. If starting fresh, prefer:

```
src/main/java/com/example/
├── domain/              # Core business objects (records, entities, enums)
│   ├── Order.java
│   └── OrderStatus.java
├── service/             # Business logic
│   └── OrderService.java
├── repository/          # Data access
│   └── OrderRepository.java
├── controller/          # HTTP/API layer
│   └── OrderController.java
├── config/              # Configuration and wiring
└── exception/           # Custom exception types

src/test/java/com/example/
├── service/
│   └── OrderServiceTest.java
└── ...
```
