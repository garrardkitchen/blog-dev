---
title: "36 C# Interview Questions, Explained by When They Matter"
description: "A practical, grouped guide to 36 C# interview questions covering types, async, concurrency, memory, and deployment."
date: 2026-08-02
draft: false
featured: true
tags: ["engineering", "csharp", "dotnet", "interviews", "performance", "async"]
---

Strong C# answers connect language mechanics to the code you would choose in production. This guide groups 36 common interview questions into five useful areas: type semantics, modern abstractions, asynchronous work, concurrency, and memory/runtime behaviour.

You will learn what the runtime is doing, when each feature earns its place, and the trade-offs worth saying out loud in an interview. The payoff is fewer accidental allocations, safer concurrent code, more predictable services, and better deployment choices.

> [!Tip]
> A memorable interview answer has three parts: **what it is**, **when I use it**, and **the benefit or trade-off**. That turns trivia into engineering judgement.

## 1. Types, equality, strings, and time

### 1. What is boxing, when does it happen implicitly, and what does it cost?

**Boxing** copies a value type (`int`, `DateTime`, struct) into an object on the managed heap. It happens when a value is converted to `object`, an interface, or a non-generic API that expects an object.

```csharp
int count = 42;
object boxed = count;                 // Boxing: heap allocation and copy
int again = (int)boxed;               // Unboxing: type check and copy
```

Use generic collections such as `List<int>` instead of `ArrayList`. The benefit is type safety and avoiding allocation/GC pressure in hot paths.

### 2. Why can a struct implementing an interface get boxed, and what does it cost?

An interface variable needs an object reference. Assigning a struct to it traditionally boxes the struct; invoking through a constrained generic call can avoid it.

```csharp
public readonly struct Meter : IFormattable
{
    public string ToString(string? format, IFormatProvider? provider) => "1 m";
    public override string ToString() => "1 m";
}

IFormattable text = new Meter();       // Usually boxes

static string Format<T>(T value) where T : IFormattable
    => value.ToString(null, null);     // Constrained call; avoids boxing
```

Use constrained generics in reusable, performance-sensitive code. The benefit is interface-shaped APIs without an allocation per call.

### 3. What is a `ref struct`, and what restrictions apply?

A `ref struct` is stack-only and can safely hold managed references or stack memory. `Span<T>` is the flagship example. It cannot be boxed, stored in a class field, captured by a lambda, or live across an `await`/`yield` suspension point.

```csharp
Span<byte> header = stackalloc byte[16];
header[0] = 0xCA;
```

Use it for short-lived parsing and buffer work. The benefit is fast, allocation-free slices; the restriction protects the GC from a reference escaping the stack.

### 4. How does `record struct` differ from `record class` in semantics and performance?

`record class` is an immutable-friendly **reference type**; copies share the same object until `with` creates another. `record struct` is a **value type**; assignment copies its value. Both provide value-based equality by default.

```csharp
public record class CustomerId(string Value);
public readonly record struct Point(int X, int Y);
```

Use a record class for identities and messages passed around an application; use a small record struct for compact values such as coordinates. The benefit is concise value equality—though a large struct can be slower because it is copied.

### 5. What breaks if you override `Equals` but not `GetHashCode`?

Hash-based collections first choose a bucket from `GetHashCode`, then compare with `Equals`. Equal values that return different hashes may not be found.

```csharp
public sealed class OrderKey
{
    public string Value { get; init; } = "";
    public override bool Equals(object? obj) =>
        obj is OrderKey other && Value == other.Value;

    public override int GetHashCode() => StringComparer.Ordinal.GetHashCode(Value);
}
```

Whenever you define value equality for a key used by `Dictionary<TKey, TValue>` or `HashSet<T>`, override both. The benefit is correct lookup behaviour and good hash distribution.

> [!Warning]
> Never mutate fields that participate in equality while an object is a dictionary key. The key can become effectively lost in the wrong bucket.

