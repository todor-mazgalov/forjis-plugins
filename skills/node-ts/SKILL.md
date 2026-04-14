---
name: node-ts
description: >
  Node.js TypeScript development skill. Produces clean, type-safe, idiomatic
  modern TypeScript code for Node.js applications. Enforces strict type checking,
  proper async patterns, correct module system usage (ESM), modern runtime features,
  secure coding practices, proper error handling, and structured project conventions.
  TRIGGER when: project uses Node.js with TypeScript (tsconfig.json, package.json
  with ts-node/tsx/typescript dependencies, *.ts files in src/), or user asks to
  write Node.js TypeScript code, implement a Node.js feature, or fix Node.js TS bugs.
  DO NOT TRIGGER when: project is frontend-only (React/Vue/Angular SPA without
  Node.js backend), Deno-only, Bun-only without Node.js compatibility, or plain
  JavaScript without TypeScript.
---

# Node.js TypeScript Development

Write clean, type-safe, idiomatic modern TypeScript for Node.js. Prioritize
strict typing, code reuse, security, and built-in Node.js capabilities.

## Before Writing Code

1. **Read existing code first.** Search the project for existing modules, utilities,
   services, and shared types before creating anything new. Reuse what exists.
2. **Check the TypeScript and Node.js versions.** Look at `tsconfig.json` (`target`,
   `module`, `moduleResolution`), `package.json` (`engines`, `type`), and
   `.nvmrc`/`.node-version` to know which features are available.
3. **Check `tsconfig.json` strictness.** Ensure `strict: true` is enabled. If it
   isn't, treat it as if it were — write code that would pass strict checks.
4. **Follow project conventions.** Match naming, directory structure, import style,
   error handling patterns, and testing approach already in use.

---

## Strict TypeScript Configuration

Every Node.js TypeScript project must use strict configuration. If `tsconfig.json`
is missing or lax, recommend these settings:

```jsonc
{
  "compilerOptions": {
    "strict": true,
    "target": "ES2022",
    "module": "Node16",
    "moduleResolution": "Node16",
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInImports": true,
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "noUncheckedIndexedAccess": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "exactOptionalPropertyTypes": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}
```

Key flags and why they matter:

| Flag | Purpose |
|------|---------|
| `strict: true` | Enables all strict type checks — non-negotiable |
| `noUncheckedIndexedAccess` | Array/object index access returns `T \| undefined` — prevents runtime errors |
| `exactOptionalPropertyTypes` | Distinguishes `undefined` from missing — catches subtle bugs |
| `module: "Node16"` | Correct ESM/CJS interop for Node.js |
| `moduleResolution: "Node16"` | Matches Node.js module resolution algorithm |

---

## Type System Best Practices

### Never use `any` — use `unknown` for truly unknown types

```typescript
// WRONG — disables type checking entirely
function parse(input: any): any {
  return JSON.parse(input);
}

// CORRECT — forces caller to narrow the type
function parse(input: string): unknown {
  return JSON.parse(input);
}

// CORRECT — with proper narrowing
function parseUser(input: string): User {
  const data: unknown = JSON.parse(input);
  if (!isUser(data)) {
    throw new TypeError("Invalid user data");
  }
  return data;
}
```

### Use type guards for runtime validation

```typescript
// Type guard function
function isUser(value: unknown): value is User {
  return (
    typeof value === "object" &&
    value !== null &&
    "id" in value &&
    "name" in value &&
    typeof (value as User).id === "string" &&
    typeof (value as User).name === "string"
  );
}
```

For complex validation, use a schema validation library (Zod, Valibot) at
system boundaries — not `any` casts.

### Prefer interfaces for object shapes, type aliases for unions/intersections

```typescript
// CORRECT — interface for object shape (extendable, better error messages)
interface UserService {
  findById(id: string): Promise<User | null>;
  create(data: CreateUserDto): Promise<User>;
}

// CORRECT — type alias for union
type Result<T> = { ok: true; value: T } | { ok: false; error: Error };

// CORRECT — type alias for mapped/utility types
type Readonly<T> = { readonly [K in keyof T]: T[K] };
```

