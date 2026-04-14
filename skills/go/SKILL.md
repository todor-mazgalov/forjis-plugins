---
name: go
description: >
  Go development skill. Produces clean, readable, idiomatic Go code following
  the conventions of Effective Go, the Go Code Review Comments wiki, and the
  standard library style. Enforces simplicity, explicit error handling, composition
  over inheritance, and strong use of the standard library before reaching for
  third-party packages.
  TRIGGER when: project uses Go (go.mod, go.sum, *.go files), or user asks to
  write Go code, implement a Go feature, design a Go service, or fix Go bugs.
  DO NOT TRIGGER when: project is Rust-only, C/C++-only, or another systems
  language without Go source files.
---

# Go Development

Write clean, readable, idiomatic Go. Prioritize simplicity, clarity, and the
standard library.

## Before Writing Code

1. **Read existing code first.** Search the project for existing packages,
   types, helpers, and shared modules before creating anything new. Reuse
   what exists.
2. **Check the Go version.** Look at `go.mod` for the `go` directive to know
   which language features are available (generics require 1.18+, `log/slog`
   requires 1.21+, range-over-func requires 1.23+). Use the newest features
   the project supports.
3. **Follow project conventions.** Look at package naming, directory layout,
   error handling patterns, and testing approach already in use. Match them.

## Core Go Philosophy

Go favors **simplicity over cleverness**. A clear 10-line function beats a
clever 3-line one. Code is read far more than it is written — optimize for
the reader.

- **Accept interfaces, return structs.** Functions should take the narrowest
  interface they need and return concrete types.
- **Make the zero value useful.** Design types so their zero value is valid
  and ready to use (e.g., `sync.Mutex`, `bytes.Buffer`).
- **A little copying is better than a little dependency.** Don't pull in a
  library for something you can write in 20 lines.
- **Don't panic.** Reserve `panic` for truly unrecoverable situations
  (programmer bugs, impossible states). Use error returns for everything else.

## Error Handling

Errors are values in Go. Handle them explicitly — never ignore them.

### Always check errors

```go
// WRONG — ignoring the error
data, _ := os.ReadFile("config.yaml")

// CORRECT — handle the error
data, err := os.ReadFile("config.yaml")
if err != nil {
    return fmt.Errorf("reading config: %w", err)
}
```

### Wrap errors with context

```go
// WRONG — bare return loses context
if err != nil {
    return err
}

// CORRECT — wrap with %w for unwrapping, add context
if err != nil {
    return fmt.Errorf("fetching user %d: %w", userID, err)
}
```

Use `%w` (not `%v`) so callers can use `errors.Is` and `errors.As`.

### Define sentinel errors and custom types when needed

```go
// Sentinel errors for expected conditions
var ErrNotFound = errors.New("not found")
var ErrConflict = errors.New("conflict")

// Custom error types for structured information
type ValidationError struct {
    Field   string
    Message string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation: %s — %s", e.Field, e.Message)
}
```

### Handle errors once

Don't log and return the same error — it leads to duplicate log lines.
Either handle it (log, retry, fallback) or return it. Not both.

```go
// WRONG — logged and returned
if err != nil {
    log.Printf("failed: %v", err)
    return err
}

// CORRECT — return with context, let the caller decide
if err != nil {
    return fmt.Errorf("processing order: %w", err)
}
```

## Naming

Go names are terse but descriptive. Shorter names for smaller scopes.

### Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| Packages | short, lowercase, no underscores | `http`, `strconv`, `orderservice` |
| Exported types/funcs | PascalCase | `OrderService`, `NewClient` |
| Unexported | camelCase | `parseHeader`, `maxRetries` |
| Interfaces (1 method) | method + "er" | `Reader`, `Stringer`, `Closer` |
| Acronyms | all caps when exported | `HTTPClient`, `userID`, `xmlParser` |
| Local variables | short, context-dependent | `r` for reader, `ctx` for context, `err` for error |
| Receivers | 1–2 letter abbreviation of type | `func (s *Server) Start()` |
| Test functions | `Test` + what's being tested | `TestParseConfig_InvalidYAML` |

