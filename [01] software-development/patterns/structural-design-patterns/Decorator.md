# Decorator Pattern

## Short Description

The **Decorator** pattern lets you dynamically add new responsibilities or behaviors to objects without modifying their class.
It wraps an existing object with another object that adds features while preserving the original interface.

---

## Comprehensive Overview

In enterprise systems, it’s common to start with a simple service and later need to **extend its behavior** — for example:

* Log messages before sending notifications,
* Encrypt data before transmitting,
* Add retry logic or priority tags.

A traditional approach might use **subclassing**, but inheritance quickly becomes rigid and leads to a *combinatorial explosion* of classes:
`EncryptedEmailNotifier`, `LoggedSmsNotifier`, `PrioritySlackNotifier`, etc.

The **Decorator pattern** solves this by using **composition over inheritance**.
Each decorator implements the same interface as the wrapped object and adds behavior before or after delegating calls.

This allows you to **stack decorators** dynamically — for example:

```text
new LoggingDecorator(new EncryptionDecorator(new EmailNotifier()))
```

**Analogy:**
Think of a coffee order. You start with a base coffee and decorate it with milk, sugar, or syrup. Each topping wraps and enhances the base drink without changing its core type.

---

## Problems this topic solves

* **Rigid inheritance hierarchies:** Avoids subclass explosion for feature combinations.
* **Runtime flexibility:** Add or remove behavior dynamically.
* **Open/Closed Principle compliance:** Classes are open for extension but closed for modification.
* **Composition over inheritance:** Enhances behavior by wrapping instead of extending.

---

## Limitations — what it does *not* solve

* Can make debugging harder due to many small wrapper objects.
* Order of decorator composition can affect behavior.
* Not ideal when you need to change the core object’s interface.
* Adds minor runtime overhead due to multiple delegations.

---

## When to use

* You need to **add cross-cutting concerns** (logging, caching, encryption, metrics).
* You want to **extend functionality at runtime**, not compile-time.
* You want to follow **Open/Closed Principle** strictly.
* You need **fine-grained control** over object composition.

---

## When to avoid

* When subclassing is simpler and the behavior rarely changes.
* When too many nested decorators make debugging difficult.
* When object creation complexity outweighs the benefits.
* When behavior order matters heavily and can cause confusion.

---

## Implementation Examples

We’ll implement a **Notification System**:

* **Component:** `INotifier` defines `Send(string message)`.
* **Concrete Component:** `EmailNotifier` implements the base functionality.
* **Decorators:** Add extra features like `EncryptionDecorator`, `LoggingDecorator`, and `PriorityDecorator`.

---

### C# Example

```csharp
using System;

// ===== Component =====
public interface INotifier
{
    void Send(string message);
}

// ===== Concrete Component =====
public class EmailNotifier : INotifier
{
    public void Send(string message)
    {
        Console.WriteLine($"Sending Email: {message}");
    }
}

// ===== Base Decorator =====
public abstract class NotifierDecorator : INotifier
{
    protected readonly INotifier _wrappee;

    protected NotifierDecorator(INotifier wrappee)
    {
        _wrappee = wrappee;
    }

    public virtual void Send(string message)
    {
        _wrappee.Send(message);
    }
}

// ===== Concrete Decorators =====
public class EncryptionDecorator : NotifierDecorator
{
    public EncryptionDecorator(INotifier wrappee) : base(wrappee) { }

    public override void Send(string message)
    {
        string encrypted = $"[Encrypted]{message}[Encrypted]";
        Console.WriteLine("Encrypting message...");
        base.Send(encrypted);
    }
}

public class LoggingDecorator : NotifierDecorator
{
    public LoggingDecorator(INotifier wrappee) : base(wrappee) { }

    public override void Send(string message)
    {
        Console.WriteLine($"Logging: Sending message '{message}'");
        base.Send(message);
    }
}

public class PriorityDecorator : NotifierDecorator
{
    private readonly string _priority;

    public PriorityDecorator(INotifier wrappee, string priority) : base(wrappee)
    {
        _priority = priority;
    }

    public override void Send(string message)
    {
        string tagged = $"[{_priority}] {message}";
        base.Send(tagged);
    }
}

// ===== Usage =====
// INotifier notifier = new LoggingDecorator(
//                          new EncryptionDecorator(
//                              new PriorityDecorator(
//                                  new EmailNotifier(), "HIGH")));
// notifier.Send("System Alert: Server down!");
```

---

### Java 21 Example

