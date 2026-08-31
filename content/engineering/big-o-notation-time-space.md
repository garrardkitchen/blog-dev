---
title: "Big O Notation: The Shape of Time and Space"
description: "A practical guide to Big O notation with tested C# and Python examples for runtime performance, memory complexity, measurement, and choosing the right trade-off."
date: 2026-07-24
tags:
  - Algorithms
  - Big O
  - C#
  - Python
  - Performance
  - Computer Science
---

# Big O Notation: The Shape of Time and Space

## What you will learn—and why it matters

By the end of this article, you will be able to:

- explain Big O notation without reaching for a textbook;
- recognise `O(1)`, `O(log n)`, `O(n)`, `O(n log n)`, and `O(n²)`;
- analyse both **runtime** and **memory** growth;
- compare working C# and Python implementations of the same ideas;
- avoid the mistakes that make Big O sound more precise than it is; and
- choose an algorithm by understanding its trade-offs rather than memorising a chart.

Why does this matter? A program that feels instant with 100 records may become unusable with one million. Faster hardware can reduce how long each operation takes, but it does not change how the number of operations grows. Big O gives us a shared language for describing that growth before production traffic, data, and cost expose it for us.

One small correction before we begin: it is **Big O**, using the letter O for “order,” not Big Zero. Big O describes an asymptotic upper bound: beyond some input size, the algorithm’s growth is bounded above by that rate, ignoring constant factors.

> [!TIP]
> **Think in shapes, not stopwatch readings.** Big O does not ask, “How many milliseconds did this take?” It asks, “What happens to the work when the input grows?”

![A plotter-style chart comparing the growth of common Big O complexity classes and showing what happens when the input doubles](/blog/img/big-o-growth-curves.svg)

## The idea in one minute

Suppose `n` is the number of items an algorithm receives.

- If the algorithm performs the same amount of work regardless of `n`, it is `O(1)`.
- If doubling `n` adds roughly one more step, it is `O(log n)`.
- If doubling `n` roughly doubles the work, it is `O(n)`.
- If doubling `n` roughly quadruples the work, it is `O(n²)`.

Big O deliberately ignores constant multipliers and lower-order terms. If an algorithm performs `3n² + 20n + 500` operations, `n²` eventually dominates, so we call it `O(n²)`.

| Complexity | Common name | If `n` doubles | Typical example |
|---|---|---:|---|
| `O(1)` | Constant | No meaningful change | Array indexing |
| `O(log n)` | Logarithmic | About one extra step | Binary search |
| `O(n)` | Linear | About twice the work | Scan every item |
| `O(n log n)` | Linearithmic | A little more than twice | Merge sort |
| `O(n²)` | Quadratic | About four times the work | Compare every pair |
| `O(2ⁿ)` | Exponential | The work squares | Explore every subset-like choice |
| `O(n!)` | Factorial | Worse again | Try every ordering |

> [!TIP]
> At `n = 1,000,000`, a binary search needs at most about 20 comparisons. A linear scan may need one million. That gap is the practical meaning of growth rate.

## Big O is not a speed ranking

An `O(n)` algorithm can beat an `O(log n)` algorithm for small inputs if its individual operations are cheaper. An `O(n²)` solution may be the simplest and safest choice when `n` can never exceed ten. Big O tells us how algorithms **scale**; benchmarks tell us how particular implementations behave on particular hardware.

We should also state which case we are analysing:

- **Worst case:** the maximum work for an input of size `n`;
- **average case:** the expected work under stated assumptions; and
- **best case:** the minimum possible work.

When people give one complexity without qualification, they usually mean the worst case. That convention is useful, but it is not a substitute for saying what assumptions apply.

Big O need not be the tightest possible bound. A linear algorithm is technically also `O(n²)`, but `O(n)` is the more informative upper bound. Big Theta (`Θ`) describes a tight asymptotic bound: the growth is bounded both above and below by the same rate. In everyday engineering, people often say “Big O” when they informally mean “the growth rate.” The distinction matters in formal analysis; in a design review, state the tightest useful bound and its assumptions.

## Runtime complexity in C#