### Use `const` assertions and `satisfies` for type-safe literals

```typescript
// WRONG — type widens to string[]
const ROLES = ["admin", "user", "viewer"];

// CORRECT — preserves literal types
const ROLES = ["admin", "user", "viewer"] as const;
type Role = (typeof ROLES)[number]; // "admin" | "user" | "viewer"

// CORRECT — validates shape while preserving literal types
const CONFIG = {
  port: 3000,
  host: "localhost",
} satisfies ServerConfig;
```

### Use discriminated unions for state modeling

```typescript
// WRONG — optional fields create ambiguous state
interface ApiResponse {
  data?: User;
  error?: string;
  loading?: boolean;
}

// CORRECT — each state is explicit and exhaustive
type ApiResponse =
  | { status: "loading" }
  | { status: "success"; data: User }
  | { status: "error"; error: string };

function handle(response: ApiResponse): string {
  switch (response.status) {
    case "loading":
      return "Loading...";
    case "success":
      return response.data.name; // type-safe access
    case "error":
      return response.error;
  }
}
```

### Never use non-null assertion (`!`) — narrow instead

```typescript
// WRONG — hides potential null/undefined bugs
const user = users.find((u) => u.id === id)!;

// CORRECT — handle the null case
const user = users.find((u) => u.id === id);
if (!user) {
  throw new NotFoundError(`User ${id} not found`);
}
// user is now narrowed to User
```

### Use generics to avoid repetition, not to show off

```typescript
// GOOD — generic adds real value
async function fetchJson<T>(url: string, guard: (v: unknown) => v is T): Promise<T> {
  const response = await fetch(url);
  const data: unknown = await response.json();
  if (!guard(data)) {
    throw new TypeError(`Unexpected response shape from ${url}`);
  }
  return data;
}

// BAD — generic adds no value (just use the concrete type)
function getName<T extends { name: string }>(obj: T): string {
  return obj.name;
}
```

---

## Module System

### Use ESM (ECMAScript Modules)

Set `"type": "module"` in `package.json` and use ESM imports/exports throughout.

```typescript
// CORRECT — ESM imports
import { readFile } from "node:fs/promises";
import { join } from "node:path";
import { createServer } from "node:http";

// WRONG — CommonJS require
const fs = require("fs");
const path = require("path");
```

### Use `node:` protocol for built-in modules

```typescript
// CORRECT — explicit built-in import
import { EventEmitter } from "node:events";
import { Buffer } from "node:buffer";
import { setTimeout } from "node:timers/promises";

// WRONG — ambiguous, could shadow with npm package
import { EventEmitter } from "events";
```

### Use `.js` extensions in relative imports (required for Node16 module resolution)

```typescript
// CORRECT — include .js extension (even though source is .ts)
import { UserService } from "./services/user.service.js";
import type { User } from "./types/user.js";

// WRONG — missing extension (fails at runtime with Node16 resolution)
import { UserService } from "./services/user.service";
```

### Use `import type` for type-only imports

```typescript
// CORRECT — type-only import (erased at compile time)
import type { User, CreateUserDto } from "./types/user.js";
import { UserService } from "./services/user.service.js";

// WRONG — imports types as values
import { User, CreateUserDto, UserService } from "./index.js";
```

---

## Async Patterns

### Always use `async`/`await`, never raw callbacks

```typescript
// WRONG — callback hell
fs.readFile("config.json", (err, data) => {
  if (err) throw err;
  fs.writeFile("output.json", process(data), (err) => {
    if (err) throw err;
  });
});

// CORRECT — async/await with promises API
import { readFile, writeFile } from "node:fs/promises";

async function processConfig(): Promise<void> {
  const data = await readFile("config.json", "utf-8");
  await writeFile("output.json", process(data));
}
```

### Use `Promise.all` for independent concurrent operations

