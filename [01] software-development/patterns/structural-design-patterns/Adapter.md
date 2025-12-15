# Adapter Pattern

## Short Description

The **Adapter** pattern allows incompatible interfaces to work together by wrapping one class with another that translates its interface into something the client expects.
It’s commonly used to integrate external libraries, legacy systems, or APIs into a uniform structure without changing their source code.

---

## Comprehensive Overview

In enterprise systems, teams often need to integrate multiple external systems that provide similar functionality but expose **different APIs** — for example, several payment gateways or data providers.

The **Adapter pattern** acts as a *translator* between your system and those external components.
Instead of rewriting your client code for each new API, you define a **common interface** (`IPaymentGateway`) and write one adapter per third-party API.
The adapter converts your system’s expected calls into the third party’s native method calls.

**Analogy:**
Think of a laptop charger adapter — it converts a plug shape or voltage to fit the outlet, without changing either the laptop or the wall socket.

This pattern promotes flexibility and decoupling: your core application code remains stable even when underlying integrations change.

---

## Problems this topic solves

* **API incompatibility:** Integrates third-party or legacy APIs without modifying their code.
* **Client code consistency:** The system interacts with a single unified interface.
* **Encapsulation of integration logic:** Keeps external API complexity out of business logic.
* **Improved testability:** You can mock adapters easily for unit testing.

---

## Limitations — what it does *not* solve

* Does not eliminate the need to understand both APIs (source and target).
* Adds an **extra layer of abstraction** — slightly more code and runtime indirection.
* Not useful if all systems already share a standard interface.
* Complex adapters can hide performance or behavioral differences between APIs.

---

## When to use

* Integrating third-party or legacy services with different APIs.
* Standardizing access to multiple providers of the same type (e.g., payment, storage).
* Migrating from one API to another gradually.
* Isolating client code from frequent API changes.

---

## When to avoid

* When you control all the code — just standardize interfaces directly.
* When adapting radically different semantics (e.g., sync vs. async) becomes too complex.
* When the adaptation logic becomes larger than the domain logic itself.

---

## Implementation Examples

We’ll simulate integrating **three payment gateways**:

* **StripeAPI** — modern interface.
* **PayPalService** — legacy interface.
* **LegacyGateway** — internal old API.

We’ll create an **Adapter Layer** that allows them to conform to one interface:
`IPaymentGateway → ProcessPayment(amount, currency)`.

---

### C# Example

```csharp
using System;

// ===== Target Interface =====
public interface IPaymentGateway
{
    void ProcessPayment(decimal amount, string currency);
}

// ===== Adaptees (Third-Party APIs) =====
public class StripeAPI
{
    public void MakePayment(decimal amount, string currency)
    {
        Console.WriteLine($"Stripe: charged {amount} {currency}");
    }
}

public class PayPalService
{
    public void SendPayment(double total, string curr)
    {
        Console.WriteLine($"PayPal: sent payment of {total} {curr}");
    }
}

public class LegacyGateway
{
    public void ExecuteTransaction(string data)
    {
        Console.WriteLine($"Legacy: executing transaction -> {data}");
    }
}

// ===== Adapters =====
public class StripeAdapter : IPaymentGateway
{
    private readonly StripeAPI _stripe;
    public StripeAdapter(StripeAPI stripe) => _stripe = stripe;

    public void ProcessPayment(decimal amount, string currency)
    {
        _stripe.MakePayment(amount, currency);
    }
}

public class PayPalAdapter : IPaymentGateway
{
    private readonly PayPalService _paypal;
    public PayPalAdapter(PayPalService paypal) => _paypal = paypal;

    public void ProcessPayment(decimal amount, string currency)
    {
        _paypal.SendPayment((double)amount, currency);
    }
}

public class LegacyAdapter : IPaymentGateway
{
    private readonly LegacyGateway _legacy;
    public LegacyAdapter(LegacyGateway legacy) => _legacy = legacy;

    public void ProcessPayment(decimal amount, string currency)
    {
        string data = $"{amount}:{currency}";
        _legacy.ExecuteTransaction(data);
    }
}

// ===== Client =====
public class PaymentProcessor
{
    private readonly IPaymentGateway _gateway;

    public PaymentProcessor(IPaymentGateway gateway)
    {
        _gateway = gateway;
    }

    public void Pay(decimal amount, string currency)
    {
        _gateway.ProcessPayment(amount, currency);
    }
}

// ===== Usage =====
// var processor = new PaymentProcessor(new PayPalAdapter(new PayPalService()));
// processor.Pay(99.99m, "USD");
```

---

### Java 21 Example