The methods below progress from constant time to quadratic time. The comments describe worst-case runtime and **auxiliary space**—memory created by the algorithm in addition to its input.

```csharp
namespace BigONotation;

public static class RuntimeExamples
{
    // O(1) time, O(1) auxiliary space.
    public static int FirstOrDefault(ReadOnlySpan<int> values) =>
        values.IsEmpty ? default : values[0];

    // O(log n) time, O(1) auxiliary space.
    // Precondition: values is sorted in ascending order.
    public static int BinarySearch(ReadOnlySpan<int> values, int target)
    {
        var low = 0;
        var high = values.Length - 1;

        while (low <= high)
        {
            // Avoids the overflow risk of (low + high) / 2.
            var middle = low + ((high - low) / 2);
            var candidate = values[middle];

            if (candidate == target)
            {
                return middle;
            }

            if (candidate < target)
            {
                low = middle + 1;
            }
            else
            {
                high = middle - 1;
            }
        }

        return -1;
    }

    // O(n) time, O(1) auxiliary space.
    public static long Sum(ReadOnlySpan<int> values)
    {
        long total = 0;
        foreach (var value in values)
        {
            total += value;
        }

        return total;
    }

    // O(n²) time, O(1) auxiliary space.
    public static bool ContainsDuplicateSlow(ReadOnlySpan<int> values)
    {
        for (var left = 0; left < values.Length; left++)
        {
            for (var right = left + 1; right < values.Length; right++)
            {
                if (values[left] == values[right])
                {
                    return true;
                }
            }
        }

        return false;
    }
}
```

`ReadOnlySpan<int>` is intentional: spans provide constant-time length and indexing. An arbitrary `IReadOnlyList<T>` implementation does not promise the same performance. Binary search discards half of the remaining input after each comparison. For `n` items, the number of halvings is approximately `log₂(n)`, which gives us `O(log n)`.

The duplicate check is different. In the worst case, each item is compared with nearly every item after it:

```text
(n - 1) + (n - 2) + ... + 1 = n(n - 1) / 2
```

That expression is dominated by `n²`, so the algorithm is `O(n²)`.

> [!TIP]
> Sequential loops do not automatically multiply. Two separate loops over `n` items are `O(n + n)`, simplified to `O(n)`. Nested loops are often `O(n²)`, but only when both ranges grow with `n`.

### C# unit tests

Complexity is an analytical property, but correctness still needs executable tests. These tests cover empty inputs, boundaries, successful searches, missing values, and duplicate detection.

```csharp
using BigONotation;
using Xunit;

namespace BigONotation.Tests;

public sealed class RuntimeExamplesTests
{
    [Fact]
    public void FirstOrDefault_ReturnsDefaultForEmptyInput()
    {
        Assert.Equal(0, RuntimeExamples.FirstOrDefault([]));
    }

    public static TheoryData<int[], int, int> SearchCases => new()
    {
        { [1], 1, 0 },
        { [1, 3, 5, 7, 9], 1, 0 },
        { [1, 3, 5, 7, 9], 5, 2 },
        { [1, 3, 5, 7, 9], 9, 4 },
        { [1, 3, 5, 7, 9], 4, -1 }
    };

    [Theory]
    [MemberData(nameof(SearchCases))]
    public void BinarySearch_ReturnsExpectedIndex(
        int[] values,
        int target,
        int expected)
    {
        Assert.Equal(expected, RuntimeExamples.BinarySearch(values, target));
    }

    [Fact]
    public void Sum_UsesLongToAvoidIntOverflow()
    {
        Assert.Equal(4_294_967_294L, RuntimeExamples.Sum(
            [int.MaxValue, int.MaxValue]));
    }

    public static TheoryData<int[], bool> DuplicateCases => new()
    {
        { [1, 2, 3], false },
        { [1, 2, 1], true },
        { [], false }
    };

    [Theory]
    [MemberData(nameof(DuplicateCases))]
    public void ContainsDuplicateSlow_ReturnsExpectedResult(
        int[] values,
        bool expected)
    {
        Assert.Equal(expected, RuntimeExamples.ContainsDuplicateSlow(values));
    }
}
```

