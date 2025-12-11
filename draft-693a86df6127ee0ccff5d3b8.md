---
title: "Treat Special cases explicitly but uniformly "
slug: treat-special-cases-explicitly-but-uniformly

---

Recently, I came across an [article](https://martinfowler.com/eaaDev/Range.html) by Martin Fowler about modeling a `Range`, which is a great example of enriching our domain with Value Objects.

The article discussed how much we can benefit from that Type, making our code easier and more importantly to get related types (begin and end of a range) together.

However, what I want to discuss today isn't how valuable this type is, but rather how to model it correctly. The seemingly simple Range type hides some interesting design challenges that can teach us about type design principles.

# Is it worth thinking too much?

At first glance, you could just say it is just two attributes, `begin` and `end`. so why bother?

Well at most times yes, but what about more special cases like an `OpenEndedRange`, where for example a task has `startDate` but no `dueDate`.

## Why not just make it null when no boundary?

You are again right, but just think of how bad this code can get if at each operation I am checking against the null. Additionally, this may work, but what if the requirements changed to have range with two ends open? You would need to make the other type nullable and check for nullability in each use case.

## I am convinced, but what to do then?

The main problem is the null-checking everywhere, so why not centralizing it?

We can solve this by introducing a `RangeBoundary` type, which is just a wrapper for the value. **Rich types again!**

# Encapsulate what varies

One of the principles of oop (and actually any good design regardless of the paradigm) is encapsulation. From the Range point of view, it shouldn't make a difference whether the value is an actual value or Infinity. This is what we mean by encapsulation.

So what does a range really need from some value? Typically just to compare them with each other (and probably other values too). Give the minimal working public API, no less, no more (YAGNI). So it's really obvious, we just need that boundary to be Comparable. This can be easily achieved by implementing the `Comparable` interface. Minimal working `RangeBound`

```csharp
public readonly record struct RangeBound<T> : IComparable<RangeBound<T>>
    where T : IComparable<T>
{
    private readonly T? _value;
    
    private readonly bool _isInfinity;

    private RangeBound(T? value, bool isInfinity) => 
        (_value, _isInfinity) = (value, isInfinity);
  
    public int CompareTo(RangeBound<T> other) =>   
       (_isInfinity, other._isInfinity)  switch
        {
            (true, true) => 0,     // Both infinity: equal
            (true, false) => 1,    // This is infinity, other is not: this is greater
            (false, true) => -1,   // Other is infinity, this is not: this is less
            (false, false) => _value!.CompareTo(other._value!) // Both concrete: compare values
        };

    
    public int CompareTo(T other) => CompareTo(ValueOf(other));
    public static RangeBound<T> Infinity() => new RangeBound<T>(default, true);
    public static RangeBound<T> ValueOf(T value) => new RangeBound<T>(value, false);
}
```

This code was forced to use additional `_isInfinity` boolean, cause we need it to work with Non-Reference types as `int` for example.

Maybe the most important take here is that we do not expose our constructor but rather static functions that holds the invariant of this class: `_value` *shall not be* `null` *if* `_isInfinity` *is* `false`.

Now let's see how Range uses this abstraction:

```csharp
public readonly record struct Range<T>
    where T : IComparable<T>
{
    public RangeBound<T> Start { get; }
    public RangeBound<T> End { get; }
    private Range(RangeBound<T> start, RangeBound<T> end)
    {
        if (start.CompareTo(end) > 0)
            throw new ArgumentException("Start must be less than or equal to End");
        Start = start;
        End = end;
    }

    public bool Contains(T value) =>
        Start.CompareTo(value) <= 0 && End.CompareTo(value) >= 0;
    
    public static Range<T> Create(T start, T end) =>
        new Range<T>(RangeBound<T>.ValueOf(start), RangeBound<T>.ValueOf(end));
    public static Range<T> OpenEndedRange(T start) =>
        new Range<T>(RangeBound<T>.ValueOf(start), RangeBound<T>.Infinity());
}
```

So now we can just use Range like so easily:

```csharp
var range = Range<int>.OpenEndedRange(5); // [5, inf)
Assert.True(range.Contains(10));

var range2 = Range<int>.Create(5, 10);
Assert.True(range2.Contains(7));
```

This implementation works, but let's explore what happens when requirements evolve.

## Problems?

Well, I honestly did not like the idea of that boolean + nullable T, but let me tell you first how it could get so messy faster than you think:

What if we had some requirement to have an unbounded Range? i.e, any value is valid. For example if some access control is valid at any point of time.

With the above design, you would need to add one more boolean to `RangeBound` to be able to express whether it is negative or positive infinity.

Although this would make the `CompareTo` method messier, it reveals a bigger design problem: this class violates the [Open/Closed Principle](https://en.wikipedia.org/wiki/Open%E2%80%93closed_principle). Every time we need a new type of boundary (like negative infinity), we must modify the `RangeBound` struct itself - adding new boolean fields and updating the `CompareTo` logic. The class isn't closed for modification; each new requirement forces us to change existing code, increasing the risk of bugs.

So how do we fix this? By making each type of boundary explicit instead of hiding them behind flags.

## Treat Special Values Explicitly

The core problem lies in how we're treating fundamentally different concepts as variations of the same thing. We're trying to handle `+Infinity`, `-Infinity`, and concrete values as if they were all the same type, just with different flags. But are they really the same?

Think about it:

* A concrete value has actual data that can be accessed
    
* Positive infinity is a conceptual boundary with no concrete value
    
* Negative infinity is similarly conceptual but in the opposite direction
    

These are three **distinct concepts** that happen to share a common interface (comparison). This is a perfect case for polymorphism!

### Making Each State Explicit

So instead of using boolean flags and nullable values, let's make each type of boundary explicit as its own class. We can use an abstract base class that each specific boundary type inherits from:

```csharp
public abstract record  RangeBound<T> : IComparable<RangeBound<T>>
    where T : IComparable<T>
{
    public abstract int CompareTo(RangeBound<T>? other);
    public int CompareTo(T other) => CompareTo(new Concrete<T>(other));

}
```

Now each type of boundary becomes its own class, handling its own comparison logic:

```csharp
public record Concrete<T>(T Value) : RangeBound<T>
    where T : IComparable<T>
{
    public override int CompareTo(RangeBound<T>? other) =>
        other switch
        {
            PositiveInfinity<T> => -1, 
            NegativeInfinity<T> => 1, 
            Concrete<T>{ Value : var otherVal} => Value.CompareTo(otherVal),
            _ => throw new NotImplementedException()
        };

}
public record  PositiveInfinity<T> : RangeBound<T>
    where T : IComparable<T>
{
    public override int CompareTo(RangeBound<T>? other) =>
        other switch
        {
            PositiveInfinity<T> => 0,
            NegativeInfinity<T> or Concrete<T> => 1,
            _ => throw new NotImplementedException(),
        };
}

public record NegativeInfinity<T> : RangeBound<T>
    where T : IComparable<T>
{
    public override int CompareTo(RangeBound<T>? other) =>
        other switch
        {
            NegativeInfinity<T> => 0,
            PositiveInfinity<T> or Concrete<T> => -1,
            _ => throw new NotImplementedException(),
        };  
}
```

This design is now open for extension. When we needed negative infinity, we simply added a new class without touching existing code. Each boundary type handles its own comparison logic: `Concrete` knows how to compare values, `PositiveInfinity` knows it's always greater than everything except itself, and `NegativeInfinity` knows it's always less. The `Range` class remains completely unchanged throughout these modifications. That's the power of encapsulation.

# Wrapping Up: The Evolution of Our Design

Throughout this article, we've evolved our Range type through three distinct approaches, each teaching us something valuable:

**Approach 1: Nullable values** Simple to start with, but forces null-checking everywhere and doesn't scale when requirements change.

**Approach 2: Boolean flags + nullable** Better encapsulation, but still violates the Open/Closed Principle. Every new boundary type requires modifying the struct and updating comparison logic.

**Approach 3: Explicit type hierarchy** Each boundary type is its own class with its own comparison logic. Open for extension, closed for modification. This is the practical solution in C#.

But there's an even better world: languages with native **sum types** (also called discriminated unions or algebraic data types). In Rust, F#, Haskell, or TypeScript, you can explicitly tell the compiler: "A `RangeBound` is exactly one of these three types, NOTHING else exists." The compiler then enforces exhaustiveness in pattern matching, eliminating the need for defensive `NotImplementedException` clauses.

In C#, we can approximate this with libraries like [OneOf](https://github.com/mcintyre321/OneOf), or hope for future language features. But our abstract class approach gets us 90% of the way there.

### Key Takeaways

* **Avoid representing distinct concepts with boolean flags**: If positive/negative infinity are real domain concepts, make them real types
    
* **Centralize validation and special case handling**: Don't scatter null-checks and flag-checks throughout your codebase
    
* **Encapsulation enables fearless refactoring**: Range didn't change at all when we completely rewrote RangeBound's internals
    
* **Design for extension**: The abstract class approach lets us add new boundary types without touching existing code
    
* **Type systems are design tools**: Use them to make invalid states unrepresentable, not just to satisfy the compiler
    

The next time you're tempted to add a boolean flag to represent different states, ask yourself: "Are these variations of one thing, or fundamentally different concepts that deserve their own types?"