```typescript
// WRONG — sequential when operations are independent
const users = await fetchUsers();
const orders = await fetchOrders();
const products = await fetchProducts();

// CORRECT — concurrent execution
const [users, orders, products] = await Promise.all([
  fetchUsers(),
  fetchOrders(),
  fetchProducts(),
]);
```

### Use `Promise.allSettled` when partial failure is acceptable

```typescript
const results = await Promise.allSettled([
  sendEmail(user1),
  sendEmail(user2),
  sendEmail(user3),
]);

const failures = results.filter(
  (r): r is PromiseRejectedResult => r.status === "rejected"
);
if (failures.length > 0) {
  logger.warn(`${failures.length} emails failed to send`);
}
```

### Never use `async void` — always return the Promise

```typescript
// WRONG — fire-and-forget, errors are swallowed
app.get("/users", async (req, res) => {
  processInBackground(); // async void — unhandled rejection if it throws
});

// CORRECT — handle or await the promise
app.get("/users", async (req, res) => {
  await processInBackground();
});

// If truly fire-and-forget, catch explicitly
void processInBackground().catch((err) => logger.error("Background task failed", err));
```

### Use `AbortController` for cancellable operations

```typescript
async function fetchWithTimeout(url: string, timeoutMs: number): Promise<Response> {
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), timeoutMs);

  try {
    return await fetch(url, { signal: controller.signal });
  } finally {
    clearTimeout(timeoutId);
  }
}
```

### Handle unhandled rejections at the process level

```typescript
process.on("unhandledRejection", (reason: unknown) => {
  logger.fatal("Unhandled promise rejection", { reason });
  process.exit(1);
});

process.on("uncaughtException", (error: Error) => {
  logger.fatal("Uncaught exception", { error });
  process.exit(1);
});
```

---

## Error Handling

### Use custom error classes with proper inheritance

```typescript
class AppError extends Error {
  constructor(
    message: string,
    public readonly code: string,
    public readonly statusCode: number = 500,
    public readonly isOperational: boolean = true,
  ) {
    super(message);
    this.name = this.constructor.name;
    // Fix prototype chain for instanceof checks
    Object.setPrototypeOf(this, new.target.prototype);
  }
}

class NotFoundError extends AppError {
  constructor(resource: string, id: string) {
    super(`${resource} with id '${id}' not found`, "NOT_FOUND", 404);
  }
}

class ValidationError extends AppError {
  constructor(
    message: string,
    public readonly fields: Record<string, string>,
  ) {
    super(message, "VALIDATION_ERROR", 400);
  }
}
```

### Distinguish operational errors from programmer errors

```typescript
// Operational error — expected, recoverable (bad input, network timeout)
throw new ValidationError("Invalid email", { email: "Must be a valid email" });

// Programmer error — bug, crash and restart
// Let these propagate to the uncaughtException handler
const user = users[0]; // if users is unexpectedly undefined, that's a bug
```

### Use Result types for expected failures instead of try/catch

```typescript
type Result<T, E = Error> =
  | { ok: true; value: T }
  | { ok: false; error: E };

async function findUser(id: string): Promise<Result<User, NotFoundError>> {
  const user = await db.users.findUnique({ where: { id } });
  if (!user) {
    return { ok: false, error: new NotFoundError("User", id) };
  }
  return { ok: true, value: user };
}

// Caller handles both cases explicitly
const result = await findUser(id);
if (!result.ok) {
  return res.status(404).json({ error: result.error.message });
}
const user = result.value; // narrowed to User
```

### Never catch errors silently

```typescript
// WRONG — error swallowed
try {
  await riskyOperation();
} catch {
  // silently ignored
}

// CORRECT — handle, rethrow, or log with context
try {
  await riskyOperation();
} catch (error) {
  logger.error("Failed to complete risky operation", {
    error,
    context: { userId, operationId },
  });
  throw new AppError("Operation failed", "OPERATION_FAILED", 500);
}
```

---

## Node.js Built-in APIs — Prefer Over Third-Party

