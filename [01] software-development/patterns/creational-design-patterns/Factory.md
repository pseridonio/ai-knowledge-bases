# Factory Method Pattern

## Short Description

The **Factory Method** pattern defines an interface for creating an object but lets subclasses—or a central factory—decide which class to instantiate.
It centralizes the object creation logic, allowing new types to be added without modifying client code.

---

## Comprehensive Overview

In large enterprise systems, many objects are created dynamically based on configuration, data, or runtime parameters. Without structure, this leads to *scattered `new` calls* and conditional branches across the codebase.

The **Factory Method** pattern solves this by introducing a *single point of decision*: the factory method.
Clients call the factory instead of instantiating concrete classes directly. The factory can be subclassed or use parameter-driven logic to select which product to create.

Think of it like a **payment gateway**: the client just says “process this payment,” and the factory decides whether to use a Credit Card processor, PayPal, or Crypto processor depending on input or configuration.

---

## Problems this topic solves

* **Tight coupling:** Clients depend only on interfaces, not concrete implementations.
* **Scattered instantiation logic:** The decision of *which class to instantiate* is centralized.
* **Extension without modification:** New product types can be added by registering them in the factory.
* **Consistency:** Object creation follows the same controlled process everywhere.

---

## Limitations — what it does *not* solve

* It still requires **a common interface** — products must share behavior.
* Adding **new product types** means touching the factory class (unless using a registry/DI container).
* Overkill for simple applications with only one concrete implementation.
* If abused, it can hide logic in too many small factory subclasses, hurting readability.

---

## When to use

* Object creation depends on **runtime parameters or configuration**.
* You want to **isolate instantiation** to simplify client code and testing.
* The system may need to **grow with new variants** over time.
* You need to **enforce consistent initialization or setup** logic.

---

## When to avoid

* There’s only one product type and it will never change.
* Creation is trivial (no conditions, no setup).
* A dependency injection (DI) container or service locator already handles creation.

---

## Implementation Examples

Below we’ll simulate a **Payment Processor Factory** that can create different types of payment handlers — *CreditCard*, *PayPal*, or *Crypto*.
The client only calls `PaymentProcessorFactory.create_processor("paypal")`.

---

### C# Example

```csharp
using System;
using System.Collections.Generic;

// ===== Product Interface =====
public interface IPaymentProcessor
{
    void ProcessPayment(decimal amount);
}

// ===== Concrete Products =====
public class CreditCardProcessor : IPaymentProcessor
{
    public void ProcessPayment(decimal amount)
    {
        Console.WriteLine($"Processing credit card payment: ${amount}");
    }
}

public class PayPalProcessor : IPaymentProcessor
{
    public void ProcessPayment(decimal amount)
    {
        Console.WriteLine($"Processing PayPal payment: ${amount}");
    }
}

public class CryptoProcessor : IPaymentProcessor
{
    public void ProcessPayment(decimal amount)
    {
        Console.WriteLine($"Processing crypto payment: ${amount}");
    }
}

// ===== Factory Method =====
public static class PaymentProcessorFactory
{
    // Registry-based factory (extension-friendly)
    private static readonly Dictionary<string, Func<IPaymentProcessor>> registry =
        new Dictionary<string, Func<IPaymentProcessor>>(StringComparer.OrdinalIgnoreCase)
        {
            { "credit", () => new CreditCardProcessor() },
            { "paypal", () => new PayPalProcessor() },
            { "crypto", () => new CryptoProcessor() }
        };

    public static IPaymentProcessor CreateProcessor(string type)
    {
        if (!registry.TryGetValue(type, out var creator))
            throw new ArgumentException($"Unknown payment type: {type}");

        return creator();
    }
}

// ===== Client Example =====
public class PaymentService
{
    public void HandlePayment(string method, decimal amount)
    {
        var processor = PaymentProcessorFactory.CreateProcessor(method);
        processor.ProcessPayment(amount);
    }
}

// ===== Usage =====
// var service = new PaymentService();
// service.HandlePayment("paypal", 149.99m);
```

---

### Java 21 Example