### 6. Why does `==` behave differently for `string` variables and object-typed strings?

Overload resolution happens from the variables' **compile-time** types. `string == string` uses string value equality; `object == object` uses reference equality unless an operator is available for those static types.

```csharp
string left = new('x', 1);
string right = new('x', 1);
object a = left;
object b = right;

bool valuesMatch = left == right; // true
bool refsMatch = a == b;          // false
```

Use `string.Equals(a, b, StringComparison.Ordinal)` when the comparison rule should be explicit. The benefit is predictable, culture-safe code.

### 7. What is string interning, and when are identical strings not reference-equal?

Interning keeps one shared instance for equal string literals (and strings explicitly interned). Equal strings created at runtime are usually separate objects.

```csharp
string literal = "cloud";
string built = new string("cloud".ToCharArray());

bool sameText = literal == built;                         // true
bool sameObject = ReferenceEquals(literal, built);         // false
```

Rely on value equality, never reference equality, for string content. Explicit interning is a niche optimisation for a bounded, repeated vocabulary; an unbounded input set can retain memory for the process lifetime.

### 8. What is the difference between `float`, `double`, and `decimal`, and which should you use for money?

`float` is ~7 decimal digits and 32-bit; `double` is ~15–16 digits and 64-bit. Both use binary floating point, so many decimal fractions are approximations. `decimal` is base-10-oriented, 128-bit, and has 28–29 significant digits.

```csharp
decimal total = 19.99m * 3m; // Money: decimal suffix is required
```

Use `double` for scientific, graphics, telemetry, and most numerical work; use `decimal` for money where decimal rounding rules matter. The benefit is choosing either speed/range or predictable base-10 arithmetic deliberately.

### 9. What is the difference between `DateTime`, `DateTimeOffset`, `DateOnly`, and `TimeOnly`?

| Type | Represents | Good use |
|---|---|---|
| `DateTime` | Date/time plus a `Kind` hint; not a time-zone rule | Local, short-lived application logic |
| `DateTimeOffset` | Instant plus UTC offset | Persisted timestamps, APIs, audit events |
| `DateOnly` | Calendar date only | Birthdays, billing dates |
| `TimeOnly` | Time of day only | Opening hours, schedules |

```csharp
DateTimeOffset occurredAt = DateTimeOffset.UtcNow;
DateOnly renewalDate = new(2026, 8, 1);
```

Use `DateTimeOffset` for an event that occurred at a point in time. The benefit is preserving the offset; use a time-zone identifier as well when you must apply future DST rules.

## 2. Generics and modern abstraction

### 10. How does the JIT specialise generic code for value types versus reference types?

For reference-type arguments, the runtime can commonly share one generic implementation because references have the same representation. For value types, differing sizes and calling conventions usually require specialised native code.

```csharp
static T Max<T>(T left, T right) where T : IComparable<T>
    => left.CompareTo(right) >= 0 ? left : right;
```

Use generics rather than `object`-based APIs. The benefit is static type safety and value-type code that can avoid boxing, balanced against more generated code for many value-type instantiations.

### 11. Explain covariance and contravariance with generics.

Covariance (`out`) lets a producer of a more-derived type stand in for a producer of a base type. Contravariance (`in`) lets a consumer of a base type stand in for a consumer of a more-derived type.

```csharp
IEnumerable<string> names = ["Ada"];
IEnumerable<object> items = names;          // out T

Action<object> log = value => Console.WriteLine(value);
Action<string> logName = log;                // in T
```

Use covariance for read-only outputs and contravariance for handlers/consumers. The benefit is flexible APIs without casts; mutable collections cannot safely be covariant.

### 12. Why are arrays covariant in C#?

It is a historical CLR compatibility feature: a `string[]` can be assigned to `object[]`. The runtime must then check every write, because storing a non-string object would violate the actual array type.

```csharp
object[] values = new string[1];
// values[0] = 42; // ArrayTypeMismatchException at runtime
```