```java
// ===== Target Interface =====
interface PaymentGateway {
    void processPayment(double amount, String currency);
}

// ===== Adaptees =====
class StripeAPI {
    public void makePayment(double amount, String currency) {
        System.out.println("Stripe: charged " + amount + " " + currency);
    }
}

class PayPalService {
    public void sendPayment(double total, String curr) {
        System.out.println("PayPal: sent payment of " + total + " " + curr);
    }
}

class LegacyGateway {
    public void executeTransaction(String data) {
        System.out.println("Legacy: executing transaction -> " + data);
    }
}

// ===== Adapters =====
class StripeAdapter implements PaymentGateway {
    private final StripeAPI stripe = new StripeAPI();
    public void processPayment(double amount, String currency) {
        stripe.makePayment(amount, currency);
    }
}

class PayPalAdapter implements PaymentGateway {
    private final PayPalService paypal = new PayPalService();
    public void processPayment(double amount, String currency) {
        paypal.sendPayment(amount, currency);
    }
}

class LegacyAdapter implements PaymentGateway {
    private final LegacyGateway legacy = new LegacyGateway();
    public void processPayment(double amount, String currency) {
        String data = amount + ":" + currency;
        legacy.executeTransaction(data);
    }
}

// ===== Client =====
class PaymentProcessor {
    private final PaymentGateway gateway;
    public PaymentProcessor(PaymentGateway gateway) { this.gateway = gateway; }
    public void pay(double amount, String currency) { gateway.processPayment(amount, currency); }
}

// ===== Usage =====
// PaymentProcessor processor = new PaymentProcessor(new LegacyAdapter());
// processor.pay(250.00, "EUR");
```

---

### Python 13 Example

```python
# ===== Target Interface =====
class PaymentGateway:
    def process_payment(self, amount: float, currency: str) -> None:
        raise NotImplementedError

# ===== Adaptees =====
class StripeAPI:
    def make_payment(self, amount: float, currency: str) -> None:
        print(f"Stripe: charged {amount} {currency}")

class PayPalService:
    def send_payment(self, total: float, curr: str) -> None:
        print(f"PayPal: sent payment of {total} {curr}")

class LegacyGateway:
    def execute_transaction(self, data: str) -> None:
        print(f"Legacy: executing transaction -> {data}")

# ===== Adapters =====
class StripeAdapter(PaymentGateway):
    def __init__(self, stripe: StripeAPI) -> None:
        self._stripe = stripe

    def process_payment(self, amount: float, currency: str) -> None:
        self._stripe.make_payment(amount, currency)

class PayPalAdapter(PaymentGateway):
    def __init__(self, paypal: PayPalService) -> None:
        self._paypal = paypal

    def process_payment(self, amount: float, currency: str) -> None:
        self._paypal.send_payment(amount, currency)

class LegacyAdapter(PaymentGateway):
    def __init__(self, legacy: LegacyGateway) -> None:
        self._legacy = legacy

    def process_payment(self, amount: float, currency: str) -> None:
        data = f"{amount}:{currency}"
        self._legacy.execute_transaction(data)

# ===== Client =====
class PaymentProcessor:
    def __init__(self, gateway: PaymentGateway) -> None:
        self.gateway = gateway

    def pay(self, amount: float, currency: str) -> None:
        self.gateway.process_payment(amount, currency)

# ===== Usage =====
# processor = PaymentProcessor(PayPalAdapter(PayPalService()))
# processor.pay(75.5, "USD")
```

---

## Instructions for AIs (Knowledge Source Usage)

**When to apply the Adapter pattern:**

* When integrating external APIs with incompatible interfaces.
* When migrating from legacy systems without rewriting client code.
* When enforcing consistency across multiple third-party implementations.
* When working with APIs you can’t modify (vendor SDKs, libraries, legacy code).

**When not to use it:**

* When you control both interfaces — use direct refactoring instead.
* When adapting behavior (not just interface) would require rewriting logic.
* When the extra indirection layer introduces unnecessary complexity.

**Relation to other patterns:**

| Pattern       | Relationship                                                                 |
| ------------- | ---------------------------------------------------------------------------- |
| **Bridge**    | Adapter changes interface; Bridge separates abstraction from implementation. |
| **Decorator** | Adapter changes interface; Decorator adds responsibilities.                  |
| **Facade**    | Adapter targets one class; Facade unifies a whole subsystem.                 |
| **Proxy**     | Adapter changes interface; Proxy controls access.                            |

**Cares & considerations:**

* Keep adapter logic minimal — only translate, don’t rewrite.
* Document any semantic mismatches (e.g., units, time zones).
* Consider using composition (preferred) instead of inheritance.
* Test adapters with real and mock APIs to ensure fidelity.
* For many adapters, use a **registry or factory** to select the right one dynamically.

**Common pitfalls:**

* Creating “God Adapters” that handle too many APIs in one class.
* Failing to update adapters when external APIs change.
* Hiding errors silently during interface translation.
* Over-adapting when a simpler integration suffices.

---

## References and Bibliography

* Gamma, Erich et al. *Design Patterns: Elements of Reusable Object-Oriented Software*. Addison-Wesley, 1994.
* Refactoring.Guru — [Adapter Pattern](https://refactoring.guru/design-patterns/adapter)
* Microsoft Docs — *Adapter Design Pattern in C#*.
* Martin Fowler — *Patterns of Enterprise Application Architecture*.
* Oracle Java Tutorials — “Working with Design Patterns.”
* Real Python — *Adapter Pattern in Python: Integrating External APIs.*

---

## Quick Summary / Mnemonic

**“Make it fit.”**  
Use an Adapter to bridge between incompatible interfaces.
You keep your client code stable while flexibly connecting to external or legacy systems.

---
