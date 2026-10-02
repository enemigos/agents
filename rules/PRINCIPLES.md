# Principles

## Subtract Before You Add
When: Apply when sequencing an addition, refactor, or rewrite. Remove dead code, redundant validators, and stub references first, then build on the simpler base.

When evolving a system, remove complexity first, then build.

**Why:** Adding to a complex system compounds complexity. Removing first leaves less code, reveals the essential structure, and usually makes the next design obvious. Default to subtraction.

Make simplification a continual investment. Leave the design slightly simpler and more capable behind the same or smaller surface than you found it.

**The pattern:**
- Sequence removal before construction
- Cut before you polish (get to the minimum before investing in quality)
- Design for observed usage, not speculative edge cases
- No speculative validators, parsers, or guards beyond what the spec demands
- Simplify prompts (remove redundant instructions, excessive templates)
- When a reference has no novel content, delete it rather than leaving a stub

## Redesign From First Principles
When: Apply when integrating a new requirement into an existing design. Redesign as if the requirement had been a foundational assumption from day one, instead of bolting it on.

When integrating a change, don't bolt it onto the existing design. Redesign as if the requirement had been there from the start.

- Read all affected files and understand the current design
- Ask: "if we were writing this from scratch with this new requirement, what would we build?"
- Propagate the change through every reference: types, docs, examples, rationale sections
- Think about the whole redesign, then deliver it incrementally

This is the method for preserving option value when integrating changes into an existing design.

## Attack the Premise

When two or more fixes that share one premise have failed the same gate, suspect the premise, not the fixes.

**Why:** Each failure under a shared premise is evidence about the premise.

