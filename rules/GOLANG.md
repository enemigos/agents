# Go

## Naming for domain modeling

- Packages are short, lowercase, singular domain nouns: `billing`, `invoice`. Never use `util`, `common`, `shared`, `helpers`, `types`, `models`, or `lib`.
- One concept, one spelling. Pick `orgID` or `organizationID` and use it everywhere.
- Use plain functions and explicit data flow. Use methods for the primary type of a package.
- Do not embed types in public structs. Use a named field and delegate.
- Copy slices and maps that you retain from callers, and mutable internal state that you return. Copies are shallow.
- Add explicit tags to every serialized field. A `nil` slice encodes as `null` in default `encoding/json`. Use an empty slice only when the API requires `[]`.

## Branded types and correct construction

Use defined types for IDs, parsed strings, constrained numbers, and units: `type UserID string`, `type Cents int64`.

A defined type stops mix-ups, but any caller can convert `UserID("x")`. When a value must be valid, use a struct with an unexported field and a parser:

```go
type Email struct{ value string }

func ParseEmail(raw string) (Email, error) { ... }
```

A parser alone cannot make invalid construction impossible. Callers can still write `Email{}` or `var e Email`. Document the zero value. If it is invalid, reject it at domain boundaries.

Use `time.Time` for instants and `time.Duration` for periods. Never pass `int` seconds. If an external format needs a number, put the unit in the name: `TimeoutMillis`.

Pass capability types for privileged operations. Require `OrgScopedDB`, not `*sql.DB`.

## State machines and boolean blindness

For an entity with lifecycle states, use a sealed interface with an unexported marker method and pointer variants:

```go
//sumtype:decl
type InvoiceState interface{ isInvoiceState() }

type Draft struct{ Lines []LineItem }
type Sent struct{ SentAt time.Time }
type Paid struct{ PaidAt time.Time }

func (*Draft) isInvoiceState() {}
func (*Sent) isInvoiceState()  {}
func (*Paid) isInvoiceState()  {}

switch inv.State.(type) {
case *Draft:
case *Sent:
case *Paid:
default:
	// Reject nil interfaces and unknown variants.
}
```

Use pointer marker receivers and pointer cases together. A value receiver lets both `Draft` and `*Draft` satisfy the interface, and `gochecksumtype` treats pointers and values as equivalent, so a value case can miss pointer variants without a lint error.

Enable the `gochecksumtype` linter so a missing case fails lint, and set `linters.settings.gochecksumtype.default-signifies-exhaustive: false`. By default the linter treats a `default` case as exhaustive, which hides a missing variant.

The linter checks known variants only. It does not prove a closed union. A `nil` interface or an external type that embeds the interface bypasses the seal and reaches `default`. A typed nil pointer such as `(*Draft)(nil)` matches its own case instead, so reject typed nils where you construct or decode states.

For enums, use a defined type with `iota`. Enum types accept arbitrary numeric conversions, so validate at boundaries. Make the zero value `Unknown` or start at 1, so a zero value is never a valid state. Enable the `exhaustive` linter.

Avoid boolean parameters that control behavior. Use an options struct for a behavior flag or for more than 3 parameters. Use functional options only for extensible public library constructors. Booleans are fine as predicate results: `IsExpired(token) bool`.

## Error Handling

- Return `(T, error)`. Do not add `Result` libraries.
- Handle each error once. Wrap and return, or log and continue. Never log and return.
- Return a literal `nil` on success. An interface that holds a typed nil pointer is non-nil.
- Add context with `%w`: `fmt.Errorf("load user %q: %w", id, err)`. Use `%v` when the cause is an internal detail that callers must not depend on.
- Pick the error kind by contract: a sentinel for a fixed matchable case, a custom type for a matchable case with data, and `fmt.Errorf` when callers do not match. Match with `errors.Is` and `errors.As`. Never compare error text.
- Panic only for broken invariants. Call `os.Exit` and `log.Fatal` only in `main`. Put the program in `run() error` and exit once.

## Concurrency and context

- Never store `ctx context.Context` in a struct. Pass the caller's context down and honor cancellation. Putting `ctx` first does neither by itself.
- Never start a goroutine without a known way to stop it and wait for it. Prefer synchronous APIs. Let the caller add goroutines.
- Store `sync.Mutex` as a named, non-pointer field. Never embed it. Never copy a used mutex or a struct that contains one. Use pointer receivers on types that hold a mutex.
- Document whether exported types are safe for concurrent use.

## Configuration and resources

- Parse config in `main` into a typed struct. Do not call `os.Getenv` anywhere else.
- Do not use `init()`. Do not start goroutines, open connections, or do I/O at package load.
- Do not use mutable package-level variables. Inject dependencies through constructors.
- Inject the clock: `now func() time.Time`. Pure functions take `now time.Time`.
- Use `log/slog` for structured logs with safe fields.
- Wrap secrets in a `Redacted` type with value receivers, so values and pointers both satisfy the interfaces. `String` and `GoString` return `[REDACTED]`. `LogValue` returns `slog.StringValue("[REDACTED]")`. `MarshalJSON` returns a valid JSON string, quotes included: `"\"[REDACTED]\""`. `fmt` skips formatting methods on unexported fields, so a struct dump can still leak a secret. Keep the safe-fields rule. Do not rely on the wrapper alone.