### Use `node:fs/promises` for file operations

```typescript
import { readFile, writeFile, mkdir, readdir, stat } from "node:fs/promises";

// WRONG — sync operations block the event loop
import { readFileSync } from "node:fs";
const data = readFileSync("file.txt", "utf-8");

// CORRECT — async operations
const data = await readFile("file.txt", "utf-8");
```

### Use `node:crypto` for hashing and random values

```typescript
import { randomUUID, randomBytes, createHash } from "node:crypto";

const id = randomUUID();
const token = randomBytes(32).toString("hex");
const hash = createHash("sha256").update(data).digest("hex");

// WRONG — using uuid package when randomUUID exists (Node 19+)
// WRONG — using Math.random() for tokens or IDs
```

### Use `node:test` for testing (Node 18+)

```typescript
import { describe, it, before, after, mock } from "node:test";
import assert from "node:assert/strict";

describe("UserService", () => {
  it("should create a user", async () => {
    const service = new UserService(mockRepo);
    const user = await service.create({ name: "Alice", email: "alice@test.com" });

    assert.equal(user.name, "Alice");
    assert.equal(user.email, "alice@test.com");
    assert.ok(user.id);
  });
});
```

If the project uses Jest, Vitest, or Mocha — follow that convention. But for
new projects, `node:test` is sufficient and has zero dependencies.

### Use `node:util` for promisify, inspect, and formatting

```typescript
import { promisify, inspect } from "node:util";
import { pipeline } from "node:stream/promises";

// Stream pipeline (already promise-based)
await pipeline(readStream, transformStream, writeStream);
```

### Use `node:path` and `node:url` for path manipulation

```typescript
import { join, resolve, dirname, basename, extname } from "node:path";
import { fileURLToPath } from "node:url";

// ESM equivalent of __dirname and __filename
const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);

// WRONG — string concatenation for paths
const configPath = "./config/" + env + ".json";

// CORRECT — path.join handles separators
const configPath = join(__dirname, "config", `${env}.json`);
```

### Use `node:timers/promises` for promisified timers

```typescript
import { setTimeout, setInterval } from "node:timers/promises";

// CORRECT — awaitable timeout
await setTimeout(1000);

// CORRECT — async iterable interval
for await (const _ of setInterval(5000)) {
  await healthCheck();
}
```

### When third-party packages ARE justified

| Use case | Recommended | Why |
|----------|------------|-----|
| Schema validation | Zod, Valibot | No built-in schema validation |
| Logging | Pino, Winston | `console.log` lacks structured output and log levels |
| HTTP framework | Fastify, Express | `node:http` is too low-level for applications |
| ORM / Query builder | Prisma, Drizzle, Knex | Raw SQL is error-prone at scale |
| Environment config | dotenv (minimal) | `process.env` has no validation |
| Task scheduling | node-cron | No built-in cron scheduler |

Before adding any dependency, check if Node.js built-in APIs handle the use
case. If they do, use them.

---

## Security

### Validate all external input at system boundaries

```typescript
import { z } from "zod";

const CreateUserSchema = z.object({
  name: z.string().min(1).max(100),
  email: z.string().email(),
  age: z.number().int().min(0).max(150).optional(),
});

type CreateUserDto = z.infer<typeof CreateUserSchema>;

function createUser(input: unknown): CreateUserDto {
  return CreateUserSchema.parse(input); // throws ZodError if invalid
}
```

### Never interpolate user input into commands or queries

```typescript
// WRONG — command injection
import { exec } from "node:child_process";
exec(`ls ${userInput}`); // shell injection

// CORRECT — use execFile with argument array
import { execFile } from "node:child_process";
execFile("ls", [userInput]); // no shell interpretation

// WRONG — SQL injection
const query = `SELECT * FROM users WHERE id = '${userId}'`;

// CORRECT — parameterized queries
const query = "SELECT * FROM users WHERE id = $1";
await db.query(query, [userId]);
```