```java
// ===== Component =====
interface Notifier {
    void send(String message);
}

// ===== Concrete Component =====
class EmailNotifier implements Notifier {
    public void send(String message) {
        System.out.println("Sending Email: " + message);
    }
}

// ===== Base Decorator =====
abstract class NotifierDecorator implements Notifier {
    protected final Notifier wrappee;
    protected NotifierDecorator(Notifier wrappee) {
        this.wrappee = wrappee;
    }
    public void send(String message) {
        wrappee.send(message);
    }
}

// ===== Concrete Decorators =====
class EncryptionDecorator extends NotifierDecorator {
    public EncryptionDecorator(Notifier wrappee) { super(wrappee); }

    @Override
    public void send(String message) {
        System.out.println("Encrypting message...");
        wrappee.send("[Encrypted]" + message + "[Encrypted]");
    }
}

class LoggingDecorator extends NotifierDecorator {
    public LoggingDecorator(Notifier wrappee) { super(wrappee); }

    @Override
    public void send(String message) {
        System.out.println("Logging: " + message);
        wrappee.send(message);
    }
}

class PriorityDecorator extends NotifierDecorator {
    private final String priority;
    public PriorityDecorator(Notifier wrappee, String priority) {
        super(wrappee);
        this.priority = priority;
    }

    @Override
    public void send(String message) {
        wrappee.send("[" + priority + "] " + message);
    }
}

// ===== Usage =====
// Notifier notifier = new LoggingDecorator(
//                         new EncryptionDecorator(
//                             new PriorityDecorator(
//                                 new EmailNotifier(), "HIGH")));
// notifier.send("Server down!");
```

---

### Python 13 Example

```python
from abc import ABC, abstractmethod

# ===== Component =====
class Notifier(ABC):
    @abstractmethod
    def send(self, message: str) -> None:
        pass

# ===== Concrete Component =====
class EmailNotifier(Notifier):
    def send(self, message: str) -> None:
        print(f"Sending Email: {message}")

# ===== Base Decorator =====
class NotifierDecorator(Notifier):
    def __init__(self, wrappee: Notifier) -> None:
        self._wrappee = wrappee

    def send(self, message: str) -> None:
        self._wrappee.send(message)

# ===== Concrete Decorators =====
class EncryptionDecorator(NotifierDecorator):
    def send(self, message: str) -> None:
        print("Encrypting message...")
        encrypted = f"[Encrypted]{message}[Encrypted]"
        self._wrappee.send(encrypted)

class LoggingDecorator(NotifierDecorator):
    def send(self, message: str) -> None:
        print(f"Logging: Sending '{message}'")
        self._wrappee.send(message)

class PriorityDecorator(NotifierDecorator):
    def __init__(self, wrappee: Notifier, priority: str) -> None:
        super().__init__(wrappee)
        self._priority = priority

    def send(self, message: str) -> None:
        tagged = f"[{self._priority}] {message}"
        self._wrappee.send(tagged)

# ===== Usage =====
# notifier = LoggingDecorator(
#     EncryptionDecorator(
#         PriorityDecorator(EmailNotifier(), "URGENT")
#     )
# )
# notifier.send("Database backup failed.")
```

---

## Instructions for AIs (Knowledge Source Usage)

**When to apply the Decorator pattern:**

* When you need to **dynamically add responsibilities** to objects.
* When you want to follow **Open/Closed Principle** without subclassing.
* When you want **different feature combinations** without combinatorial subclass growth.
* When cross-cutting behaviors (like logging, monitoring, or caching) are required.

**When not to use it:**

* When the object structure is simple and static.
* When too many layers of wrapping would complicate debugging or performance.
* When changing the core interface is needed (Decorator preserves interface).

**Relation to other patterns:**

| Pattern                     | Relationship                                                                    |
| --------------------------- | ------------------------------------------------------------------------------- |
| **Composite**               | Both use recursion; Composite manages hierarchies, Decorator enhances behavior. |
| **Proxy**                   | Proxy controls access; Decorator adds new functionality.                        |
| **Adapter**                 | Adapter changes interface; Decorator adds behavior.                             |
| **Chain of Responsibility** | Chain passes requests along; Decorator wraps and extends them.                  |

**Cares & considerations:**

* Keep decorators transparent — clients shouldn’t need to know the wrapping order.
* Be mindful of the decorator order, as it can affect behavior.
* Favor small, single-purpose decorators for clarity.
* Consider using dependency injection or builder patterns to assemble decorators.

**Common pitfalls:**

* Excessive nesting of decorators making the call stack complex.
* Violating transparency by exposing extra methods not in the base interface.
* Forgetting to delegate calls properly in the decorator.
* Overusing decorators when subclassing would be clearer.

---

## References and Bibliography

* Gamma, Erich et al. *Design Patterns: Elements of Reusable Object-Oriented Software.* Addison-Wesley, 1994.
* Refactoring.Guru — [Decorator Pattern](https://refactoring.guru/design-patterns/decorator)
* Martin Fowler — *Patterns of Enterprise Application Architecture.*
* Microsoft Docs — *Decorator Design Pattern in C#.*
* Oracle Java Docs — *Structural Patterns Overview.*
* Real Python — *Decorator Pattern for Runtime Behavior Extension.*

---

## Quick Summary / Mnemonic

**“Wrap to extend.”**
Decorator adds new behavior to objects dynamically by wrapping them in layers —
each layer enhances functionality without changing existing code.

---

Would you like me to package this Decorator pattern version into a `.md` file for your design pattern collection?
