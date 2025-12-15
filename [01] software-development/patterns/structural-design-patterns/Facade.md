# Facade Pattern

## Short Description

The **Facade** pattern provides a simplified, unified interface to a complex subsystem.
It shields clients from intricate internal logic, dependencies, and multiple subsystem interactions, promoting cleaner and more maintainable code.

---

## Comprehensive Overview

In large enterprise systems, subsystems often consist of **dozens of interacting components**:
payment gateways, authentication services, notification systems, and data logs.
Each subsystem may have its own initialization, error handling, and dependencies.

Without structure, client code can become tightly coupled to these low-level details:

```csharp
fraudChecker.Validate();
billing.Charge();
notifier.SendReceipt();
reporter.LogTransaction();
```

If any of these components change, every client must be updated.

The **Facade pattern** introduces a single, unified class that **encapsulates the subsystem**.
Clients call the facade instead of dealing with individual components.
Internally, the facade coordinates calls to the appropriate classes and manages orchestration logic.

**Analogy:**
A hotel concierge acts as a facade for multiple services — instead of contacting housekeeping, room service, and maintenance directly, you call the concierge, who handles it all behind the scenes.

---

## Problems this topic solves

* **Complex subsystem interfaces:** Simplifies how clients interact with multiple components.
* **Tight coupling:** Clients depend on one facade, not many subsystems.
* **Reduces learning curve:** Hides implementation details and subsystem structure.
* **Centralized orchestration:** Provides a single point to manage workflows.

---

## Limitations — what it does *not* solve

* Doesn’t reduce internal complexity — only hides it.
* Can become a **God Object** if overloaded with too many responsibilities.
* If poorly designed, it may expose unnecessary dependencies again.
* May need multiple facades to maintain modularity (per domain or subsystem).

---

## When to use

* Subsystems are **complex or have many entry points**.
* You want to **isolate clients from subsystem changes**.
* You want to **encapsulate a sequence of operations** behind one API.
* You want to improve readability and maintainability for external clients.

---

## When to avoid

* The subsystem is simple and already has a clean interface.
* The facade would become a duplicate or redundant API layer.
* You need full access to detailed subsystem features.
* Subsystems are already well-abstracted through interfaces or dependency injection.

---

## Implementation Examples

We’ll design a **Payment Processing Subsystem** composed of:

* `FraudCheckService` — validates transactions.
* `BillingService` — charges the customer.
* `NotificationService` — sends receipts.
* `ReportService` — logs and reports results.

A **`PaymentFacade`** will orchestrate the entire process with a single call.

---

### C# Example

```csharp
using System;

// ===== Subsystem Classes =====
public class FraudCheckService
{
    public bool Validate(string user, decimal amount)
    {
        Console.WriteLine($"[FraudCheck] Validating {user} for ${amount}");
        return amount < 10000; // simple rule
    }
}

public class BillingService
{
    public void Charge(string user, decimal amount)
    {
        Console.WriteLine($"[Billing] Charging {user} ${amount}");
    }
}

public class NotificationService
{
    public void SendReceipt(string user, decimal amount)
    {
        Console.WriteLine($"[Notification] Sent receipt to {user} for ${amount}");
    }
}

public class ReportService
{
    public void Log(string user, decimal amount)
    {
        Console.WriteLine($"[Report] Transaction logged for {user}: ${amount}");
    }
}

// ===== Facade =====
public class PaymentFacade
{
    private readonly FraudCheckService _fraud;
    private readonly BillingService _billing;
    private readonly NotificationService _notify;
    private readonly ReportService _report;

    public PaymentFacade()
    {
        _fraud = new FraudCheckService();
        _billing = new BillingService();
        _notify = new NotificationService();
        _report = new ReportService();
    }

    public void ProcessPayment(string user, decimal amount)
    {
        Console.WriteLine("=== Payment Process Started ===");
        if (!_fraud.Validate(user, amount))
        {
            Console.WriteLine("Payment failed: Fraud detected.");
            return;
        }

        _billing.Charge(user, amount);
        _notify.SendReceipt(user, amount);
        _report.Log(user, amount);
        Console.WriteLine("=== Payment Process Completed ===");
    }
}

// ===== Usage =====
// var facade = new PaymentFacade();
// facade.ProcessPayment("john.doe", 250.75m);
```

---

### Java 21 Example