**Pattern:**
- **Write the premise down.** The premise is the one sentence that every failed fix assumed.
- **Take a census before the next fix.** Count the imbalance per actor. The census shows which actors hold the imbalance, not how large it is. Write the census as a rerunnable script.
- **Read the skew.** If the same few actors hold most of the imbalance on every run, something assigns them that role. Find what assigns the role. That assignment is the next "why" to trace to its root cause.
- **Remove the asymmetry instead of compensating for it**, per the [Laziness Protocol](#laziness-protocol). Rotate the role between actors, randomize the assignment, or move the role, so that no actor holds it on every run. A return path, a shared pool, a batched hand-off, or a periodic rebalance leaves the assignment in place and adds work on every run.

**Stop:**
- Do not start the next fix before the premise is written down and the census exists.
- If the census is even across actors, the premise is not the cause. Look for the cause elsewhere and keep the census as evidence.

This principle is distinct from [Redesign From First Principles](#redesign-from-first-principles), which rebuilds a design around a new requirement. It questions a fact the current design assumes.

## Laziness Protocol
When: Apply when refactoring, evaluating diff size, or tempted to add abstractions, layers, or signal threading. Bias toward deletion and the smallest change that solves the problem.

Aim for the most result with the least code and complexity.

- **Prefer deletion.** When asked to refactor or improve, look for removals before additions.
- **Maintain a flat call hierarchy.** Avoid deep call chains. A rich interface that hides substantial work is not a deep call chain. If answering a question requires tracing through more than 3 files or layers, flatten it.
- **Consolidate decisions.** Do not repeat the same choice in several places. Put it behind one source of truth and pass the result as a simple flag.
- **Minimize the diff.** Make the smallest change that solves the problem. Fewer lines beat "elegant" boilerplate.
- **Question the threading.** If a task asks you to pass a new signal through types, schemas, pipelines, or similar layers, stop and look for a more direct path.
- **Sweat the small leaks.** Remove tiny pass-throughs, representation leaks, and duplicated choices before they spread. Small leaks compound into permanent coordination costs.

**The test:** If a human developer would find the code exhausting to maintain, it is a bad solution.

## Model the Domain
When: Apply when writing stateful logic, or when code branches a lot or repeats a shape assumption across files. Encode the domain in a structure instead of scattered conditionals.

Encode the real domain in a data structure instead of scattering it across conditionals.

**Why:** Scattered booleans, repeated shape assumptions, and branching spread across files are accidental complexity. A structure that matches the domain makes invalid states unrepresentable and deletes branches. Choosing it at write time is cheap. Recovering it later reads as a refactor and gets deferred.

**Reach for structures like these:**

- A state machine instead of scattered booleans, phases, or lifecycle checks.
- A typed object/model instead of loose parameters or repeated shape assumptions.
- A map, registry, lookup table, or discriminated union instead of branching spread across files.
- A reducer or command/event model instead of ad hoc state mutations.
- A module organized around one body of domain knowledge instead of a sequence such as load, validate, transform, and save. Execution order is not ownership.
- A small module boundary that gathers repeated behavior, ownership, or invariants.
- A queue, cache, index, graph/tree, or normalized collection where the data access pattern calls for it.
- Any other structure that fits. When none fits, work out what the code must never allow and how the data gets read, then find the structure that encodes exactly that.

Do not force an abstraction. Prefer boring code if the current shape is already clear, local, and unlikely to grow. Be skeptical of an abstraction that adds indirection without removing branches, duplicated rules, invalid states, or lifecycle risk.

The sign that you skipped this is a new feature that grows an existing if/else chain by one more branch, or a second boolean that must stay in sync with the first. Temporal decomposition is another sign. Phase-named modules repeat the same domain rules across steps.

## Sequence Work Into Verifiable Units
When: Apply to multi-step work (sweeps, migrations, runs of similar edits) and to how you stack commits and PRs. Break work into small units that each end in a verifiable state, check each before the next, and order delivery so the sequence proves itself to a reviewer.

Order work as a sequence of small units, each ending in a state you can check, and don't advance until the current one is green.

**Why:** A break caught at the unit that caused it is cheap to localize. A break caught after a batch is buried, and you have already built further on a broken base. Sequencing those same units into a delivery a reviewer can replay turns "trust me" into "watch it go red, then green."

**Execution.** In a sweep, migration, or any run of similar edits, verify each change before starting the next. Each unit is a before/after bracket: known-good state, one change, run the check, then proceed. Rebase onto clean trunk first so every check measures against the real baseline. When a lever does the edits, the per-unit check is nearly free. Run it anyway.

**Delivery.** Stack commits and PRs in the order that proves the work. The canonical shape is the failing test first, then the fix on top. Other story orders are a subtraction before the reshape, a baseline capture before the treatment, the scaffold before the feature. Each commit lands on its own and the sequence reads as an argument.

**Pattern:**
- Pick the smallest unit that ends in a check: an edit plus its test, or a commit that stands alone.
- Verify before advancing. Red to green per unit, never deferred to a final batch.
- Order the units so the sequence builds confidence on its own, for you while executing and for a reviewer reading the stack.

Keep each check against the real artifact, and make the per-unit check cheap enough to run every time.

## Boundary Discipline

Place validation, type narrowing, and error handling at system boundaries. Trust internal code unconditionally. Business logic lives in pure functions. The shell is thin and mechanical.

**Why:** Scattered validation is noisy, redundant, and gives a false sense of safety. Keep logic out of framework wiring so it can be tested without the framework.

**The pattern:**
- **At boundaries** (CLI args, config files, external APIs, network protocols): validate, return errors, handle defensively.
- **Inside the system:** typed data, error propagation, no re-validation. Trust the types.
- **Across the boundary.** Expose domain concepts, not the boundary's private representation. Keep general-purpose mechanism inside and special-purpose policy at the edge.

**Applications:**

Validation and error handling:
- Validate config at parse time (the boundary), not inside business logic
- Parse raw data into domain types at the boundary
- Do not re-export transport, storage, framework, or wire types through the public surface
- No redundant nil checks deep in call chains if the boundary already validated

Code organization:
- Business logic in pure functions with no framework dependencies
- Parse functions: pure transforms from raw bytes to typed state
- Prompt construction: structured state in, string out
- Scoring and assessment: pure transforms from state to results

**The tests:**
- "Is this data crossing a system boundary right now?" If not, validation is redundant.
- "Can this be a pure function that the shell just calls?" If yes, extract it.

## Encode Lessons in Structure

Encode recurring fixes in mechanisms (tools, code, metadata, automation) instead of textual instructions. Every error, human correction, and unexpected outcome is a learning signal. Capture it, route it, and close the loop.

**Why:** Textual instructions are easy to miss. They require the reader to notice, remember, and comply. Structural mechanisms (lint rules, metadata flags, runtime checks, automation scripts) enforce the rule without cooperation.

**Pattern:**
When you catch yourself writing the same instruction a second time:
1. Ask: can this be a lint rule, a metadata flag, a runtime check, or a script?
2. If yes, encode it. Delete the instruction
3. If no (requires judgment), make the instruction more prominent and add an example of the failure mode

**Pick the strongest mechanism.** When more than one mechanism would work, choose the strongest the situation allows (an unrepresentable state that cannot compile, then a lint or banned API that fails CI, then a canonical helper, then a runtime check), because agents copy whatever the surrounding code already does and a weaker guard becomes the next template.

**Corollary:** If the fix is structural, only use the structural fix. The instruction is the symptom.

**Feedback loop:**
- **Capture every correction.** When the human intervenes or tests fail, decide if it's a one-off or a pattern.
- **Route to the right layer.** One-off -> session note. Recurring fix -> rule or lint. Systemic issue -> principle.
- **Close the loop.** Don't just record. Apply now or create a concrete todo.

**Anti-patterns:**
- Acknowledging without recording ("I'll keep that in mind" does not persist)
- Recording without routing (a note about a lint rule that should exist is wasted unless the lint rule gets implemented)
- Fixing without generalizing (fixing one instance while leaving the recurring pattern intact)

## Never Block on the Human

The human supervises asynchronously. Agents must stay unblocked. Make reasonable decisions, proceed, and let the human course-correct after the fact.

**Why:** Every permission pause stalls the pipeline and makes the human the bottleneck. Since code changes are reversible and reviewable, a wrong decision usually costs less than blocking.

**Pattern:**
- **Proceed, then present.** Do the work, show the result. Don't ask "should I do X?" Do X, explain why.
- **Reserve questions for genuine ambiguity.** Ask only when you cannot infer intent from context.
- **Make the system self-healing.** When you notice a problem, log it and fix it in the next round.
- **Supervision is async.** Design workflows for review-after-the-fact.

**Boundaries:**
- **Irreversible actions** (force-push, delete production data, send external messages) still require confirmation.
- **Reversible actions** (write code, edit notes, split tasks) should proceed without blocking.
- **Product direction** comes from the human. *Execution* should not block.

## Modules and abstractions

**Domain Module**, **Application Service Module**, and **Adapter Module** name responsibilities, not required folders, suffixes, or TypeScript constructs. A module may be a function, object, class, file, or package with a cohesive public interface. Use the roles at any scale; do not create three layers when the behavior does not need them.

The normal dependency and call flow for an operation with application policy or effects is:

```txt
external input -> inbound Adapter -> Application Service -> Domain Module
                                           |
                                           +-> application-owned port
                                                 -> outbound Adapter -> external system
```

An inbound Adapter may call a Domain Module directly only for a pure operation with no authorization, application policy, persistence, external calls, or effect sequencing:

```txt
external input -> inbound Adapter -> Domain Module
```

The composition root constructs concrete Adapters and supplies them to Application Services. Dependencies point inward: Domain Modules know neither services nor Adapters; Application Services know application-owned port contracts, not concrete technologies; Adapters depend on those contracts and translate at the edge.

### Choosing a role

Classify code by the responsibility that would make it change:

- A business meaning, invariant, calculation, or legal state transition changes: **Domain Module**.
- An application operation's policy, authorization, or effect sequence changes: **Application Service Module**.
- A protocol, framework, database, runtime, or third-party API changes: **Adapter Module**.
- Only construction, configuration, or resource wiring changes: **composition root**.

Split an abstraction when it owns more than one of these reasons to change. Do not split code merely to satisfy the taxonomy: a pure operation may need only a Domain Module, while a simple boundary may call an Application Service with no new domain type.

### Applying the roles in any codebase

For a new feature or a local refactor:

1. Trace one caller-visible operation from ingress to every effect.
2. Put intrinsic meanings, invariants, calculations, and transitions in Domain Modules.
3. Put application policy and effect ordering in an Application Service; define its dependencies as narrow ports.
4. Put each protocol or technology translation in an inbound or outbound Adapter.
5. Wire concrete Adapters to ports at the composition root.
6. Verify each role through its public seam: domain results, application outcomes, and boundary records/responses.

Apply these responsibilities inside the project's existing layout and framework vocabulary. Migrate mixed code only across the feature's required semantic surface; otherwise contain the old convention at an Adapter seam rather than forcing a broad rewrite.

For example, in password reset: `EmailAddress` and `ResetToken` are Domain Modules; `PasswordReset` is the Application Service; an HTTP route is an inbound Adapter; Postgres and email-provider implementations are outbound Adapters; bootstrap performs the wiring.

### Deep modules

A deep module hides substantial behavior, invariants, policy, sequencing, or translation behind a cohesive, low-burden interface. Low-burden does not necessarily mean few functions.

Avoid shallow abstractions that merely forward calls, mirror tables, rename another API, or expose implementation steps.

Use the deletion test:

- if deleting the module makes complexity disappear, it was probably pass-through waste
- if deleting it spreads complexity across callers, it was probably earning its keep

### Domain Modules

A **Domain Module** is a pure, type-centric abstract data type in the OCaml tradition. It centers one primary domain type or tightly related type family and owns what values mean and which operations are legal.

Use one when the code has a meaningful domain distinction, invariant, calculation, decision, or lifecycle. Keep a primitive or local pure function when introducing a domain abstraction would prevent no realistic misuse and centralize no meaningful rule.

A Domain Module should:

- co-locate its type, supporting types, parsers, smart constructors, combinators, predicates, legal transitions, domain projections, formatting, and test generators as applicable
- return refined values from parsers and constructors so callers cannot create invalid instances
- express expected failures as precise values
- remain deterministic and independent of I/O, frameworks, persistence, ambient time, randomness, and mutable global state

It may define pure permission decisions over parsed domain values. It should not authenticate callers, gather authorization context, enforce permissions while carrying out an application operation, choose effect order, query storage, call a network, or expose transport/persistence DTOs. Callers use its operations instead of recreating its checks or branding values with casts.

Example:

```ts
// email-address.ts

/** A parsed, normalized email address. */
export type EmailAddress = Brand<string, "EmailAddress">;

/** Parse an email address from untrusted input. */
export function parse(input: string): Result<EmailAddress, InvalidEmailAddress>;

/** Render an email address as a string. */
export function toString(email: EmailAddress): string;

/** Compare two email addresses for equality. */
export function equals(left: EmailAddress, right: EmailAddress): boolean;
```

Domain Modules may use plain functions, immutable value classes, or static-style classes when cohesive. If using classes:

- construct through `parse` / `make` / smart constructors
- make invalid instances unconstructable
- keep fields readonly/immutable from callers
- keep methods cohesive over that value
- do not hide dependencies or I/O inside domain value classes
- avoid inheritance for domain behavior

### Application Service Modules

An **Application Service Module** owns one cohesive application operation or capability, such as `PasswordReset`, `Invitations`, or `SubscriptionLifecycle`. It applies application policy and sequences effects through narrow, application-owned ports while delegating intrinsic business rules to Domain Modules.

Use one when an operation must coordinate authorization, domain decisions, persistence, external calls, transactions, messages, time, IDs, or telemetry, or when the same operation must be callable from multiple entrypoints. A direct Domain Module call is enough when no application policy or effect orchestration exists.

An Application Service should:

- accept and return application/domain types with precise expected-error unions
- define the smallest meaningful ports required by the operation
- receive ports, configuration, clocks, randomness, and similar capabilities explicitly
- own which effects occur, under what policy, and in what order
- remain independent of HTTP, CLI, queue, ORM, vendor SDK, and runtime types

It should not parse protocol envelopes, render responses, execute SQL, translate vendor DTOs, or duplicate Domain Module invariants. Prefer constructor injection for dependency-bearing classes; in Effect codebases, use services/tags/layers. Avoid dependency bags passed into every call.

There is no arbitrary method limit. Split methods that represent unrelated capabilities, change for different reasons, or require unrelated dependencies. Avoid vague names such as `Manager`, `Processor`, `Helper`, or generic `UserService` unless established by the project.

### Adapter Modules

An **Adapter Module** owns one boundary's translation and technology mechanics. Use one whenever application code crosses a framework, protocol, serialization, process, persistence, runtime, or third-party boundary.

There are two directions:

- An **inbound Adapter** parses an external request/event/command, invokes an Application Service or a directly callable pure Domain Module as described above, and projects its result into the external protocol. Examples: HTTP route, GraphQL resolver, CLI command, queue consumer.
- An **outbound Adapter** implements an Application Service port using a concrete technology and translates raw records, SDK values, and external failures into application/domain types and typed errors. Examples: Postgres store, Stripe client, email sender, system clock.

An Adapter should own schema/DTO translation, framework lifecycle, external error classification, and safe diagnostics for its boundary. It may retry a short-lived technical failure only when the operation is safely repeatable and the retry does not change the port's meaning. It should not decide business eligibility, authorization policy, legal state transitions, or application-operation ordering. Keep raw external types inside the Adapter or composition root.

A port is not an Adapter. A port is the application-owned contract that states what an operation needs; an outbound Adapter is one replaceable implementation. Do not add an Adapter that only forwards the same shape to another internal module without hiding real translation or mechanics.

### Composition root

The composition root parses environment and configuration, acquires resources, constructs concrete Adapters, and injects them into Application Services. Keep framework bindings and concrete wiring here; do not turn the composition root into a place for domain rules, application policy, or reusable boundary translation.

### Adapter reuse audit

Before creating a new adapter or service, agents must audit existing adapters/services.

Prefer, in order:

1. Reuse an existing adapter as-is through a narrow dependency type.
2. Extend an existing adapter if the new method fits its existing cohesive capability and changes for the same reason.
3. Create a new adapter only when reuse/extension would create bad coupling or an accidental interface.

Do not require an ADR for a routine feature-level Adapter or Application Service. Create an ADR when the new module introduces a lasting architectural boundary, shared pattern, provider strategy, or deliberate exception to these standards. The ADR should explain:

- what existing Adapters or Application Services were checked
- why reuse or extension did not fit
- why the new boundary or pattern is a separate cohesive capability

### Repositories and persistence

Avoid repository-per-table by default.

Repository-like adapters are acceptable when they represent a cohesive domain persistence capability. They should expose meaningful domain operations and return parsed domain types / typed errors, not raw rows and ORM errors.

Treat raw database rows and ORM models as infrastructure DTOs. Parse them before application/core logic. Keep SQL/ORM details inside infrastructure adapters or persistence modules.

## Functional core, imperative shell, and entrypoints

Domain Modules form the functional core. Application Service Modules and Adapter Modules form the imperative shell, but only Adapters contain technology-specific concerns. This keeps the same application operation reusable across REST, CLI, GraphQL, workers, and other entrypoints.

The functional core contains domain parsers, invariants, state transitions, calculations, combinators, and decision functions. It avoids I/O, hidden dependencies, ambient time/randomness, thrown expected failures, and framework-specific concerns.

The imperative shell has two distinct responsibilities:

- Application Services apply application policy and sequence effects through explicit ports.
- Adapters parse or project boundary values, classify external failures, and perform concrete I/O.

Entrypoint Adapters should be thin protocol translation layers. They parse protocol-specific input, call Domain Module parsers to obtain refined values, invoke an Application Service when application policy or effects are involved, and render protocol-specific output. A pure operation may call a Domain Module directly as described above. Do not duplicate business rules in controllers, resolvers, commands, or handlers.

Within authentication and authorization, inbound Adapters verify boundary credentials and produce a parsed identity such as `Principal`, `Session`, or `CommandActor`. Domain Modules may define pure permission decisions over parsed domain values. Application Services gather the required context and enforce those decisions while carrying out an application operation. Adapters project missing or invalid credentials and denied operations into protocol-specific outcomes; they do not define permission policy.

## Workflows, transactions, and idempotency

Use ordinary function calls or database transactions for simple single-boundary operations.

Use a saga or durable workflow when progress must survive process loss or redelivery, or when the operation requires long delays, compensation, resumability, timers, human approval, cross-service coordination, or multiple transaction boundaries. A short-lived retry by itself does not require durable workflow machinery.

Adapters own safe, short-lived technical retries. Application Services decide whether an application operation should be attempted again. Durable workflows own retries that must survive crashes, delays, or redelivery.

Do not hold database transactions open across network calls or long-running operations.

Any externally observable mutation or state transition that may be retried needs an explicit idempotency strategy:

- idempotency key
- natural unique constraint
- deduplication record
- state-machine transition guard
- transactional outbox/inbox

Retrying should not rely on "probably safe" side effects.

# Coding Standards

## HTTP APIs

Without an error schema, a caller must guess what failed. A structured error lets the caller fix the input and retry with intent.

- Each 4xx and 5xx response has a schema. 429 and 503 responses give `Retry-After` data. 400 responses identify invalid fields.
- All errors use one schema.
- Each error has a machine-readable code. Each schema has a human-readable message.
- Examples do not expose internal details.

Predictable names let callers infer paths safely. Inconsistent names force callers to memorize exceptions and can cause bugs.

- Resource paths use nouns, such as `/users` and `/orders`. Collection paths use plural names.
- All paths use one case. Prefer kebab-case.
- Use HTTP methods, not action verbs in URLs.

### Error Message Design

- Write error messages to help the reader understand and recover: say what happened, why it happened if known, what the impact is, and what to do next
- Prefer specific, concrete wording over vague or generic messages
- If the cause is unknown, say that plainly; do not invent false precision
- State what is still true or preserved, especially whether data, prior work, or system state remain intact
- Include the most useful recovery action or next diagnostic step
- Match detail to audience: user-facing errors should be plain and actionable; internal errors should include precise operational context needed for debugging
- Internal errors should name the failing operation, relevant identifiers, expected vs actual state when useful, and the most likely remediation path

## Sensitive data, telemetry, and debugging

Prefer end-to-end structured tracing across requests, jobs, workflows, application modules, adapters, and external calls.

Tracing/logging should make failures diagnosable with safe fields:

- domain IDs
- operation names
- dependency/provider names
- state tags
- retry counts
- typed error tags
- safe summaries

Do not put secrets in errors, traces, logs, or snapshots.

### Explicitness

Favor explicit, traceable code over implicit "magic". A reader (human or agent) should be able to follow where every name comes from without running tooling.

### Comments

Avoid code comments at all cost unless it's for JSDocs / Library documentation or a required runtime, compiler, or linter directive.