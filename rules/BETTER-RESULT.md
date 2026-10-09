# `better-result`

- USE `better-result` v3 for fallible TypeScript operations.
- RETURN `Err` for expected failures. Let `Panic` expose broken invariants and broken callback contracts.
- USE `Result.try()` and `Result.tryPromise()` to capture third-party exceptions and map them to tagged errors.
- USE `tap()` and `tapAsync()` for success side effects. USE `tapError()` and `tapErrorAsync()` for error side effects. These methods preserve the original Result.
- USE `Result.codec()` at serialization boundaries. `Result.serialize()`, `Result.deserialize()`, and `Result.hydrate()` do not exist in v3.

## Return at boundaries

```typescript
type HttpResponse = { status: number; body: string };

const response: HttpResponse = result.match({
  ok: (user) => ({ status: 200, body: JSON.stringify(user) }),
  err: (error) => error.match({
    UserNotFound: () => ({ status: 404, body: JSON.stringify(null) }),
    DatabaseUnavailable: () => ({ status: 503, body: JSON.stringify(null) }),
  }),
});
```

USE `matchError(error, handlers)` for structurally tagged errors. USE `matchError(handlers)(error)` for data-last matching.

## Functional Composition

```typescript
function processUser(result: Result<User, AppError>): string {
  return result
    .map((user) => user.name)
    .map((name) => name.toUpperCase())
    .match({
      ok: (name) => `User: ${name}`,
      err: (error) => `Failed: ${error.message}`,
    });
}
```

## Error handling

```typescript
class ValidationError extends TaggedError("ValidationError")<{
  message: string;
  field: string;
}> {}

class DbError extends TaggedError("DbError")<{
  message: string;
  operation: "read" | "write" | "delete";
}> {}

class AuthError extends TaggedError("AuthError")<{
  message: string;
  reason: "expired" | "invalid" | "missing";
}> {}

type AppError = ValidationError | DbError | AuthError;
type UserManagementError = ValidationError | DbError;
type AuthenticationError = AuthError | DbError;
```

## Pattern Matching

- ALWAYS include context in `TaggedError`. Store enough information to debug the failure.
- ALWAYS cover every error variant in exhaustive matching.
- USE `error.match({...})` for `TaggedError` unions. USE `matchError(error, {...})` for structural tagged errors.
- NEVER define a `TaggedError` payload property or subclass member named `match`.
- USE `TaggedError` for domain errors. `type DomainError = NotFoundError | PermissionError`.

```typescript
type ApiError = NotFoundError | ValidationError | InternalError;

function toHttpResponse(result: Result<Data, ApiError>): HttpResponse {
  return result.match({
    ok: (data) => ({
      status: 200,
      body: JSON.stringify(data),
    }),
    err: (error) => error.match({
      NotFoundError: (e) => ({
        status: 404,
        body: JSON.stringify({ error: e.message }),
      }),
      ValidationError: (e) => ({
        status: 400,
        body: JSON.stringify({ field: e.field, error: e.message }),
      }),
      InternalError: () => ({
        status: 500,
        body: JSON.stringify({ error: "Internal server error" }),
      }),
    }),
  });
}
```

## Generators

- ALWAYS return `Result.ok(...)` or `Result.err(...)` from a generator.
- NEVER return a bare value from a generator. `Result.gen` treats that as `Panic`.
- USE `Result.await(...)` in async generators.

```typescript
const result = await Result.gen(async function* () {
  const user = yield* Result.await(fetchUser(id));
  const validated = yield* Result.await(validateUser(user));
  const saved = yield* Result.await(saveUser(validated));
  return Result.ok(saved);
});
```

When any `yield*` encounters an `Err`, execution stops immediately. A `TaggedError` can also be yielded directly.

```typescript
class InactiveUser extends TaggedError("InactiveUser")<{
  userId: string;
  message: string;
}> {}

function authorize(user: User) {
  return Result.gen(function* () {
    if (!user.active) {
      yield* new InactiveUser({ userId: user.id, message: "User is inactive" });
    }
    return Result.ok(user);
  });
}
```

## Retries

```typescript
class DeadlockError extends TaggedError("DeadlockError")<{
  message: string;
}> {}

class TransactionError extends TaggedError("TransactionError")<{
  message: string;
  cause: unknown;
}> {}

async function runTransaction<T>(
  operation: () => Promise<T>
): Promise<Result<T, DeadlockError | TransactionError>> {
  return Result.tryPromise(
    {
      try: operation,
      catch: (cause) => {
        const error = cause instanceof Error ? cause : new Error(String(cause));
        const code = typeof cause === "object" && cause !== null && "code" in cause
          ? cause.code
          : undefined;

        if (code === "40P01" || code === "ER_LOCK_DEADLOCK") {
          return new DeadlockError({ message: error.message });
        }

        return new TransactionError({ message: error.message, cause });
      },
    },
    {
      retry: {
        times: 3,
        delayMs: 50,
        backoff: "exponential",
        shouldRetry: (error) => error._tag === "DeadlockError",
      },
    },
  );
}
```

USE `delayMs(error, context)` when the delay depends on the error. Do not combine dynamic delays with `backoff` or `jitter`. Pass a top-level `signal` and forward `context.signal` to the abort-aware operation. This stops pending delays and later retries. It cancels active work only when the operation forwards the signal.

- `shouldRetry` receives `(error, { attempt })`. USE `attempt` for per-attempt policies.
- `jitter` takes `true` or a number from 0 to 1. USE it to spread retry delays.
- Sync `Result.try()` accepts only `retry: { times }`. USE `Result.tryPromise()` for delays, backoff, jitter, and `signal`.

Catch `Panic` only at a true defect boundary. Do not convert it to a generic `Err`.

## Serialization

```typescript
const UserResultCodec = Result.codec({
  serialize: {
    ok: UserToWireSchema,
    err: DomainErrorToWireSchema,
  },
  deserialize: {
    ok: UserFromWireSchema,
    err: DomainErrorFromWireSchema,
  },
});

const encoded = await UserResultCodec.serialize(result);
const decoded = await UserResultCodec.deserialize(input);
```

USE safe codec methods at public, persisted, or independently versioned boundaries. USE `serializeUnsafe()` and `deserializeUnsafe()` only when a schema mismatch means a defect.

## Collections

- USE `Result.all()` and `Result.allAsync()` to combine a list of Results into one Result of a list. The first error in input order wins.
- USE `Result.partition()` and `Result.partitionAsync()` to split a list of Results into a `[successes, errors]` tuple.
- USE `Result.flatten()` to unwrap one level of nesting in a Result. The outer error unions with the inner error.
- Async variants also accept promises of Results. A rejected promise throws `Panic`.
