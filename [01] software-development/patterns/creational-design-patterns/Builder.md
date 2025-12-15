# Builder Pattern

## Short Description

**Builder** separates the construction of a complex object from its representation so the same construction process can create different representations. It provides a fluent, step-by-step API to assemble objects with many optional parts while keeping the final object immutable and readable.

---

## Comprehensive Overview

The Builder pattern (from the Gang of Four) addresses object construction complexity by moving the details of assembling parts into a separate object — the *Builder*. The Builder exposes methods for setting parts (often returning the builder for chaining), and a final `build()` method returns the finished product.

Key ideas:

* Keep the *Product* immutable and free of construction clutter.
* Put construction logic (validation, defaults, complex assembly) in the Builder.
* Optionally use a *Director* to orchestrate a sequence of building steps when multiple standard variants exist.
* Variations include *step builders* (typed builders that force calling methods in a given order) and builders implemented differently depending on language features (e.g., records, named arguments).

Analogy: think of ordering a custom sandwich — you specify bread, spreads, toppings one step at a time. The kitchen (Builder) assembles the sandwich and hands you a boxed finished product (Product).

---

## Problems this topic solves

* **Telescoping constructors:** avoids many overloaded constructors with long parameter lists.
* **Clarity:** shows required vs optional parameters through builder API rather than positional args.
* **Immutability:** supports creating immutable final objects with all fields set once.
* **Complex setup:** centralizes validation and assembly steps in one place.
* **Multiple representations:** same build algorithm can produce different outputs (e.g., JSON, XML, object graph).

---

## Limitations — what it does *not* solve

* Not ideal for very simple objects (adds boilerplate).
* Doesn't magically remove complexity in construction logic — it relocates it.
* If validation rules are complex and scattered, a naive Builder can become large and hard to maintain.
* Builders are usually mutable and not thread-safe — using them concurrently without care is unsafe.

---

## When to use

* Object has many optional parameters or config options.
* You want a clear, readable object construction DSL for clients.
* You need immutable objects with validated construction.
* You want to support several product variants built by the same procedure.

---

## When to avoid

* Object is simple (few fields) or language provides good named/default arguments and immutability constructs.
* You would introduce unnecessary classes and complexity for little benefit.
* You cannot centralize or limit builder responsibilities — builder becomes a “god class”.

---

## Implementation Examples

> The following examples implement the same conceptual product (`NutritionFacts`) in each language. Each example uses a Builder that enforces required fields, provides fluent optional setters, validates in `build()`, and creates an immutable final object.

### C#

```csharp
// C# example (works in modern .NET versions)
// Product: NutritionFacts (immutable)
// Builder: nested class `Builder` providing fluent setters and Build().

using System;

namespace DesignPatterns.Examples
{
    // Immutable product
    public sealed class NutritionFacts
    {
        public int Servings { get; }
        public int Calories { get; }
        public int Fat { get; }
        public int Sodium { get; }
        public int Carbohydrate { get; }

        // Private constructor: only Builder can create an instance
        private NutritionFacts(Builder builder)
        {
            Servings = builder._servings;
            Calories = builder._calories;
            Fat = builder._fat;
            Sodium = builder._sodium;
            Carbohydrate = builder._carbohydrate;
        }

        // Nested Builder class (encapsulates construction)
        public sealed class Builder
        {
            // Required parameters
            internal int _servings;
            internal int _calories;

            // Optional parameters with defaults
            internal int _fat = 0;
            internal int _sodium = 0;
            internal int _carbohydrate = 0;

            // Constructor enforces required fields
            public Builder(int servings, int calories)
            {
                if (servings <= 0) throw new ArgumentException("servings must be > 0");
                if (calories < 0) throw new ArgumentException("calories must be >= 0");
                _servings = servings;
                _calories = calories;
            }

            // Fluent setters for optional parameters
            public Builder WithFat(int fat) { _fat = fat; return this; }
            public Builder WithSodium(int sodium) { _sodium = sodium; return this; }
            public Builder WithCarbohydrate(int carbohydrate) { _carbohydrate = carbohydrate; return this; }

            // Final validation and object creation
            public NutritionFacts Build()
            {
                // Example invariant: no negative nutrients
                if (_fat < 0 || _sodium < 0 || _carbohydrate < 0)
                    throw new InvalidOperationException("nutrient values cannot be negative");

                return new NutritionFacts(this);
            }
        }

        // Optional: override ToString for easy debugging
        public override string ToString()
        {
            return $"NutritionFacts(servings={Servings}, calories={Calories}, fat={Fat}, sodium={Sodium}, carbohydrate={Carbohydrate})";
        }
    }

    // Usage
    // var facts = new NutritionFacts.Builder(1, 200)
    //               .WithFat(10)
    //               .WithSodium(35)
    //               .Build();
}
```