Prefer `IReadOnlyList<out T>` for covariant reads or `List<T>` for mutable collections. The benefit is compile-time safety and no runtime store check.

### 13. What are static abstract interface members?

They let an interface require static operations that a generic type can call. They power generic math without reflection or runtime type switches.

```csharp
static T Add<T>(T left, T right) where T : System.Numerics.IAdditionOperators<T, T, T>
    => left + right;

int total = Add(2, 3);
```

Use them for generic numeric, unit, or domain-value algorithms. The benefit is compile-time checked polymorphism over static members.

## 3. Async: work, continuations, cancellation, and failures

### 14. Does `async` create a new thread? What happens at `await`?

No. `async` is a control-flow feature, not a thread factory. At an incomplete `await`, the method registers a continuation and returns its `Task` to the caller; a later completion schedules the continuation. I/O can wait without occupying a thread.

```csharp
public static async Task<string> GetPageAsync(HttpClient client, Uri uri)
    => await client.GetStringAsync(uri);
```

Use async for I/O-bound web calls, database calls, and file access. The benefit is scalable services that do not block a thread while waiting.

### 15. What state machine does the compiler generate for an async method?

The compiler rewrites an `async` method into an `IAsyncStateMachine` with fields for its state, awaiters, locals that survive suspension, and an async method builder. `MoveNext` advances execution before and after each await.

```csharp
public static async Task<int> ReadLengthAsync(Stream stream)
{
    byte[] buffer = new byte[64];
    return await stream.ReadAsync(buffer);
}
```

This explains why locals can survive an await and why an async method that completes synchronously can be very cheap. The benefit of knowing it is avoiding needless `async`/`await` wrappers and state that crosses awaits.

### 16. What is the difference between `Task<T>` and `ValueTask<T>`?

`Task<T>` is simple, reusable, and can be awaited many times. `ValueTask<T>` can hold either a result directly or wrap a task, avoiding an allocation when synchronous completion is common—but it has stricter consumption rules.

```csharp
public ValueTask<int> TryReadAsync()
    => _cachedValue is { } value
        ? ValueTask.FromResult(value)
        : new ValueTask<int>(ReadFromStoreAsync());
```

Use `ValueTask<T>` only on a measured, hot API where synchronous completion is frequent. The benefit can be fewer allocations; otherwise `Task<T>` is the safer default. Do not await the same `ValueTask` repeatedly or combine it freely with `WhenAll` unless you first call `AsTask()`.

### 17. Why is `async void` dangerous?

It gives the caller no task to await, observe for failure, compose, or cancel. Exceptions escape to the active synchronization context and can terminate the process.

```csharp
private async void SaveButton_Click(object? sender, EventArgs e)
{
    await SaveAsync(); // Event handlers are the intentional exception
}
```

Use `async void` only for event-handler signatures that require it. Return `Task` everywhere else. The benefit is reliable error handling and testability.

### 18. Why can calling `.Result` or `.Wait()` deadlock with a `SynchronizationContext`?

On a UI or legacy ASP.NET context, an awaited continuation may be queued back to the context. If that same context thread blocks on `.Result`, it cannot run the continuation: both wait forever.

```csharp
// Avoid on a context-bound thread:
// string body = client.GetStringAsync(uri).Result;
string body = await client.GetStringAsync(uri);
```

Use “async all the way”. The benefit is deadlock-free responsiveness and no blocked worker thread. Modern ASP.NET Core normally has no custom `SynchronizationContext`, but blocking is still wasteful and risks thread-pool starvation.

### 19. What does `ConfigureAwait(false)` actually do?

It tells an await not to capture the current `SynchronizationContext` (or a non-default task scheduler) for its continuation. It does **not** create a thread, guarantee a background thread, or make a call faster by magic.

```csharp
string body = await client.GetStringAsync(uri).ConfigureAwait(false);
```

