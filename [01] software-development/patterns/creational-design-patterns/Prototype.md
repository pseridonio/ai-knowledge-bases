# Prototype Pattern

## Short Description

The **Prototype** pattern allows creating new objects by cloning existing ones instead of instantiating from scratch.
It is especially useful when object initialization is expensive or when many similar objects differ only slightly in configuration.

---

## Comprehensive Overview

In enterprise systems, objects like **user profiles**, **forms**, or **configuration templates** often have complex default states: multiple permissions, preloaded preferences, and linked resources.
Building these from scratch every time can be inefficient or repetitive.

The **Prototype** pattern provides a way to create new objects by cloning preconfigured “prototype” instances.
Each prototype acts as a blueprint: when the client requests a new object, the system copies the prototype and adjusts only the differences (e.g., username, department, theme).

This avoids subclass explosion (no need for “AdminProfile,” “SalesProfile,” “SupportProfile” classes) and supports dynamic configuration — new prototypes can even be registered at runtime.

**Analogy:** In HR software, instead of manually setting up a new employee from scratch, you duplicate an existing user profile (same permissions, department settings) and just update the name and ID.

---

## Problems this topic solves

* **Expensive initialization:** Avoid reconstructing large, complex objects (e.g., permissions, preferences) repeatedly.
* **Dynamic configuration:** Allow creation of variants at runtime based on preconfigured instances.
* **Avoiding subclass proliferation:** No need for dozens of slightly different classes.
* **Runtime extensibility:** Add or modify prototypes dynamically without changing source code.

---

## Limitations — what it does *not* solve

* Cloning doesn’t work well for objects with **non-copyable resources** (file handles, sockets, threads).
* Requires careful management of **deep vs. shallow copies** to avoid shared references.
* Does not eliminate the need for validation — clones may need post-copy adjustments.
* May increase memory usage if many clones coexist in memory.

---

## When to use

* Creating new instances from existing templates or configurations.
* Objects have **complex, costly** setup steps (e.g., loading permissions or settings).
* Variations are mostly differences in **state**, not behavior.
* You need runtime flexibility: users or systems can define new “prototypes.”

---

## When to avoid

* Object creation is simple and cheap.
* The system requires strict control or validation during instantiation.
* Objects manage external resources or depend on context that cannot be cloned.

---

## Implementation Examples

We’ll implement a **User Profile Prototype System**.
Each user profile holds permissions, preferences, and role data.
Instead of creating profiles from scratch, we clone an existing prototype and adjust the user-specific fields.

---

### C# Example

```csharp
using System;
using System.Collections.Generic;

// ===== Prototype Interface =====
public interface IPrototype<T>
{
    T Clone();
}

// ===== Concrete Prototype =====
public class UserProfile : IPrototype<UserProfile>
{
    public string Username { get; set; }
    public string Role { get; set; }
    public List<string> Permissions { get; set; }
    public Dictionary<string, string> Preferences { get; set; }

    public UserProfile(string username, string role)
    {
        Username = username;
        Role = role;
        Permissions = new List<string>();
        Preferences = new Dictionary<string, string>();
    }

    // Deep clone
    public UserProfile Clone()
    {
        return new UserProfile(this.Username, this.Role)
        {
            Permissions = new List<string>(this.Permissions),
            Preferences = new Dictionary<string, string>(this.Preferences)
        };
    }

    public void Display()
    {
        Console.WriteLine($"User: {Username}, Role: {Role}");
        Console.WriteLine($"Permissions: {string.Join(", ", Permissions)}");
        Console.WriteLine($"Theme: {Preferences.GetValueOrDefault("theme", "default")}");
        Console.WriteLine();
    }
}

// ===== Client =====
public static class ProfileRegistry
{
    private static readonly Dictionary<string, UserProfile> _prototypes = new();

    public static void Register(string key, UserProfile profile) => _prototypes[key] = profile;
    public static UserProfile CreateProfile(string key, string username)
    {
        if (!_prototypes.ContainsKey(key))
            throw new ArgumentException($"Unknown profile type: {key}");
        var clone = _prototypes[key].Clone();
        clone.Username = username;
        return clone;
    }
}

// ===== Usage =====
// var admin = new UserProfile("admin-template", "Administrator");
// admin.Permissions.AddRange(new[] { "READ", "WRITE", "DELETE" });
// admin.Preferences["theme"] = "dark";
// ProfileRegistry.Register("admin", admin);

// var john = ProfileRegistry.CreateProfile("admin", "john.doe");
// john.Display();
```

---

### Java 21 Example