**Notes (C#):**

* Nesting the Builder keeps it logically associated with the Product and hides builder internals.
* `Build()` performs final validation before producing an immutable instance.
* Use XML comments or README to document which parameters are required/optional.

---

### Java 21

```java
// Java 21 example using `record` for an immutable product.
// Product: NutritionFacts (record)
// Builder: static nested Builder class that constructs the record.

import java.util.Objects;

public record NutritionFacts(int servings, int calories, int fat, int sodium, int carbohydrate) {

    // The Builder encapsulates construction and validation
    public static final class Builder {
        // Required
        private final int servings;
        private final int calories;

        // Optional with defaults
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        // Enforce required parameters at builder creation
        public Builder(int servings, int calories) {
            if (servings <= 0) throw new IllegalArgumentException("servings must be > 0");
            if (calories < 0) throw new IllegalArgumentException("calories must be >= 0");
            this.servings = servings;
            this.calories = calories;
        }

        // Fluent setters
        public Builder fat(int fat) { this.fat = fat; return this; }
        public Builder sodium(int sodium) { this.sodium = sodium; return this; }
        public Builder carbohydrate(int carbohydrate) { this.carbohydrate = carbohydrate; return this; }

        // Build + validate final invariants
        public NutritionFacts build() {
            if (fat < 0 || sodium < 0 || carbohydrate < 0) {
                throw new IllegalStateException("nutrient values cannot be negative");
            }
            // Use record constructor for immutability
            return new NutritionFacts(servings, calories, fat, sodium, carbohydrate);
        }
    }

    // Optional: convenient static factory for the builder
    public static Builder builder(int servings, int calories) {
        return new Builder(servings, calories);
    }
}

// Usage:
// NutritionFacts facts = NutritionFacts.builder(2, 250)
//                                .fat(5)
//                                .sodium(20)
//                                .build();
```

**Notes (Java 21):**

* Using `record` makes the Product clearly immutable and concise.
* Builder validates before calling the record constructor.
* `builder(...)` static factory is ergonomic for clients.

---

### Python 13

```python
# Python 3.13+ example using dataclasses for an immutable product.
# Product: NutritionFacts (frozen dataclass)
# Builder: NutritionFactsBuilder with fluent setters.

from dataclasses import dataclass
from typing import Final

@dataclass(frozen=True)
class NutritionFacts:
    servings: Final[int]
    calories: Final[int]
    fat: Final[int]
    sodium: Final[int]
    carbohydrate: Final[int]

class NutritionFactsBuilder:
    """
    Builder for NutritionFacts.
    Required: servings, calories (provided at builder creation).
    Optional: fat, sodium, carbohydrate with sensible defaults.
    """
    def __init__(self, servings: int, calories: int) -> None:
        if servings <= 0:
            raise ValueError("servings must be > 0")
        if calories < 0:
            raise ValueError("calories must be >= 0")

        self._servings = servings
        self._calories = calories
        self._fat = 0
        self._sodium = 0
        self._carbohydrate = 0

    # Fluent setters
    def set_fat(self, fat: int) -> "NutritionFactsBuilder":
        self._fat = fat
        return self

    def set_sodium(self, sodium: int) -> "NutritionFactsBuilder":
        self._sodium = sodium
        return self

    def set_carbohydrate(self, carbohydrate: int) -> "NutritionFactsBuilder":
        self._carbohydrate = carbohydrate
        return self

    # Final construction + validation
    def build(self) -> NutritionFacts:
        if self._fat < 0 or self._sodium < 0 or self._carbohydrate < 0:
            raise ValueError("nutrient values cannot be negative")
        # dataclass is frozen => immutable product
        return NutritionFacts(
            servings = self._servings,
            calories = self._calories,
            fat = self._fat,
            sodium = self._sodium,
            carbohydrate = self._carbohydrate
        )

# Usage:
# builder = NutritionFactsBuilder(1, 180)
# facts = builder.set_fat(8).set_sodium(30).build()
# print(facts)
```

**Notes (Python):**

* `dataclass(frozen=True)` yields immutability.
* Builder is a normal class (Python rarely nests builders).
* Type hints and `Final` help convey immutability intent.

---

## Instructions for AIs (Knowledge Source Usage)

**When an AI coding assistant *should* use Builder:**

* Generating code for complex config objects (many options) where named parameters or defaults aren't sufficient or would produce messy signatures.
* When the product must be **immutable** and must be produced only after validation.
* When creating a fluent, discoverable API for domain-heavy constructs (e.g., query builders, complex requests, UI components).

**When an AI coding assistant *should not* use Builder:**

* For trivial objects with 2–3 fields where constructors or language-native defaults are simpler and clearer.
* When the target language has excellent named args/defaults that read clearly (e.g., Python dataclasses with defaults, Kotlin named args). In that case prefer idiomatic constructs.
* When the project policy forbids builder proliferation (e.g., strict class-count limits in a constrained environment).

**How Builder relates to other patterns / concepts:**

* **Factory / Static Factory:** simpler creation, not stepwise. Use Factory for family selection, Builder for stepwise assembly.
* **Abstract Factory:** for families of related products — different purpose.
* **Prototype:** cloning an existing object instead of constructing from scratch.
* **Step/Typed Builder:** variant that enforces calling order via types / interfaces — useful to guarantee required arguments at compile time.
* **Fluent Interfaces:** Builder commonly uses fluent method chaining — but keep the API discoverable and not ambiguous.

**Cares & considerations when implementing:**

* Validate required parameters and invariants *in `build()`* — that centralizes checks. Fail fast with clear exceptions.
* Keep builder methods minimal and purposeful — avoid one method per tiny field if it will explode API surface. Group logically related options.
* Document defaults and required fields in code comments and public docs; developers expect clear guidance.
* Avoid exposing internal builder state after `build()`; consider making builder unusable after `build()` if reusing could create subtle bugs.
* Do not make builders globally shared mutable objects — typically builders are *not* thread safe. If thread-safety is required, construct new builders per thread or make them immutable.

**Common pitfalls / anti-patterns:**

* Overuse: creating builders for every tiny class adds boilerplate and cognitive load.
* Allowing invalid states by splitting validation across many methods instead of centralizing it in `build()`.
* Making Builder logic too smart (e.g., heavy algorithms) — Builder should orchestrate assembly; complex algorithms might belong to another class/service.
* Using Director for simple cases — Director is helpful when multiple standard products share the build sequence; otherwise it often adds unnecessary indirection.

---

## References and Bibliography

* Gamma, E., Helm, R., Johnson, R., & Vlissides, J. *Design Patterns: Elements of Reusable Object-Oriented Software*. Addison-Wesley, 1994.
* Bloch, J. *Effective Java*, 3rd Edition. Addison-Wesley, 2018 — discussion on builders and static factories.
* Refactoring.Guru — Builder Pattern. [https://refactoring.guru/design-patterns/builder](https://refactoring.guru/design-patterns/builder)
* Oracle Java Documentation — Records & Immutability. [https://docs.oracle.com/en/java/](https://docs.oracle.com/en/java/)
* Microsoft Docs — Patterns and practices (C#). [https://learn.microsoft.com/](https://learn.microsoft.com/) (search "Builder pattern C#")
* Martin Fowler — Patterns of Enterprise Application Architecture (concepts about construction and factories). [https://martinfowler.com/](https://martinfowler.com/)
* Wikipedia — Builder pattern. [https://en.wikipedia.org/wiki/Builder_pattern](https://en.wikipedia.org/wiki/Builder_pattern)

---

## Quick Summary / Mnemonic (for students)

**"Build last — configure first."**
Put configuration steps in a Builder. Validate in `build()`. Keep the final object immutable.

---