# Composite Pattern

## Short Description

The **Composite** pattern lets you treat individual objects and groups of objects uniformly.
It organizes objects into tree-like structures where clients can interact with both single elements and entire compositions using the same interface.

---

## Comprehensive Overview

In enterprise systems, you often deal with **hierarchical data**:
organizations, product categories, file systems, UI elements, etc.
Each node in the hierarchy can be either:

* a **Leaf** (e.g., individual employee, file, or product), or
* a **Composite** (e.g., department, folder, or category).

Without a unified interface, client code must constantly check types:

```csharp
if (obj is Employee) ... else if (obj is Department) ...
```

This violates the **Open/Closed Principle** and leads to complex, brittle code.

The **Composite Pattern** solves this by defining a common interface (`IOrganizationUnit`) for both simple and composite objects.
Each composite can **contain and delegate** to its children recursively.

**Analogy:**
Think of a company’s org chart.
Each node (CEO, Department, Employee) can perform an operation like `GetSalary()` or `ShowDetails()`.
You don’t care if it’s a single person or a whole department — you just call the same method.

---

## Problems this topic solves

* **Uniform access:** Treat individual and grouped objects through the same interface.
* **Simplified client code:** No more type checking or special-casing.
* **Recursive structures:** Easily represent trees and hierarchies.
* **Scalable operations:** Add new node types without changing client logic.

---

## Limitations — what it does *not* solve

* Can make it **hard to restrict certain operations** (e.g., adding children to leaves).
* Tree traversal and recursion can add **performance overhead** in deep hierarchies.
* May introduce **complex debugging** in large recursive structures.
* Violates **Liskov Substitution** if certain composites or leaves fail to fully implement the interface.

---

## When to use

* You have **tree-like data structures** (org charts, file systems, product catalogs).
* You want to treat leaf and composite elements uniformly.
* Operations apply recursively to groups and individual elements.
* You want flexibility to grow or reorganize the hierarchy dynamically.

---

## When to avoid

* The hierarchy is flat and won’t evolve into nested structures.
* Leaf and composite behaviors differ too much for a common interface.
* You need strict type safety or compile-time validation of node types.
* You must enforce distinct logic for leaves vs. composites (breaks uniformity).

---

## Implementation Examples

We’ll model a **Corporate Organization Structure**:

* **Component (Interface):** `IOrganizationUnit` defines operations like `GetDetails()` and `GetTotalSalary()`.
* **Leaf:** `Employee` represents a single worker.
* **Composite:** `Department` represents a group that can contain employees or other departments.

---

### C# Example

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

// ===== Component =====
public interface IOrganizationUnit
{
    string Name { get; }
    decimal GetTotalSalary();
    void ShowDetails(int indent = 0);
}

// ===== Leaf =====
public class Employee : IOrganizationUnit
{
    public string Name { get; }
    public decimal Salary { get; }

    public Employee(string name, decimal salary)
    {
        Name = name;
        Salary = salary;
    }

    public decimal GetTotalSalary() => Salary;

    public void ShowDetails(int indent = 0)
    {
        Console.WriteLine($"{new string(' ', indent)}- Employee: {Name} (${Salary})");
    }
}

// ===== Composite =====
public class Department : IOrganizationUnit
{
    public string Name { get; }
    private readonly List<IOrganizationUnit> _units = new();

    public Department(string name) => Name = name;

    public void Add(IOrganizationUnit unit) => _units.Add(unit);
    public void Remove(IOrganizationUnit unit) => _units.Remove(unit);

    public decimal GetTotalSalary() => _units.Sum(u => u.GetTotalSalary());

    public void ShowDetails(int indent = 0)
    {
        Console.WriteLine($"{new string(' ', indent)}+ Department: {Name}");
        foreach (var unit in _units)
            unit.ShowDetails(indent + 2);
    }
}

// ===== Usage =====
// var devDept = new Department("Development");
// devDept.Add(new Employee("Alice", 7000));
// devDept.Add(new Employee("Bob", 6500));
//
// var hrDept = new Department("HR");
// hrDept.Add(new Employee("Clara", 5000));
//
// var headOffice = new Department("Head Office");
// headOffice.Add(devDept);
// headOffice.Add(hrDept);
//
// headOffice.ShowDetails();
// Console.WriteLine($"Total Salary: ${headOffice.GetTotalSalary()}");
```

---

### Java 21 Example

```java
import java.util.*;

// ===== Component =====
interface OrganizationUnit {
    String getName();
    double getTotalSalary();
    void showDetails(int indent);
}

// ===== Leaf =====
class Employee implements OrganizationUnit {
    private final String name;
    private final double salary;

    public Employee(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }

    public String getName() { return name; }
    public double getTotalSalary() { return salary; }

    public void showDetails(int indent) {
        System.out.println(" ".repeat(indent) + "- Employee: " + name + " ($" + salary + ")");
    }
}

// ===== Composite =====
class Department implements OrganizationUnit {
    private final String name;
    private final List<OrganizationUnit> units = new ArrayList<>();

    public Department(String name) { this.name = name; }

