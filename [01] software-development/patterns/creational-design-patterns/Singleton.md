# Singleton Pattern

## Short Description

The **Singleton** pattern ensures a class has only one instance and provides a global point of access to it. This pattern controls object creation, restricting instantiation to a single object while making that object easily accessible throughout the application.

---

## Comprehensive Overview

The **Singleton** pattern is one of the creational design patterns defined by the *Gang of Four (GoF)*. It addresses the need for exactly one instance of a class that must be accessible from anywhere in the system while controlling when and how that instance is created.

The pattern solves several key problems:
- It prevents multiple instantiations of classes that represent shared resources
- It provides lazy initialization (creating the instance only when needed)
- It ensures thread-safe access in multi-threaded environments
- It offers a controlled way to access global state

Common examples include configuration managers, logging services, connection pools, and caches where having multiple instances would cause inconsistencies or waste resources.

---

## Problems this topic solves

1. **Global State Management** - Ensures all parts of the system share the same instance for resources like configuration settings, logging systems, or caches.
2. **Controlled Access** - Prevents uncontrolled instantiation of shared resources that should exist only once.
3. **Resource Optimization** - Avoids creating multiple instances of heavy objects, saving memory and initialization time.
4. **Consistent State** - Maintains consistent state across the application for shared resources.

---

## Limitations — what it does *not* solve

* **Testability Issues** - Singletons are difficult to mock in unit tests due to tight coupling.
* **Hidden Dependencies** - Dependencies on singletons are not explicit in method signatures.
* **Violates Single Responsibility Principle** - The class manages both its business logic and its instance control.
* **Global State Problems** - Can lead to hidden dependencies and make code harder to reason about.
* **Lifecycle Management** - Difficult to manage destruction or recreation of the singleton instance.

---

## When to use

* You need **exactly one instance** of a class that must be accessible throughout the application.
* The instance requires **controlled initialization** (lazy loading, thread safety).
* The object represents a **shared resource** that would be expensive to create multiple times.
* Examples: Configuration managers, logging services, connection pools, application caches.

---

## When to avoid

* You need **multiple independent instances** of the class.
* The object holds **mutable, user-specific state** (e.g., user sessions).
* **Testability** is a primary concern (singletons resist mocking).
* You're using **dependency injection** frameworks (which provide better alternatives).
* The class doesn't represent a truly singular resource.

---

## Implementation Examples

The following examples show thread-safe Singleton implementations in different programming languages, demonstrating proper lazy initialization and thread safety.

---

### C# Example (Thread-Safe with Lazy<T>)

```csharp
using System;

public sealed class ConfigurationManager
{
    // Thread-safe lazy initialization using Lazy<T>
    private static readonly Lazy<ConfigurationManager> _instance =
        new Lazy<ConfigurationManager>(() => new ConfigurationManager());

    // Public property to access the singleton instance
    public static ConfigurationManager Instance => _instance.Value;

    // Private constructor prevents external instantiation
    private ConfigurationManager()
    {
        // Initialize configuration settings
        Console.WriteLine("ConfigurationManager initialized");
    }

    // Business method example
    public string GetSetting(string key)
    {
        // In real implementation, this would read from config file/database
        return $"value-for-{key}";
    }

    public void UpdateSetting(string key, string value)
    {
        // Update configuration setting
        Console.WriteLine($"Updated {key} to {value}");
    }
}

// ===== Usage =====
// var config = ConfigurationManager.Instance;
// string connectionString = config.GetSetting("DatabaseConnection");
// config.UpdateSetting("Timeout", "30");
```

---

### Java 21 Example (Thread-Safe with Static Holder)

```java
public class ConfigurationManager {
    private java.util.Map<String, String> settings;
    
    // Private constructor prevents external instantiation
    private ConfigurationManager() {
        this.settings = new java.util.HashMap<>();
        // Load initial configuration
        settings.put("database.host", "localhost");
        settings.put("database.port", "5432");
        System.out.println("ConfigurationManager initialized");
    }
    
    // Static holder class ensures lazy initialization and thread safety
    private static class Holder {
        private static final ConfigurationManager INSTANCE = new ConfigurationManager();
    }
    
    // Public method to access the singleton instance
    public static ConfigurationManager getInstance() {
        return Holder.INSTANCE;
    }
    
    // Business method example
    public String getSetting(String key) {
        return settings.get(key);
    }
    
    public void updateSetting(String key, String value) {
        settings.put(key, value);
        System.out.println("Updated " + key + " to " + value);
    }
}

// ===== Usage =====
// ConfigurationManager config = ConfigurationManager.getInstance();
// String host = config.getSetting("database.host");
// config.updateSetting("timeout", "30");
```