### Package names

```go
// WRONG — redundant, stutter when used
package order
type OrderService struct{}  // order.OrderService

// CORRECT — the package name provides context
package order
type Service struct{}  // order.Service
```

Don't use `util`, `common`, `misc`, or `helpers` as package names — they say
nothing. Name packages by what they provide, not what they contain.

### Getters — no "Get" prefix

```go
// WRONG — Java-style getter
func (u *User) GetName() string { return u.name }

// CORRECT — Go convention
func (u *User) Name() string { return u.name }
```

## Interfaces

### Keep interfaces small

```go
// WRONG — kitchen-sink interface
type UserManager interface {
    Create(ctx context.Context, u User) error
    Get(ctx context.Context, id string) (User, error)
    Update(ctx context.Context, u User) error
    Delete(ctx context.Context, id string) error
    List(ctx context.Context) ([]User, error)
    Search(ctx context.Context, q string) ([]User, error)
    Export(ctx context.Context, format string) ([]byte, error)
}

// CORRECT — focused interfaces
type UserReader interface {
    Get(ctx context.Context, id string) (User, error)
}

type UserWriter interface {
    Create(ctx context.Context, u User) error
    Update(ctx context.Context, u User) error
}
```

### Define interfaces where they are consumed, not where they are implemented

```go
// WRONG — interface defined next to the implementation
package storage

type Store interface { ... }
type PostgresStore struct { ... }

// CORRECT — interface defined by the consumer
package orderservice

type OrderStore interface {
    Get(ctx context.Context, id string) (Order, error)
    Save(ctx context.Context, o Order) error
}

type Service struct {
    store OrderStore
}
```

This keeps packages decoupled and avoids unnecessary abstraction.

## Structs and Composition

### Use embedding for composition, not inheritance

```go
type Logger struct {
    *slog.Logger
}

type Server struct {
    Logger
    router *http.ServeMux
    db     *sql.DB
}
```

### Use functional options for complex construction

```go
type Server struct {
    addr    string
    timeout time.Duration
    logger  *slog.Logger
}

type Option func(*Server)

func WithTimeout(d time.Duration) Option {
    return func(s *Server) { s.timeout = d }
}

func WithLogger(l *slog.Logger) Option {
    return func(s *Server) { s.logger = l }
}

func NewServer(addr string, opts ...Option) *Server {
    s := &Server{addr: addr, timeout: 30 * time.Second}
    for _, opt := range opts {
        opt(s)
    }
    return s
}
```

### Use constructor functions

```go
// CORRECT — constructor validates and sets defaults
func NewClient(baseURL string) (*Client, error) {
    u, err := url.Parse(baseURL)
    if err != nil {
        return nil, fmt.Errorf("invalid base URL: %w", err)
    }
    return &Client{
        baseURL:    u,
        httpClient: &http.Client{Timeout: 10 * time.Second},
    }, nil
}
```

## Concurrency

### Start goroutines with clear ownership

Every goroutine must have a clear owner responsible for its lifecycle. Know
how it will stop.

```go
// WRONG — fire and forget
go processItem(item)

// CORRECT — managed lifecycle
func (s *Server) Start(ctx context.Context) error {
    g, ctx := errgroup.WithContext(ctx)
    for _, item := range items {
        g.Go(func() error {
            return processItem(ctx, item)
        })
    }
    return g.Wait()
}
```

### Use `context.Context` for cancellation and deadlines

```go
func fetchData(ctx context.Context, url string) ([]byte, error) {
    req, err := http.NewRequestWithContext(ctx, http.MethodGet, url, nil)
    if err != nil {
        return nil, err
    }
    resp, err := http.DefaultClient.Do(req)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()
    return io.ReadAll(resp.Body)
}
```