Use it in reusable libraries that do not need to return to a UI context. The benefit is avoiding unnecessary context hops and deadlock-prone coupling. In ASP.NET Core it usually changes little because there is no request synchronization context to capture.

### 20. What is the real cost of `Task.Run` in a server application?

`Task.Run` queues CPU work to the thread pool. It adds scheduling overhead and consumes a worker thread; it does not make I/O asynchronous and it does not create a dedicated thread per request.

```csharp
// Sensible only for brief, genuinely CPU-bound work.
int result = await Task.Run(() => CalculateScore(input));
```

For I/O, call the provider's async API directly. For sustained CPU work, use bounded background workers/queues or dedicated compute. The benefit is preventing request threads from being blocked; indiscriminate `Task.Run` merely moves the queue and may exhaust the pool.

### 21. What is thread-pool starvation, and how do you diagnose it?

Starvation occurs when pool threads block or run long work faster than the pool can supply healthy throughput. Symptoms include queued requests, rising latency, low CPU, and slow timers. Sync-over-async and blocking I/O are common causes.

```csharp
// A starvation smell in request handling:
// return repository.GetAsync(id).Result;
return await repository.GetAsync(id);
```

Diagnose with `dotnet-counters monitor System.Runtime`, traces (`dotnet-trace`/OpenTelemetry), thread-pool queue length, and blocked stacks. The benefit is fixing the actual blocker—usually by making I/O async or bounding work—not blindly raising minimum thread counts.

### 22. How do you use `CancellationToken` correctly?

Accept it at operation boundaries, pass it to every cancellable downstream API, honour it at sensible checkpoints, and let `OperationCanceledException` propagate rather than converting normal cancellation into a failure.

```csharp
public async Task ImportAsync(Stream source, CancellationToken cancellationToken)
{
    using StreamReader reader = new(source);
    while (await reader.ReadLineAsync(cancellationToken) is { } line)
    {
        cancellationToken.ThrowIfCancellationRequested();
        await ProcessAsync(line, cancellationToken);
    }
}
```

Use the request-aborted token in APIs and a service-stop token in workers. The benefit is freeing resources promptly when the caller no longer needs the result.

### 23. How does exception handling differ between `Task.WhenAll` and a single `await`?

`await` on a single task rethrows that task's original exception. `await Task.WhenAll(...)` completes only when all inputs finish, then rethrows one observed failure; the returned `WhenAll` task's `Exception` is an `AggregateException` containing all failures.

```csharp
Task all = Task.WhenAll(ImportAAsync(), ImportBAsync());
try
{
    await all;
}
catch
{
    foreach (Exception error in all.Exception?.InnerExceptions ?? [])
        Console.Error.WriteLine(error);
}
```

Use `WhenAll` for independent fan-out work. The benefit is concurrent latency, while retaining a deliberate place to inspect every failure.

> [!Note]
> `WhenAll` does not cancel its sibling tasks when one fails. Add cooperative cancellation yourself if fail-fast behaviour is required.

## 4. Concurrency and coordination

### 24. What does `lock` compile to, and how does C# 13's `System.Threading.Lock` improve it?

Classic `lock (gate)` is lowered to `Monitor.Enter` in a `try`/`finally`, ensuring `Monitor.Exit` runs. With a `System.Threading.Lock` instance, C# 13 recognises the type and uses its scope-based enter API instead of `Monitor`.

```csharp
private readonly System.Threading.Lock _gate = new();
private int _count;

public void Increment()
{
    lock (_gate)
        _count++;
}
```

Use a private, dedicated lock object for short, synchronous critical sections. `Lock` provides clearer intent and can improve performance; never lock on public objects, strings, or `this`, and never `await` inside a lock.

### 25. What guarantees does `volatile` give, and when do you need `Interlocked` instead?

`volatile` gives visibility and ordering for individual reads/writes: a volatile write is released and a volatile read is acquired. It does not make compound work such as `count++` atomic.

