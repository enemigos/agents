# Standards

### Runtime Environment

- **Prefer isomorphic code**: Write runtime-agnostic code that works in Node, browser, and workers whenever possible
- **Clear runtime indicators**: When code is environment-specific, add a comment at the top of the file:

```ts
// @env node
// @env browser
```

## Naming for domain modeling

- Exported symbols get 2-4 word names with a domain word. `diffUserObjects`, not `diff`; `queueEventForDispatch`, not `queue`. Use the shortest name that greps uniquely.
- Give generic verbs an object. `sanitizeEmailHtml`, not `sanitize`; `validateSmtpConfig`, not `validateConfig`.
- One definition site per symbol. Move and export shared helpers to a concept-named module instead of copying. Never duplicate functions.
- Put context in the symbol, not the path. Name `formatDurationMs`, not `format` inside a `duration` module. Exception: strict conventions like contract `Input`/`Output`.
- One concept, one spelling. Pick `organizationId` or `orgId` and use it everywhere. Reuse existing codebase terms; do not introduce near-synonyms.
- Never use bare-role filenames. Prefix the domain: `billing-plan-config.ts`, not `config.ts`. Avoid bare `types.ts` or `utils.ts`. Use`index.ts` only for re-exports.
- Brand primitive IDs with `z.string().brand<'UserId'>()` or newtypes. Use `(userId: UserId, orgId: OrgId)`, not `(userId: string, orgId: string)`.
- Pass capability tokens for privileged operations. Require `OrgScopedDb`, not a raw connection.
- Model state with discriminated unions, not nullable fields with implicit rules.
- Name types for compiler feedback. `OrgScopedDb`, not `Ctx2`. Avoid `any`.  
- Mark dead ends with `@deprecated` and point to the replacement.

## TypeScript

TypeScript is infested by landmines, from `Promise` to try-catch blocks.

All steps in problem-solving must be explicit, pure and composable.

### TypeScript Patterns

- Bias for functional programming.
- Colocate code that changes often close together, code that changes together belongs together.
- Compose a program via multiple isolated functions, features are about piping data into the right shape.
- Constructive type modeling: Build the shape so the illegal value can't be constructed.
- Discriminated unions: Model variants with a kind literal discriminant so impossible states can't be represented. No optional-field bags.
- Schema-derived types: Reach for `Pick`/`Omit`/`Parameters`/`ReturnType`/`Awaited`/`typeof` before declaring a new interface.
- Object args: Pass objects, not positional, so argument order is self-documenting. Skip on hot paths (per-frame render, tokenizers, parsers).
- `Type` > `interface`
- Arrow functions are for inline callbacks only: a function written as an argument, such as `.map((user) => user.id)` or `result.match({ ok: (data) => data })`. The arrow keeps a short callback short.
- Every other function uses the `function` keyword: exported, module-level, nested, and named helpers. A standalone `function` is easy to spot when you scan a file.
- NEVER bind an arrow function to a `const`, `let`, object property, or export. Write `function toHttpResponse(...) {}`, not `const toHttpResponse = (...) => {}`.
- In an `export const X = { ... } as const` module object, reference functions declared with `function`. Do not write arrows as properties.
- If an inline callback grows past a few lines or needs a name, move it to a `function` declaration.
- Prefer small, cohesive modules organized around one primary domain type or concept
- In TypeScript, when a module is centered on a primary type, prefer an OCaml-style namespaced module pattern: `export type X = ...` plus `export const X = { ... } as const` for constructors, parsers, combinators, and other domain operations
- Prefer attaching domain logic to the module for its primary type rather than scattering it across generic utility files
- When a module starts accumulating substantial logic for other types or domains, split those concerns into their own sibling modules
- Prefer specific domain modules over catch-all `utils` files
- `Record<string, unknown>` considered harmful slop. It's slop likely indicative of widening or erasing types and or not parsing types into stronger domain types at IO boundaries.

## Branded types and correct construction