### Prefer `sync.Mutex` for simple shared state

```go
type Counter struct {
    mu    sync.Mutex
    count int
}

func (c *Counter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.count++
}
```

Use channels when you need to coordinate between goroutines (pipelines,
fan-out/fan-in). Use mutexes when you need to protect shared state.

### Use `errgroup` for concurrent tasks with error propagation

```go
g, ctx := errgroup.WithContext(ctx)

g.Go(func() error { return fetchUsers(ctx) })
g.Go(func() error { return fetchOrders(ctx) })

if err := g.Wait(); err != nil {
    return fmt.Errorf("fetching data: %w", err)
}
```

## Prefer the Standard Library

Go's standard library is comprehensive. Reach for it first.

### HTTP — `net/http`

```go
mux := http.NewServeMux()
mux.HandleFunc("GET /users/{id}", getUser)
mux.HandleFunc("POST /users", createUser)

server := &http.Server{
    Addr:         ":8080",
    Handler:      mux,
    ReadTimeout:  5 * time.Second,
    WriteTimeout: 10 * time.Second,
}
```

Go 1.22+ supports method and path parameter patterns in `http.ServeMux`
natively. Prefer this over third-party routers for new projects.

### JSON — `encoding/json`

```go
type User struct {
    ID    string `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email,omitempty"`
}

// Encoding
data, err := json.Marshal(user)

// Decoding
var user User
if err := json.NewDecoder(r.Body).Decode(&user); err != nil {
    http.Error(w, "invalid JSON", http.StatusBadRequest)
    return
}
```

### Logging — `log/slog` (Go 1.21+)

```go
logger := slog.New(slog.NewJSONHandler(os.Stdout, nil))
logger.Info("processing request",
    slog.String("method", r.Method),
    slog.String("path", r.URL.Path),
    slog.Int("status", status),
)
```

Prefer `log/slog` over `log`, `logrus`, or `zap` for new projects.

### Testing — `testing`

```go
func TestAdd(t *testing.T) {
    got := Add(2, 3)
    want := 5
    if got != want {
        t.Errorf("Add(2, 3) = %d, want %d", got, want)
    }
}
```

### Database — `database/sql`

```go
row := db.QueryRowContext(ctx,
    "SELECT name, email FROM users WHERE id = $1", userID)

var name, email string
if err := row.Scan(&name, &email); err != nil {
    if errors.Is(err, sql.ErrNoRows) {
        return ErrNotFound
    }
    return fmt.Errorf("querying user: %w", err)
}
```

### When third-party libraries ARE justified

- **Database drivers**: `pgx`, `go-sqlite3` — no built-in drivers
- **ORM (complex domains)**: `sqlc` (generates type-safe code from SQL) or `GORM`
- **CLI tools**: `cobra` — standard for CLI apps
- **gRPC**: `google.golang.org/grpc` — no built-in support
- **Observability**: `OpenTelemetry` — no built-in tracing/metrics
- **Migrations**: `goose`, `golang-migrate` — no built-in migration tool

Before adding any dependency, check if the standard library handles the use
case. If it does, use it.

## Testing

### Table-driven tests

```go
func TestParseSize(t *testing.T) {
    tests := []struct {
        name    string
        input   string
        want    int64
        wantErr bool
    }{
        {name: "bytes", input: "100B", want: 100},
        {name: "kilobytes", input: "2KB", want: 2048},
        {name: "invalid", input: "abc", wantErr: true},
        {name: "empty", input: "", wantErr: true},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := ParseSize(tt.input)
            if (err != nil) != tt.wantErr {
                t.Fatalf("ParseSize(%q) error = %v, wantErr %v",
                    tt.input, err, tt.wantErr)
            }
            if got != tt.want {
                t.Errorf("ParseSize(%q) = %d, want %d",
                    tt.input, got, tt.want)
            }
        })
    }
}
```

### Use `t.Helper()` in test helpers

```go
func assertEqual(t *testing.T, got, want any) {
    t.Helper()
    if got != want {
        t.Errorf("got %v, want %v", got, want)
    }
}
```

### Use `testdata/` for test fixtures

Place test files in `testdata/` directories — the Go toolchain ignores
these during builds.

### Test names describe scenarios

```go
func TestOrderService_CreateOrder_InsufficientStock(t *testing.T) { ... }
func TestOrderService_CreateOrder_DuplicateID(t *testing.T) { ... }
```

## Generics (Go 1.18+)

Use generics when the logic is truly type-independent. Don't use them
just because you can.

```go
// GOOD — generic utility that works across types
func Filter[T any](items []T, fn func(T) bool) []T {
    var result []T
    for _, item := range items {
        if fn(item) {
            result = append(result, item)
        }
    }
    return result
}