```csharp
private volatile bool _stopping;
private int _count;

public void Increment() => Interlocked.Increment(ref _count);
```

Use `volatile` for a simple state flag; use `Interlocked` for atomic counters, exchanges, and compare-and-swap updates. The benefit is correct lock-free coordination without losing increments.

### 26. When would you use `Channel<T>` over `BlockingCollection<T>`?

`Channel<T>` is designed for asynchronous producer/consumer pipelines, supports bounded backpressure, and does not block threads while waiting. `BlockingCollection<T>` is a synchronous, blocking collection from the pre-async era.

```csharp
Channel<string> queue = Channel.CreateBounded<string>(100);
await queue.Writer.WriteAsync("job", cancellationToken);
string job = await queue.Reader.ReadAsync(cancellationToken);
```

Use `Channel<T>` for background queues in ASP.NET Core or hosted services. The benefit is natural async flow control and better scalability under load.

### 27. What is the difference between `AsyncLocal<T>` and `ThreadLocal<T>`?

`ThreadLocal<T>` stores one value per physical thread. `AsyncLocal<T>` flows with the logical async execution context across awaits (and normally into child tasks), even when the continuation uses another thread.

```csharp
private static readonly AsyncLocal<string?> CorrelationId = new();

CorrelationId.Value = "req-42";
await Task.Delay(1);
Console.WriteLine(CorrelationId.Value); // req-42
```

Use `AsyncLocal<T>` sparingly for correlation, tracing, or ambient request context; use `ThreadLocal<T>` for per-thread caches. The benefit is the right lifetime model—while avoiding hidden dependencies and execution-context flow overhead.

## 5. Memory management and deployment

### 28. Explain GC generations and object promotion—why is a Gen 2 collection expensive?

New small objects start in Gen 0. Survivors are promoted to Gen 1, then Gen 2, based on the idea that most objects die young. A Gen 2 collection has to examine a much larger, long-lived heap and can be more disruptive.

```csharp
// Prefer releasing large, long-lived references when finished.
_cachedReport = null;
```

Use allocation profiling before tuning. The benefit is targeting retained objects and allocation rate, rather than forcing `GC.Collect()`—which usually makes matters worse.

### 29. What is the Large Object Heap (LOH), and why does it matter for long-running services?

Large allocations (roughly 85,000 bytes or more) go to the LOH. They are collected with Gen 2 and historically are not compacted by default, so repeated temporary large buffers can cause memory pressure and fragmentation.

```csharp
byte[] buffer = ArrayPool<byte>.Shared.Rent(128 * 1024);
try { /* use buffer */ }
finally { ArrayPool<byte>.Shared.Return(buffer); }
```

Use streaming, chunking, or pooling for large transient payloads. The benefit is less LOH churn and steadier memory in long-running services.

### 30. How does a finalizer affect object lifetime, and why call `Dispose` and `GC.SuppressFinalize`?

An object with a finalizer survives at least until finalization is queued and completed, often promoting it and delaying memory reclamation. `Dispose` releases scarce unmanaged resources deterministically; `GC.SuppressFinalize(this)` avoids needless finalizer work after that.

```csharp
public sealed class NativeHandleOwner : IDisposable
{
    public void Dispose()
    {
        // Release unmanaged resource.
        GC.SuppressFinalize(this);
    }

    ~NativeHandleOwner() { /* Last-resort cleanup only. */ }
}
```

Prefer `SafeHandle` rather than hand-written finalizers. The benefit is prompt release of handles and far less finalization complexity.

### 31. What is the difference between Workstation GC and Server GC?

Workstation GC is tuned for responsive client applications. Server GC uses multiple managed heaps and parallel collection to favour throughput on multi-core servers, at a higher memory cost.

```xml
<PropertyGroup>
  <ServerGarbageCollection>true</ServerGarbageCollection>
</PropertyGroup>
```