Use branded/refined types when they prevent realistic misuse or invalid construction, especially for:

- IDs: `UserId`, `OrgId`, `WorkflowId`
- parsed strings: `EmailAddress`, `NonEmptyString`, `Url`
- constrained numbers: `PositiveInt`, `Cents`, `Percentage`
- units: `Milliseconds`, `Bytes`, `UsdCents`

Construct branded values through parsers or smart constructors. Avoid passing raw strings/numbers where a domain type exists.

Avoid optional/null/undefined values in functions that require a value. Push optionality outward. Branch or parse before calling.

Avoid `Partial<T>` as an application/domain input unless partiality is the real domain concept. Prefer explicit input types for each operation.

## State machines and boolean blindness

When an entity has meaningful lifecycle states, model them with tagged unions or equivalent value classes.

Prefer:

```ts
type Invoice =
  | { readonly _tag: "Draft"; readonly id: InvoiceId; readonly lines: NonEmptyArray<LineItem> }
  | { readonly _tag: "Sent"; readonly id: InvoiceId; readonly sentAt: Instant }
  | { readonly _tag: "Paid"; readonly id: InvoiceId; readonly paidAt: Instant };
```

Avoid:

```ts
type Invoice = {
  readonly isSent: boolean;
  readonly isPaid: boolean;
  readonly sentAt?: Date;
  readonly paidAt?: Date;
};
```

Avoid boolean parameters that control behavior:

```ts
createUser(input, true);
```

Prefer named options or domain types:

```ts
createUser(input, { emailVerification: "skip" });
```

Booleans are fine as clear predicate return values:

```ts
isExpired(token): boolean;
hasPermission(user, permission): boolean;
```

## Error Handling

- USE errors as values over throwing exceptions for expected failure paths
- USE `better-result` (`dmmulroy/better-result`) for fallible operations
- USE tagged/structured error types over untyped error strings
- Reserve thrown exceptions for truly exceptional, unrecoverable, or framework-boundary cases
- Propagate errors explicitly; do not swallow them or replace them with success-shaped fallbacks

If the project uses `better-result`, read `~/.dud/rules/BETTER-RESULT.md` for patterns.

## Frontend

### React

**BANNED: `useEffect`**

Direct usage of `useEffect` is BANNED in this codebase. Most `useEffect` usage compensates for something React already provides better primitives for. Banning the hook forces logic to be declarative, predictable, and event-driven, preventing race conditions, infinite loops, and dependency hell. Use a query library (e.g., React Query) for data fetching.

Banning `useEffect` forces cleaner tree design:
- Parents own orchestration and lifecycle boundaries.
- Children can assume preconditions are already met.
- Each unit does one job, and coordination happens at clear boundaries.

### Write better error messages

Treat errors as recovery flows. Design them with product, design, and engineering alongside happy paths.

Before writing, inspect the trigger and UI. Verify what failed, what succeeded or stayed safe, the known cause, placement, whether it blocks the task, and supported recovery paths. Flag missing facts or recovery options as product or engineering gaps. Never invent causes, guarantees, or actions.

Use only the parts the situation needs:

1. **Outcome:** State what happened in user terms, including partial success.
2. **Cause:** Explain the known reason plainly. Own failures on our side.
3. **Reassurance:** Say what was saved or unchanged only when verified.
4. **Action:** Give an exact recovery step. Say where or how when needed.
5. **Escape:** Offer another path or support if the user may remain blocked.

Keep copy calm, concise, and empathetic. Do not blame users or third parties. Remove unnecessary jargon and excessive apologies. Avoid jokes or cute language when money, access, work, data, or trust is at risk. Use descriptive help links, not "Learn more." Specific messages must still be understandable and actionable.

Challenge generic messages. Use a fallback only when the cause is unknown, with a next step or exit. Instrument errors, review them one month after launch, and keep improving them.

Prioritize frequent or blocking errors. Track triggers, affected state, frequency, owners, fixes, and follow-up dates. Review copy and implemented behavior together.