    public void add(OrganizationUnit unit) { units.add(unit); }
    public void remove(OrganizationUnit unit) { units.remove(unit); }

    public String getName() { return name; }

    public double getTotalSalary() {
        return units.stream().mapToDouble(OrganizationUnit::getTotalSalary).sum();
    }

    public void showDetails(int indent) {
        System.out.println(" ".repeat(indent) + "+ Department: " + name);
        for (OrganizationUnit u : units)
            u.showDetails(indent + 2);
    }
}

// ===== Usage =====
// Department itDept = new Department("IT");
// itDept.add(new Employee("John", 8000));
// itDept.add(new Employee("Sara", 7800));
//
// Department financeDept = new Department("Finance");
// financeDept.add(new Employee("Alex", 9000));
//
// Department company = new Department("Global Corp");
// company.add(itDept);
// company.add(financeDept);
//
// company.showDetails(0);
// System.out.println("Total Salary: $" + company.getTotalSalary());
```

---

### Python 13 Example

```python
from abc import ABC, abstractmethod
from typing import List

# ===== Component =====
class OrganizationUnit(ABC):
    @property
    @abstractmethod
    def name(self) -> str: ...

    @abstractmethod
    def get_total_salary(self) -> float: ...

    @abstractmethod
    def show_details(self, indent: int = 0) -> None: ...

# ===== Leaf =====
class Employee(OrganizationUnit):
    def __init__(self, name: str, salary: float) -> None:
        self._name = name
        self.salary = salary

    @property
    def name(self) -> str:
        return self._name

    def get_total_salary(self) -> float:
        return self.salary

    def show_details(self, indent: int = 0) -> None:
        print(" " * indent + f"- Employee: {self.name} (${self.salary})")

# ===== Composite =====
class Department(OrganizationUnit):
    def __init__(self, name: str) -> None:
        self._name = name
        self.units: List[OrganizationUnit] = []

    @property
    def name(self) -> str:
        return self._name

    def add(self, unit: OrganizationUnit) -> None:
        self.units.append(unit)

    def remove(self, unit: OrganizationUnit) -> None:
        self.units.remove(unit)

    def get_total_salary(self) -> float:
        return sum(unit.get_total_salary() for unit in self.units)

    def show_details(self, indent: int = 0) -> None:
        print(" " * indent + f"+ Department: {self.name}")
        for unit in self.units:
            unit.show_details(indent + 2)

# ===== Usage =====
# dev = Department("Development")
# dev.add(Employee("Alice", 7200))
# dev.add(Employee("Bob", 6800))
#
# hr = Department("Human Resources")
# hr.add(Employee("Eve", 5000))
#
# company = Department("Main Office")
# company.add(dev)
# company.add(hr)
#
# company.show_details()
# print(f"Total Salary: ${company.get_total_salary()}")
```

---

## Instructions for AIs (Knowledge Source Usage)

**When to apply the Composite pattern:**

* When objects form **part-whole hierarchies**.
* When clients should treat single objects and composites uniformly.
* When the structure should support recursive traversal or aggregation.
* When adding new component types shouldn’t require changing client logic.

**When not to use it:**

* When object relationships are flat or strictly non-recursive.
* When leaf and composite objects have incompatible responsibilities.
* When strict type control or domain constraints are more important than flexibility.

**Relation to other patterns:**

| Pattern       | Relationship                                                                  |
| ------------- | ----------------------------------------------------------------------------- |
| **Decorator** | Both use composition; Decorator adds behavior, Composite manages hierarchies. |
| **Iterator**  | Often used with Composite to traverse hierarchies.                            |
| **Visitor**   | Often complements Composite for operations over tree structures.              |
| **Flyweight** | Can reduce memory footprint for many small leaf objects.                      |

**Cares & considerations:**

* Avoid operations that make sense only for composites (e.g., “add child”) in the base interface unless necessary.
* Manage recursive calls carefully — protect against infinite loops.
* Document expected ownership and lifetime of child objects.
* Consider using composite iterators for deep trees.
* Use immutable composites for thread safety when needed.

**Common pitfalls:**

* Overgeneralizing the component interface (forcing leaves to implement meaningless methods).
* Failing to handle deep recursion (stack overflow risk).
* Mixing responsibilities between composite and leaf objects.
* Not caching aggregate results in performance-sensitive hierarchies.

---

## References and Bibliography

* Gamma, Erich et al. *Design Patterns: Elements of Reusable Object-Oriented Software*. Addison-Wesley, 1994.
* Refactoring.Guru — [Composite Pattern](https://refactoring.guru/design-patterns/composite)
* Martin Fowler — *Patterns of Enterprise Application Architecture.*
* Microsoft Docs — *Composite Design Pattern in C#.*
* Oracle Java Docs — *Composite Pattern Examples.*
* Real Python — *Composite Pattern for Hierarchical Objects.*

---

## Quick Summary / Mnemonic

**“One or many — treat them the same.”**
Use Composite when individual and composite objects must share behavior.
It’s perfect for hierarchies like organizations, file systems, and product structures.

---

Would you like me to generate this Composite version into a `.md` file for your design pattern collection (like we’ve done for the previous patterns)?