```java
import java.util.*;

// ===== Prototype Interface =====
interface Prototype<T> {
    T clonePrototype();
}

// ===== Concrete Prototype =====
class UserProfile implements Prototype<UserProfile> {
    private String username;
    private String role;
    private List<String> permissions = new ArrayList<>();
    private Map<String, String> preferences = new HashMap<>();

    public UserProfile(String username, String role) {
        this.username = username;
        this.role = role;
    }

    public void addPermission(String p) { permissions.add(p); }
    public void setPreference(String key, String value) { preferences.put(key, value); }

    @Override
    public UserProfile clonePrototype() {
        UserProfile copy = new UserProfile(this.username, this.role);
        copy.permissions = new ArrayList<>(this.permissions);
        copy.preferences = new HashMap<>(this.preferences);
        return copy;
    }

    public void setUsername(String username) { this.username = username; }

    public void display() {
        System.out.println("User: " + username + ", Role: " + role);
        System.out.println("Permissions: " + permissions);
        System.out.println("Theme: " + preferences.getOrDefault("theme", "default"));
        System.out.println();
    }
}

// ===== Registry (Client) =====
class ProfileRegistry {
    private static final Map<String, UserProfile> prototypes = new HashMap<>();

    public static void register(String key, UserProfile prototype) {
        prototypes.put(key, prototype);
    }

    public static UserProfile createProfile(String key, String username) {
        UserProfile proto = prototypes.get(key);
        if (proto == null)
            throw new IllegalArgumentException("Unknown profile type: " + key);
        UserProfile clone = proto.clonePrototype();
        clone.setUsername(username);
        return clone;
    }
}

// ===== Usage =====
// UserProfile admin = new UserProfile("template-admin", "Administrator");
// admin.addPermission("READ");
// admin.addPermission("WRITE");
// admin.setPreference("theme", "dark");
// ProfileRegistry.register("admin", admin);
//
// UserProfile mary = ProfileRegistry.createProfile("admin", "mary.smith");
// mary.display();
```

---

### Python 13 Example

```python
import copy
from typing import Dict, List

# ===== Concrete Prototype =====
class UserProfile:
    def __init__(self, username: str, role: str) -> None:
        self.username = username
        self.role = role
        self.permissions: List[str] = []
        self.preferences: Dict[str, str] = {}

    def clone(self) -> "UserProfile":
        # Deep copy to avoid shared references
        new_copy = copy.deepcopy(self)
        return new_copy

    def display(self) -> None:
        theme = self.preferences.get("theme", "default")
        print(f"User: {self.username}, Role: {self.role}")
        print(f"Permissions: {', '.join(self.permissions)}")
        print(f"Theme: {theme}\n")


# ===== Registry (Client) =====
class ProfileRegistry:
    _prototypes: Dict[str, UserProfile] = {}

    @classmethod
    def register(cls, key: str, profile: UserProfile) -> None:
        cls._prototypes[key] = profile

    @classmethod
    def create_profile(cls, key: str, username: str) -> UserProfile:
        proto = cls._prototypes.get(key)
        if not proto:
            raise ValueError(f"Unknown profile type: {key}")
        clone = proto.clone()
        clone.username = username
        return clone


# ===== Usage =====
# admin = UserProfile("template-admin", "Administrator")
# admin.permissions.extend(["READ", "WRITE", "DELETE"])
# admin.preferences["theme"] = "dark"
# ProfileRegistry.register("admin", admin)
#
# user = ProfileRegistry.create_profile("admin", "john.doe")
# user.display()
```

---

## Instructions for AIs (Knowledge Source Usage)

**When to use the Prototype pattern:**

* When creating new instances by **cloning existing configured objects** is faster or simpler.
* When the system must support **dynamic types** defined at runtime.
* When you want to **avoid subclass proliferation** and hardcoded constructors.
* When building **template-based systems** (documents, forms, user profiles, workflows).

**When not to use it:**

* When objects are simple or have trivial constructors.
* When cloning is complex or error-prone (deep copy issues).
* When external resources (DB connections, sockets) are not cloneable.

**Relation to other patterns:**

| Pattern              | Relationship                                                                              |
| -------------------- | ----------------------------------------------------------------------------------------- |
| **Factory Method**   | Factory creates new instances via instantiation; Prototype clones existing instances.     |
| **Abstract Factory** | Factory creates families of objects; Prototype clones individual instances.               |
| **Builder**          | Builder constructs complex objects step-by-step; Prototype copies them in one operation.  |
| **Singleton**        | Opposite in intent — Singleton restricts to one instance; Prototype multiplies instances. |

**Cares & considerations:**

* Define whether clones should be **deep** (fully independent) or **shallow** (shared references).
* Manage prototype registration carefully — use keys or type maps for clarity.
* Ensure each prototype can be reset, cloned, or extended safely.
* Consider using built-in clone utilities (e.g., `MemberwiseClone`, `copy.deepcopy`) where available.

**Common pitfalls:**

* Accidentally sharing mutable collections between clones.
* Forgetting to reassign unique identifiers (e.g., username, ID).
* Cloning objects with circular references or external dependencies.
* Overusing prototypes when factories or dependency injection would suffice.

---

## References and Bibliography

* Gamma, Erich et al. *Design Patterns: Elements of Reusable Object-Oriented Software*. Addison-Wesley, 1994.
* Microsoft Docs — “Prototype Design Pattern in C#.”
* Martin Fowler — *Patterns of Enterprise Application Architecture.*
* Refactoring.Guru — [Prototype Pattern](https://refactoring.guru/design-patterns/prototype).
* Oracle Java Docs — *Object Cloning and Copy Constructors.*
* Real Python — *Design Patterns in Python.*

---

## Quick Summary / Mnemonic

**“Don’t reinvent — replicate.”**
Clone a preconfigured prototype instead of creating from scratch.
Adjust what’s different, and you’ve got a new instance ready to go.

---