### Use environment variables for secrets, never hardcode

```typescript
// WRONG
const API_KEY = "sk-1234567890abcdef";

// CORRECT
const API_KEY = process.env.API_KEY;
if (!API_KEY) {
  throw new Error("API_KEY environment variable is required");
}
```

### Set security headers in HTTP responses

```typescript
// Use helmet middleware or set manually
app.use((req, res, next) => {
  res.setHeader("X-Content-Type-Options", "nosniff");
  res.setHeader("X-Frame-Options", "DENY");
  res.setHeader("Strict-Transport-Security", "max-age=31536000; includeSubDomains");
  next();
});
```

### Sanitize file paths to prevent directory traversal

```typescript
import { resolve, normalize } from "node:path";

function safeResolvePath(baseDir: string, userPath: string): string {
  const resolved = resolve(baseDir, normalize(userPath));
  if (!resolved.startsWith(resolve(baseDir))) {
    throw new Error("Path traversal detected");
  }
  return resolved;
}
```

---

## Project Structure

Follow the project's existing structure. For new projects, prefer:

```
src/
├── index.ts               # Application entry point
├── config/                # Configuration loading and validation
│   └── index.ts
├── modules/               # Feature modules (domain-driven)
│   └── user/
│       ├── user.controller.ts
│       ├── user.service.ts
│       ├── user.repository.ts
│       ├── user.schema.ts     # Validation schemas
│       └── user.types.ts      # Module-specific types
├── shared/                # Cross-cutting concerns
│   ├── errors/
│   │   └── app-error.ts
│   ├── middleware/
│   └── types/
│       └── common.ts
└── __tests__/             # Integration tests
    └── user.integration.test.ts

# Unit tests colocated with source
src/modules/user/
├── user.service.ts
└── user.service.test.ts
```

### Naming conventions

| Element | Convention | Example |
|---------|-----------|---------|
| Files | kebab-case | `user-service.ts`, `create-user.dto.ts` |
| Classes | PascalCase | `UserService`, `AppError` |
| Interfaces | PascalCase (no `I` prefix) | `UserRepository`, not `IUserRepository` |
| Functions, variables | camelCase | `createUser`, `maxRetries` |
| Constants | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT`, `DEFAULT_PORT` |
| Types | PascalCase | `CreateUserDto`, `ApiResponse` |
| Enums | PascalCase members | `enum Role { Admin, User, Viewer }` |

---

## Dependency Injection and Architecture

### Use constructor injection for testability

```typescript
class UserService {
  constructor(
    private readonly userRepo: UserRepository,
    private readonly emailService: EmailService,
    private readonly logger: Logger,
  ) {}

  async create(dto: CreateUserDto): Promise<User> {
    const user = await this.userRepo.create(dto);
    await this.emailService.sendWelcome(user.email);
    this.logger.info("User created", { userId: user.id });
    return user;
  }
}
```

### Define dependencies as interfaces, not concrete classes

```typescript
// Define the contract
interface UserRepository {
  findById(id: string): Promise<User | null>;
  create(data: CreateUserDto): Promise<User>;
  update(id: string, data: Partial<User>): Promise<User>;
  delete(id: string): Promise<void>;
}

// Implement the contract
class PostgresUserRepository implements UserRepository {
  constructor(private readonly db: DatabaseClient) {}

  async findById(id: string): Promise<User | null> {
    return this.db.users.findUnique({ where: { id } });
  }
  // ...
}
```

---

## Testing

### Write tests that assert behavior, not implementation

```typescript
// WRONG — testing implementation details
it("should call userRepo.create once", async () => {
  await service.create(dto);
  assert.equal(mockRepo.create.mock.callCount(), 1);
});

// CORRECT — testing behavior
it("should return a user with the given name", async () => {
  const user = await service.create({ name: "Alice", email: "alice@test.com" });
  assert.equal(user.name, "Alice");
});
```

### Use factories for test data, not inline objects

```typescript
function createTestUser(overrides: Partial<User> = {}): User {
  return {
    id: randomUUID(),
    name: "Test User",
    email: "test@example.com",
    createdAt: new Date(),
    ...overrides,
  };
}