Use Server GC for sustained, multi-core backend workloads and evaluate it with real load. The benefit is throughput; a small service or memory-constrained container may not benefit from extra heaps.

### 32. What problem does `Span<T>` solve, and where does it not eliminate allocations?

`Span<T>` provides a cheap view over contiguous memory—arrays, stack buffers, native memory, or strings via `ReadOnlySpan<char>`—so slicing/parsing need not create arrays or substrings.

```csharp
ReadOnlySpan<char> code = "uk-azure-42";
ReadOnlySpan<char> region = code[..2]; // View only; no substring allocation
```

Use it in parsers, serializers, and protocol code. The benefit is fewer intermediate allocations. It cannot make the original array/string allocation disappear, cannot outlive its backing memory, and cannot cross an async suspension point.

### 33. When should you use `ArrayPool<T>`?

Use it when profiling shows frequent, short-lived arrays—especially medium/large buffers—are driving GC cost. Rent, use only the required segment, and return it in `finally`.

```csharp
byte[] rented = ArrayPool<byte>.Shared.Rent(4096);
try
{
    int read = await stream.ReadAsync(rented.AsMemory(0, 4096));
    await destination.WriteAsync(rented.AsMemory(0, read));
}
finally
{
    ArrayPool<byte>.Shared.Return(rented, clearArray: true);
}
```

The benefit is lower allocation rate and fewer Gen 2/LOH collections. Do not retain or expose a returned buffer; clearing is important for sensitive data but costs CPU.

### 34. How do closures capture variables?

A lambda that uses an outer local captures the **variable**, not a frozen value. The compiler typically creates a hidden object to hold it, which can extend lifetime and allocate.

```csharp
var actions = new List<Action>();
for (int i = 0; i < 3; i++)
{
    int copy = i;
    actions.Add(() => Console.WriteLine(copy));
}
```

Use a loop-local copy when creating callbacks, and prefer static lambdas where no capture is required. The benefit is correct callback behaviour and fewer accidental allocations/lifetime leaks.

### 35. What is deferred execution in LINQ?

Most LINQ operators return a recipe, not results. The query runs each time it is enumerated, observing the source's current state and repeating any work or side effects.

```csharp
IEnumerable<string> query = users.Where(user => user.IsActive);
List<string> snapshot = query.ToList(); // Materialises once
```

Use deferred execution for composable in-memory pipelines. Materialise with `ToList`, `ToArray`, or an async equivalent when you need a snapshot, multiple enumeration, or to close a database context. The benefit is lazy work; the risk is surprise repeat work.

### 36. What does Native AOT give you, and what does it not support well?

Native AOT compiles an application ahead of time into native code. It can improve startup, reduce deployment size, and lower memory use—excellent for CLI tools, serverless cold starts, and small services.

```xml
<PropertyGroup>
  <PublishAot>true</PublishAot>
</PropertyGroup>
```

It is not a transparent switch for reflection-heavy or runtime-code-generation designs. Dynamic loading, unrestricted reflection, `Reflection.Emit`, and some serializers/ORMs need source generation, trimming annotations, or redesign. The benefit is a more predictable deployment; the trade-off is declaring what your application needs at build time.

> [!Tip]
> Treat Native AOT as an architectural feedback loop: source-generated metadata and explicit dependencies often make the application clearer even when AOT is not the final deployment target.

## Closing thought

The mature C# engineer is not the person who can recite every runtime rule. It is the person who can look at a feature and ask: **what work, allocation, ownership, or dependency is this hiding—and is that trade worth paying?**

That question turns language knowledge into systems that remain calm under load, understandable under change, and trustworthy when the easy path stops being easy.

## Alignment: adaptive intelligence

These patterns make a useful foundation for adaptive intelligence: propagating cancellation, correlation, and bounded work gives an AI-enabled service the feedback loops and operational discipline to learn and adjust without becoming an unbounded consumer of threads, memory, or trust.