## Runtime complexity in Python

Here are the same ideas in Python. Type hints make the contracts visible, while the algorithms remain idiomatic. Their stated complexities assume a built-in sequence such as `list` or `tuple`, where `len()` and indexing are constant-time operations.

```python
from collections.abc import Sequence


def first_or_default(values: Sequence[int]) -> int:
    """O(1) time and O(1) auxiliary space for a built-in sequence."""
    return values[0] if values else 0


def binary_search(values: Sequence[int], target: int) -> int:
    """O(log n) time and O(1) auxiliary space.

    The input must be a constant-time-indexed sequence sorted in
    ascending order.
    """
    low = 0
    high = len(values) - 1

    while low <= high:
        middle = low + (high - low) // 2
        candidate = values[middle]

        if candidate == target:
            return middle
        if candidate < target:
            low = middle + 1
        else:
            high = middle - 1

    return -1


def sum_values(values: Sequence[int]) -> int:
    """O(n) time and O(1) auxiliary space for bounded-size integers."""
    total = 0
    for value in values:
        total += value
    return total


def contains_duplicate_slow(values: Sequence[int]) -> bool:
    """O(n²) time and O(1) auxiliary space for a built-in sequence."""
    for left in range(len(values)):
        for right in range(left + 1, len(values)):
            if values[left] == values[right]:
                return True
    return False
```

This introductory analysis treats each integer comparison and addition as constant time. Python integers have arbitrary precision, so the cost and storage of arithmetic also grow when the integers themselves become extremely large. If numeric magnitude is part of the input, include its bit length in the analysis.

### Python unit tests

The standard library is enough to test every path in the example:

```python
import unittest

from big_o_examples import (
    binary_search,
    contains_duplicate_slow,
    first_or_default,
    sum_values,
)


class RuntimeExamplesTests(unittest.TestCase):
    def test_first_or_default_with_empty_input(self) -> None:
        self.assertEqual(0, first_or_default([]))

    def test_binary_search_boundaries_and_missing_value(self) -> None:
        values = [1, 3, 5, 7, 9]
        self.assertEqual(0, binary_search(values, 1))
        self.assertEqual(2, binary_search(values, 5))
        self.assertEqual(4, binary_search(values, 9))
        self.assertEqual(-1, binary_search(values, 4))

    def test_sum_values(self) -> None:
        self.assertEqual(12, sum_values([3, 4, 5]))

    def test_contains_duplicate_slow(self) -> None:
        self.assertFalse(contains_duplicate_slow([]))
        self.assertFalse(contains_duplicate_slow([1, 2, 3]))
        self.assertTrue(contains_duplicate_slow([1, 2, 1]))


if __name__ == "__main__":
    unittest.main()
```

## Space complexity: memory grows too

Runtime is only half the story. **Space complexity** describes how memory use grows with the input. Usually we discuss auxiliary space separately from the memory already occupied by the input and output.

Consider duplicate detection again. A hash set can trade memory for speed:

![An algorithm-lab diagram comparing pairwise duplicate checks with a hash set, including their time and auxiliary-space costs](/blog/img//big-o-time-space-tradeoff.svg)

### C#: constant space versus linear space

```csharp
namespace BigONotation;

public static class SpaceExamples
{
    // O(n²) time, O(1) auxiliary space.
    public static bool ContainsDuplicateWithoutBuffer(
        ReadOnlySpan<int> values)
    {
        for (var left = 0; left < values.Length; left++)
        {
            for (var right = left + 1; right < values.Length; right++)
            {
                if (values[left] == values[right])
                {
                    return true;
                }
            }
        }

        return false;
    }

    // O(n) expected time, O(n) auxiliary space.
    public static bool ContainsDuplicateWithSet(IEnumerable<int> values)
    {
        var seen = new HashSet<int>();

        foreach (var value in values)
        {
            if (!seen.Add(value))
            {
                return true;
            }
        }

        return false;
    }
}
```

For typical hash behaviour, `HashSet<T>.Add` is constant time on average. We perform it at most `n` times, giving expected `O(n)` time, but the set may store `n` values, giving `O(n)` auxiliary space. Poor collision behaviour can degrade lookup and insertion, so “expected” and the quality of the key’s hash implementation are important.

```csharp
using BigONotation;
using Xunit;

namespace BigONotation.Tests;

public sealed class SpaceExamplesTests
{
    public static TheoryData<int[], bool> Cases => new()
    {
        { [], false },
        { [42], false },
        { [1, 2, 3, 4], false },
        { [1, 2, 3, 1], true }
    };

    [Theory]
    [MemberData(nameof(Cases))]
    public void BothImplementationsAgree(int[] values, bool expected)
    {
        Assert.Equal(
            expected,
            SpaceExamples.ContainsDuplicateWithoutBuffer(values));
        Assert.Equal(
            expected,
            SpaceExamples.ContainsDuplicateWithSet(values));
    }
}
```

### Python: constant space versus linear space

```python
from collections.abc import Iterable, Sequence


def contains_duplicate_without_buffer(values: Sequence[int]) -> bool:
    """O(n²) time and O(1) auxiliary space."""
    for left in range(len(values)):
        for right in range(left + 1, len(values)):
            if values[left] == values[right]:
                return True
    return False


def contains_duplicate_with_set(values: Iterable[int]) -> bool:
    """O(n) expected time and O(n) auxiliary space."""
    seen: set[int] = set()
    for value in values:
        if value in seen:
            return True
        seen.add(value)
    return False
```

```python
import unittest

from space_examples import (
    contains_duplicate_with_set,
    contains_duplicate_without_buffer,
)


class SpaceExamplesTests(unittest.TestCase):
    def test_both_implementations_agree(self) -> None:
        cases = (
            ([], False),
            ([42], False),
            ([1, 2, 3, 4], False),
            ([1, 2, 3, 1], True),
        )

        for values, expected in cases:
            with self.subTest(values=values):
                self.assertEqual(
                    expected,
                    contains_duplicate_without_buffer(values),
                )
                self.assertEqual(
                    expected,
                    contains_duplicate_with_set(values),
                )


if __name__ == "__main__":
    unittest.main()
```

> [!TIP]
> **Auxiliary space is not total space.** If a function receives a list of `n` items, the list already costs `O(n)`. An in-place scan can still use `O(1)` auxiliary space because it creates no structure that grows with `n`.

## Measuring time and memory without fooling yourself

Big O comes from analysing the algorithm, not from fitting a label to one benchmark. Measurement is still valuable: it reveals constants, allocations, cache behaviour, runtime warm-up, and real-world input effects that asymptotic notation intentionally hides.

### A small C# measurement harness

This uses only the .NET runtime. It warms up each path, forces garbage collection before allocation measurement, and consumes results so the work remains observable. Static timestamp APIs avoid allocating a `Stopwatch` object inside the region being measured.

```csharp
using System.Diagnostics;
using BigONotation;

var sizes = new[] { 100, 1_000, 10_000 };

foreach (var size in sizes)
{
    var values = Enumerable.Range(0, size).ToArray();

    // Warm up JIT compilation before timing.
    _ = SpaceExamples.ContainsDuplicateWithSet(values);

    GC.Collect();
    GC.WaitForPendingFinalizers();
    GC.Collect();

    var allocatedBefore = GC.GetAllocatedBytesForCurrentThread();
    var startedAt = Stopwatch.GetTimestamp();

    var hasDuplicate = SpaceExamples.ContainsDuplicateWithSet(values);

    var allocatedAfter = GC.GetAllocatedBytesForCurrentThread();
    var elapsed = Stopwatch.GetElapsedTime(startedAt);
    var allocatedBytes = allocatedAfter - allocatedBefore;

    Console.WriteLine(
        $"n={size,6} duplicate={hasDuplicate,-5} " +
        $"elapsed={elapsed.TotalMilliseconds,8:F3} ms " +
        $"allocated={allocatedBytes,10:N0} bytes");
}
```

This is educational rather than a statistically rigorous microbenchmark. For production optimisation, isolate the benchmark, use representative data, run enough iterations, compare distributions rather than a single reading, and profile the complete request path.

### A small Python measurement harness

Python’s standard library provides `perf_counter()` for high-resolution elapsed time and `tracemalloc` for traced Python allocations:

```python
import time
import tracemalloc

from space_examples import contains_duplicate_with_set


def measure(size: int) -> tuple[float, int, bool]:
    values = list(range(size))

    # Warm up the code path before measuring it.
    _ = contains_duplicate_with_set(values)

    tracemalloc.start()
    started_at = time.perf_counter()
    result = contains_duplicate_with_set(values)
    elapsed_seconds = time.perf_counter() - started_at
    _, peak_bytes = tracemalloc.get_traced_memory()
    tracemalloc.stop()

    return elapsed_seconds, peak_bytes, result


for input_size in (100, 1_000, 10_000):
    elapsed, peak, has_duplicate = measure(input_size)
    print(
        f"n={input_size:6} duplicate={has_duplicate!s:5} "
        f"elapsed={elapsed * 1_000:8.3f} ms "
        f"peak={peak:10,d} bytes"
    )
```

`tracemalloc` observes traced Python allocations, not every byte used by the process or native libraries. It also adds measurement overhead. Measurement tools have boundaries; good conclusions state them.

> [!TIP]
> Test several input sizes and look at the **ratio**. If doubling `n` repeatedly moves the duration toward four times as much, quadratic behaviour may be surfacing. Noise at small sizes is normal.

## How to analyse unfamiliar code

Use this repeatable process:

1. Define the input size. Is `n` the number of users, edges, bytes, or rows?
2. Identify the dominant repeated operation.
3. Count how that operation grows with the input.
4. Examine nested loops carefully: do they both depend on `n`?
5. Include recursion depth and the work performed at each level.
6. Track data structures that grow with the input.
7. State best, average, or worst case and any preconditions.
8. Simplify constants and lower-order terms only after writing the full expression.

For two independent inputs, keep both variables. Comparing every user with every permission is `O(u × p)`, not automatically `O(n²)`. Preserving meaningful variables produces a more useful design conversation.

## Common traps

### Dropping costs too early

A loop that performs an `O(n)` search on every iteration is `O(n²)`, even if the loop body looks like one line.

### Treating all collection operations as constant time

Indexing an array is `O(1)`. Looking up a hash key is typically expected `O(1)`. Searching an unsorted list is `O(n)` in the worst case. Inserting into the middle of an array-backed list is `O(n)` because elements must move.

### Ignoring hidden allocations

A tidy chain of mapping, filtering, sorting, and materialising may create several intermediate collections. The time may still be acceptable while the allocation rate creates garbage-collection pressure.

### Optimising without a constraint

Replacing readable code with a complex structure to save microseconds can reduce reliability. Ask what input size is realistic, what latency or memory budget exists, and where profiling shows the bottleneck.

> [!TIP]
> The best algorithm is constrained by more than asymptotic speed: correctness, clarity, memory, input distribution, latency targets, security, and operational cost all belong in the decision.

## A practical decision guide

When choosing between solutions, ask:

- What is the maximum credible `n`, not just today’s average?
- Is latency, throughput, or memory the harder constraint?
- Can preprocessing make later lookups cheaper?
- Is the input already sorted, or would sorting add `O(n log n)` work?
- Are hash-table average-case assumptions acceptable?
- Can the data be streamed instead of materialised?
- Does a database or platform service already provide an indexed operation?
- Is the simpler algorithm comfortably inside the performance budget?

Big O is most powerful when it changes a question. Instead of asking, “Is this code fast?”, ask, “How does its cost grow, under which assumptions, and where will that growth cross our budget?”

## The shape of a decision

Big O notation does not predict the future in milliseconds. It reveals the direction in which the future is moving.

Every algorithm is a quiet agreement with scale: an agreement about how much time we will ask from people, how much memory we will ask from machines, and how much complexity we will ask future maintainers to carry. Small inputs can hide that agreement, but growth eventually reads the fine print.

So perhaps the profound lesson of Big O is not that infinity is large. It is that **the shape of our choices outlives the size of today’s problem**.