it("should format display name", () => {
  const user = createTestUser({ name: "Alice" });
  assert.equal(formatDisplayName(user), "Alice");
});
```

### Prefer integration tests over unit tests for I/O-heavy code

For services that primarily read/write databases or call APIs, integration
tests against real (or containerized) dependencies catch more bugs than
mocking everything.

---

## Performance and Runtime

### Never block the event loop

```typescript
// WRONG — CPU-intensive work blocks everything
function hashPassword(password: string): string {
  return bcryptSync(password, 12); // blocks event loop
}

// CORRECT — use async version or worker threads
import { hash } from "bcrypt";
async function hashPassword(password: string): Promise<string> {
  return hash(password, 12);
}

// CORRECT — offload heavy computation to worker threads
import { Worker } from "node:worker_threads";
```

### Use streams for large data

```typescript
import { createReadStream, createWriteStream } from "node:fs";
import { pipeline } from "node:stream/promises";
import { createGzip } from "node:zlib";

// CORRECT — streams large file without loading into memory
await pipeline(
  createReadStream("large-file.log"),
  createGzip(),
  createWriteStream("large-file.log.gz"),
);

// WRONG — loads entire file into memory
const data = await readFile("large-file.log");
```

### Implement graceful shutdown

```typescript
async function gracefulShutdown(signal: string): Promise<void> {
  logger.info(`Received ${signal}, shutting down gracefully`);

  server.close();
  await db.disconnect();
  await cache.quit();

  process.exit(0);
}

process.on("SIGTERM", () => gracefulShutdown("SIGTERM"));
process.on("SIGINT", () => gracefulShutdown("SIGINT"));
```

---

## Common Mistakes to Avoid

| Mistake | Why it's wrong | Fix |
|---------|---------------|-----|
| Using `any` | Disables type checking | Use `unknown` and narrow |
| Non-null assertion (`!`) | Hides null bugs | Narrow with conditionals |
| `require()` in ESM | Wrong module system | Use `import` |
| Sync file operations | Blocks event loop | Use `node:fs/promises` |
| `console.log` in production | No structure, no levels | Use a proper logger (Pino) |
| Catching `Error` and ignoring | Hides bugs | Log context, rethrow, or handle |
| Mutable shared state | Race conditions | Use immutable patterns or proper concurrency |
| Missing `.js` in relative imports | Runtime failure with Node16 | Always include `.js` extension |
| `enum` with numeric values | Unsafe reverse mapping | Use string enums or `as const` |
| Default exports | Harder to refactor, inconsistent imports | Use named exports |
| Barrel files (`index.ts` re-exports) | Slow builds, circular dependencies | Import from specific modules |
| `I` prefix on interfaces | Unnecessary Hungarian notation | Just use the name |
| `process.env` without validation | Runtime crashes from missing vars | Validate at startup with schema |
| `new Date()` in business logic | Untestable | Inject clock/time provider |

---

## Quick Reference

| Aspect | Recommended | Avoid |
|--------|-------------|-------|
| Types | `unknown`, discriminated unions, generics | `any`, non-null assertions, type casts |
| Modules | ESM, `node:` protocol, `.js` extensions | CJS `require()`, bare built-in imports |
| Async | `async`/`await`, `Promise.all` | Callbacks, raw `.then()` chains |
| Errors | Custom error classes, Result types | `catch {}`, generic `Error` |
| File I/O | `node:fs/promises`, streams | Sync operations, loading large files to memory |
| Testing | Behavior tests, factories, integration | Implementation detail tests, inline objects |
| Security | Input validation, parameterized queries | String interpolation in commands/queries |
| Exports | Named exports | Default exports, barrel files |
| Configuration | Validated env vars at startup | Scattered `process.env` access |
| Dependencies | Node.js built-ins first | Third-party for things Node.js already provides |