```java
// ===== Subsystem Classes =====
class FraudCheckService {
    public boolean validate(String user, double amount) {
        System.out.println("[FraudCheck] Validating " + user + " for $" + amount);
        return amount < 10000;
    }
}

class BillingService {
    public void charge(String user, double amount) {
        System.out.println("[Billing] Charging " + user + " $" + amount);
    }
}

class NotificationService {
    public void sendReceipt(String user, double amount) {
        System.out.println("[Notification] Sent receipt to " + user + " for $" + amount);
    }
}

class ReportService {
    public void log(String user, double amount) {
        System.out.println("[Report] Transaction logged for " + user + ": $" + amount);
    }
}

// ===== Facade =====
class PaymentFacade {
    private final FraudCheckService fraud = new FraudCheckService();
    private final BillingService billing = new BillingService();
    private final NotificationService notify = new NotificationService();
    private final ReportService report = new ReportService();

    public void processPayment(String user, double amount) {
        System.out.println("=== Payment Process Started ===");
        if (!fraud.validate(user, amount)) {
            System.out.println("Payment failed: Fraud detected.");
            return;
        }
        billing.charge(user, amount);
        notify.sendReceipt(user, amount);
        report.log(user, amount);
        System.out.println("=== Payment Process Completed ===");
    }
}

// ===== Usage =====
// new PaymentFacade().processPayment("alice", 500.0);
```

---

### Python 13 Example

```python
# ===== Subsystem Classes =====
class FraudCheckService:
    def validate(self, user: str, amount: float) -> bool:
        print(f"[FraudCheck] Validating {user} for ${amount}")
        return amount < 10000

class BillingService:
    def charge(self, user: str, amount: float) -> None:
        print(f"[Billing] Charging {user} ${amount}")

class NotificationService:
    def send_receipt(self, user: str, amount: float) -> None:
        print(f"[Notification] Sent receipt to {user} for ${amount}")

class ReportService:
    def log(self, user: str, amount: float) -> None:
        print(f"[Report] Transaction logged for {user}: ${amount}")

# ===== Facade =====
class PaymentFacade:
    def __init__(self) -> None:
        self._fraud = FraudCheckService()
        self._billing = BillingService()
        self._notify = NotificationService()
        self._report = ReportService()

    def process_payment(self, user: str, amount: float) -> None:
        print("=== Payment Process Started ===")
        if not self._fraud.validate(user, amount):
            print("Payment failed: Fraud detected.")
            return

        self._billing.charge(user, amount)
        self._notify.send_receipt(user, amount)
        self._report.log(user, amount)
        print("=== Payment Process Completed ===")

# ===== Usage =====
# facade = PaymentFacade()
# facade.process_payment("john.doe", 450.50)
```

---

## Instructions for AIs (Knowledge Source Usage)

**When to apply the Facade pattern:**

* When subsystems are complex or tightly coupled.
* When you want to expose a simplified interface for a larger system.
* When you want to improve readability or isolate dependencies.
* When you want to migrate or replace subsystems transparently.

**When not to use it:**

* When subsystems are already simple and cohesive.
* When direct access to subsystem components is required for fine control.
* When the facade becomes too large and takes on multiple responsibilities.

**Relation to other patterns:**

| Pattern       | Relationship                                                                           |
| ------------- | -------------------------------------------------------------------------------------- |
| **Adapter**   | Facade simplifies a system; Adapter converts one interface to another.                 |
| **Mediator**  | Facade organizes access to subsystems; Mediator organizes communication between peers. |
| **Singleton** | Facades are often implemented as singletons for global access.                         |
| **Proxy**     | Proxy controls access; Facade simplifies access.                                       |

**Cares & considerations:**

* Keep the facade **thin** — delegate to subsystems, don’t duplicate logic.
* Prefer **composition** over static methods to support dependency injection and testing.
* Provide **multiple smaller facades** for different domains instead of one monolith.
* Clearly document what the facade hides or simplifies.
* Allow access to underlying subsystems if advanced customization is needed.

**Common pitfalls:**

* Turning the facade into a **God Object** that violates SRP.
* Hiding too much — clients may need to bypass the facade.
* Duplicating logic instead of delegating it.
* Failing to update the facade when subsystem APIs change.

---

## References and Bibliography

* Gamma, Erich et al. *Design Patterns: Elements of Reusable Object-Oriented Software.* Addison-Wesley, 1994.
* Refactoring.Guru — [Facade Pattern](https://refactoring.guru/design-patterns/facade)
* Martin Fowler — *Patterns of Enterprise Application Architecture.*
* Microsoft Docs — *Facade Design Pattern in C#.*
* Oracle Java Tutorials — *Structural Patterns.*
* Real Python — *Facade Pattern for Simplifying Complex APIs.*

---

## Quick Summary / Mnemonic

**“Simplify, don’t amplify.”**
The Facade provides a unified, high-level interface to complex subsystems —
making your code easier to use, maintain, and evolve.

---