// GOOD — type constraint for numeric operations
func Sum[T int | int64 | float64](nums []T) T {
    var total T
    for _, n := range nums {
        total += n
    }
    return total
}
```

Don't use generics for types that only ever have one concrete instantiation.
If `T` is always `string`, just use `string`.

## Project Structure

Follow whatever structure the project already uses. If starting fresh, prefer:

```
project/
├── cmd/                    # Entry points (main packages)
│   └── server/
│       └── main.go
├── internal/               # Private application code
│   ├── order/
│   │   ├── service.go
│   │   ├── service_test.go
│   │   ├── store.go
│   │   └── model.go
│   └── user/
│       ├── service.go
│       └── service_test.go
├── pkg/                    # Public library code (if needed)
│   └── httputil/
│       └── response.go
├── go.mod
├── go.sum
└── Makefile
```

- `cmd/` — each subdirectory is a `main` package producing one binary
- `internal/` — code that cannot be imported by external modules
- `pkg/` — only if you intend to share code as a library; otherwise skip it
- Keep `main.go` thin — wire dependencies and start the server
- Group by domain (order, user), not by layer (models, handlers, services)

## Common Mistakes to Avoid

| Mistake | Why it's wrong | Fix |
|---------|---------------|-----|
| Ignoring errors (`_, _ :=`) | Hides bugs, silent failures | Always check errors |
| `panic` for expected errors | Crashes the program | Return errors |
| Global mutable state | Hard to test, race conditions | Pass dependencies explicitly |
| `init()` functions | Hidden side effects, test ordering issues | Use explicit initialization |
| Returning interfaces | Couples consumers to abstractions | Return concrete types |
| Large interfaces | Hard to implement and mock | Keep interfaces 1–3 methods |
| `interface{}` / `any` everywhere | Loses type safety | Use generics or concrete types |
| Not using `context.Context` | Can't cancel or set deadlines | Thread context through all I/O |
| Channels for simple locking | Overcomplicated, slower | Use `sync.Mutex` |
| Not closing response bodies | Leaks connections | `defer resp.Body.Close()` |
| Naked goroutines | Leaked goroutines, no error propagation | Use `errgroup` or managed lifecycles |
| Stuttering names | `user.UserService` is redundant | `user.Service` |
| Util/common packages | Meaningless package names | Name by purpose |
| Pointer receivers everywhere | Unnecessary indirection for small types | Value receivers for small immutable types |
| Not running `go vet` / linters | Catches bugs the compiler misses | Run `go vet`, use `golangci-lint` |

## Code Formatting

Go has one formatting standard. Do not deviate.

- **`gofmt`** — always format code with `gofmt` or `goimports`
- **`go vet`** — run it; it catches real bugs
- **`golangci-lint`** — use it if the project has it configured
- Tabs for indentation (this is non-negotiable in Go)
- No trailing whitespace
- Group imports: stdlib, blank line, third-party, blank line, internal

```go
import (
    "context"
    "fmt"
    "net/http"

    "github.com/jackc/pgx/v5"
    "golang.org/x/sync/errgroup"

    "myproject/internal/order"
)
```