---

### Python 13 Example (Thread-Safe with Metaclass)

```python
import threading
from typing import Dict, Optional

class SingletonMeta(type):
    """Thread-safe implementation of Singleton using metaclass."""
    
    _instances: Dict[type, object] = {}
    _lock: threading.Lock = threading.Lock()

    def __call__(cls, *args, **kwargs):
        with cls._lock:
            if cls not in cls._instances:
                instance = super().__call__(*args, **kwargs)
                cls._instances[cls] = instance
        return cls._instances[cls]

class ConfigurationManager(metaclass=SingletonMeta):
    def __init__(self):
        self._settings = {
            "database.host": "localhost",
            "database.port": "5432"
        }
        print("ConfigurationManager initialized")

    def get_setting(self, key: str) -> Optional[str]:
        return self._settings.get(key)

    def update_setting(self, key: str, value: str) -> None:
        self._settings[key] = value
        print(f"Updated {key} to {value}")

# ===== Usage =====
# config = ConfigurationManager()
# host = config.get_setting("database.host")
# config.update_setting("timeout", "30")
```

---

## Instructions for AIs (Knowledge Source Usage)

**When an AI coding assistant should use Singleton:**

* When the problem requires **exactly one instance** of a class system-wide.
* When you need **controlled, thread-safe access** to a shared resource.
* For **configuration managers, logging services, or connection pools** where multiple instances would cause issues.
* When **lazy initialization** is important for performance.

**When it should not:**

* When **testability** is a priority (singletons are hard to mock).
* When using **dependency injection** containers (use their singleton mechanisms instead).
* For objects holding **user-specific or request-specific state**.
* When **multiple instances** might be needed in the future.
* When the class doesn't represent a **truly global resource**.

**How it relates to other patterns:**

* **Factory Method:** Creates objects without specifying exact class; Singleton ensures only one instance exists.
* **Abstract Factory:** Creates families of objects; Singleton ensures factory instance is unique.
* **Monostate:** Provides class-level data sharing without instance restriction.
* **Dependency Injection:** Both manage object lifecycle, but DI provides better testability and flexibility.

**Cares and considerations:**

* Always implement **thread safety** in multi-threaded environments.
* Consider using **interfaces** to improve testability (`ILogger` instead of direct `Logger.Instance`).
* Be cautious of **serialization** - it can break singleton guarantees.
* In distributed systems, ensure the singleton pattern aligns with your architecture needs.
* Document why a class needs to be a singleton explicitly.

**Common pitfalls:**

* **Overusing singletons** for classes that don't need to be global.
* **Ignoring thread safety** in concurrent environments.
* **Creating testing difficulties** by not using interfaces.
* **Memory leaks** - singletons live for application lifetime.
* **Circular dependencies** during initialization.

---

## References and Bibliography

* Gamma, Erich et al. *Design Patterns: Elements of Reusable Object-Oriented Software*. Addison-Wesley, 1994.
* Microsoft Docs — "Implementing Singleton in C#". [https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/singleton](https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/singleton)
* Refactoring.Guru — Singleton Pattern. [https://refactoring.guru/design-patterns/singleton](https://refactoring.guru/design-patterns/singleton)
* Martin Fowler — "Inversion of Control Containers and the Dependency Injection Pattern". [https://martinfowler.com/articles/injection.html](https://martinfowler.com/articles/injection.html)
* JavaWorld — "Effective Singleton Patterns in Java". [https://www.javaworld.com/article/2073352/core-java-singleton-design-pattern-best-practices.html](https://www.javaworld.com/article/2073352/core-java-singleton-design-pattern-best-practices.html)

---

## Quick Summary / Mnemonic

**"One to rule them all."**
Use Singleton when you need exactly one instance of a class that must be globally accessible, but beware of the testing and coupling trade-offs.

---

Would you like me to generate this into a downloadable `.md` file?