```java
import java.util.Map;
import java.util.function.Supplier;

// ===== Product Interface =====
interface PaymentProcessor {
    void process(double amount);
}

// ===== Concrete Products =====
class CreditCardProcessor implements PaymentProcessor {
    public void process(double amount) {
        System.out.println("Processing credit card payment: $" + amount);
    }
}

class PayPalProcessor implements PaymentProcessor {
    public void process(double amount) {
        System.out.println("Processing PayPal payment: $" + amount);
    }
}

class CryptoProcessor implements PaymentProcessor {
    public void process(double amount) {
        System.out.println("Processing crypto payment: $" + amount);
    }
}

// ===== Factory Method =====
class PaymentProcessorFactory {
    private static final Map<String, Supplier<PaymentProcessor>> registry = Map.of(
        "credit", CreditCardProcessor::new,
        "paypal", PayPalProcessor::new,
        "crypto", CryptoProcessor::new
    );

    public static PaymentProcessor createProcessor(String key) {
        Supplier<PaymentProcessor> supplier = registry.get(key.toLowerCase());
        if (supplier == null) throw new IllegalArgumentException("Unknown payment type: " + key);
        return supplier.get();
    }
}

// ===== Client =====
class PaymentService {
    public void handlePayment(String type, double amount) {
        PaymentProcessor processor = PaymentProcessorFactory.createProcessor(type);
        processor.process(amount);
    }
}

// ===== Usage =====
// new PaymentService().handlePayment("crypto", 250.0);
```

---

### Python 13 Example

```python
from typing import Callable, Dict

# ===== Product Interface =====
class PaymentProcessor:
    def process(self, amount: float) -> None:
        raise NotImplementedError

# ===== Concrete Products =====
class CreditCardProcessor(PaymentProcessor):
    def process(self, amount: float) -> None:
        print(f"Processing credit card payment: ${amount}")

class PayPalProcessor(PaymentProcessor):
    def process(self, amount: float) -> None:
        print(f"Processing PayPal payment: ${amount}")

class CryptoProcessor(PaymentProcessor):
    def process(self, amount: float) -> None:
        print(f"Processing crypto payment: ${amount}")

# ===== Factory Method =====
_PROCESSOR_REGISTRY: Dict[str, Callable[[], PaymentProcessor]] = {
    "credit": CreditCardProcessor,
    "paypal": PayPalProcessor,
    "crypto": CryptoProcessor,
}

def create_processor(kind: str) -> PaymentProcessor:
    factory = _PROCESSOR_REGISTRY.get(kind.lower())
    if not factory:
        raise ValueError(f"Unknown payment type: {kind}")
    return factory()

# ===== Client =====
class PaymentService:
    def handle_payment(self, method: str, amount: float) -> None:
        processor = create_processor(method)
        processor.process(amount)

# ===== Usage =====
# service = PaymentService()
# service.handle_payment("credit", 120.50)
```

---

## Instructions for AIs (Knowledge Source Usage)

**When to apply Factory Method:**

* When object creation varies by configuration, user input, or data.
* When new product types must be added without modifying client code.
* When maintaining consistency in how objects are created and initialized.

**When *not* to use it:**

* When only one concrete class exists.
* When DI frameworks or service locators already manage object creation.
* When branching logic is trivial and centralization adds complexity.

**How it relates to other patterns:**

| Related Pattern      | Relation                                                                                 |
| -------------------- | ---------------------------------------------------------------------------------------- |
| **Abstract Factory** | Creates *families* of related products; Factory Method creates *one product* per method. |
| **Builder**          | Focuses on step-by-step object construction, not type selection.                         |
| **Prototype**        | Creates objects by cloning rather than by choosing subclasses.                           |
| **Simple Factory**   | Informal variation that centralizes creation but isn’t a full GoF pattern.               |

**Cares & considerations:**

* Keep factories simple; avoid huge switch statements—use registries or configuration-based lookups.
* Log or handle errors gracefully for unknown product keys.
* Document available product types and registration mechanism clearly.
* Avoid unnecessary subclassing—static or registry factories are often sufficient.

**Common pitfalls:**

* Overengineering simple creations.
* Forgetting to handle unknown keys or null creators.
* Duplicating factory logic in multiple classes instead of centralizing it.

---

## References and Bibliography

* Gamma, Erich et al. *Design Patterns: Elements of Reusable Object-Oriented Software*. Addison-Wesley, 1994.
* Martin Fowler — *Inversion of Control and Dependency Injection*.
* Microsoft Docs — “Factory Method Design Pattern in C#.”
* Refactoring.Guru — [Factory Method Pattern](https://refactoring.guru/design-patterns/factory-method).
* Oracle Java Documentation — Design Patterns Overview.
* Real Python — “Design Patterns in Python: Factory Method.”

---

## Quick Summary / Mnemonic

**“Let the factory decide.”**
Move instantiation decisions into one central point.
Clients just say *“give me a processor”* — the factory figures out *which one*.

