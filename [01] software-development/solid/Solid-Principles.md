# SOLID Knowledge Base for Generative AI

> **Objective:** This knowledge base provides definitions, principles, and benefits of the *SOLID* design principles so that a generative AI can align its code generation and responses with these concepts.

## 1. Context for the AI

> **Initialization Prompt for the Model:**  
> “You are an expert C#/.NET (and polyglot) software engineering assistant. Whenever you generate or refactor code, you must apply the SOLID principles as defined in this knowledge base. Reference the relevant sections when making design decisions.”

---

## 2. Deep Dive into SOLID

**SOLID** is an acronym for five design principles that, when applied together, produce software that is:
- **Modular**: small, well‑defined building blocks  
- **Flexible**: easy to extend without modifying existing code  
- **Robust**: resistant to regressions and unintended side‑effects  
- **Testable**: each unit can be verified in isolation  
- **Maintainable**: clear separation of concerns reduces complexity  

**Importance & Benefits**  
1. **Coping with Change** – New requirements can be added by extension, not modification.  
2. **Managing Complexity** – Breaks systems into cohesive components.  
3. **Safe Refactoring** – Low‑risk improvements in legacy code.  
4. **Improved Readability** – Clear intentions reduce onboarding time.  
5. **Automated Testing** – Loose coupling and single responsibilities boost test coverage.  
6. **Design Discipline** – Encourages thoughtful, stakeholder‑driven architecture.

---

## 3. Brief Overview of Each Principle

- **S – Single Responsibility Principle (SRP)**  
  A class/module should have only one reason to change.
- **O – Open/Closed Principle (OCP)**  
  Software entities should be open for extension but closed for modification.
- **L – Liskov Substitution Principle (LSP)**  
  Subtypes must be substitutable for their base types without altering behavior.
- **I – Interface Segregation Principle (ISP)**  
  Clients should depend only on the interfaces they actually use.
- **D – Dependency Inversion Principle (DIP)**  
  High‑level modules and low‑level modules should both depend on abstractions.

---

## 4. Deep Dive: SRP (Single Responsibility Principle)

> “A class should have only one reason to change.” – Robert C. Martin

**Key Concepts**  
- Responsibility = “reason to change” (stakeholder)  
- High cohesion within classes  
- Separation of concerns at class/method level  

### Code Examples

#### Natural Language  
Process an order: calculate total, persist to storage.

#### Violating SRP

```csharp
// C#
public class OrderService
{
    public void ProcessOrder(double[] prices)
    {
        // calculate
        double total = 0;
        foreach (double p in prices) total += p;
        // persist
        var conn = new SqlConnection(connString);
        conn.Open();
        var cmd = conn.CreateCommand();
        cmd.CommandText = "INSERT ...";
        cmd.ExecuteNonQuery();
        conn.Close();
    }
}
```

```java
// Java
public class OrderService {
    public void processOrder(double[] prices) throws SQLException {
        double total = 0;
        for (double p : prices) total += p;
        Connection c = DriverManager.getConnection(url);
        PreparedStatement s = c.prepareStatement("INSERT ...");
        s.executeUpdate();
        c.close();
    }
}
```

```python
# Python
class OrderService:
    def process_order(self, prices):
        total = sum(prices)
        import sqlite3
        conn = sqlite3.connect(db)
        conn.execute("INSERT ...", (total,))
        conn.commit()
        conn.close()
```

#### Respecting SRP

```csharp
// C#
public class OrderCalculator
{
    public double CalculateTotal(double[] prices)
    {
        double t = 0;
        foreach (double p in prices) t += p;
        return t;
    }
}

public class OrderRepository
{
    public void Save(double total) { /* persist */ }
}

public class OrderService
{
    private readonly OrderCalculator _calc;
    private readonly OrderRepository _repo;
    public OrderService(OrderCalculator c, OrderRepository r) { _calc=c; _repo=r; }
    public void Process(double[] prices)
    {
        var total = _calc.CalculateTotal(prices);
        _repo.Save(total);
    }
}
```

```java
// Java
public class OrderCalculator { double calculateTotal(double[] p){...} }
public class OrderRepository{ void save(double t){...} }
public class OrderService {
  private final OrderCalculator c; private final OrderRepository r;
  public OrderService(OrderCalculator c,OrderRepository r){this.c=c;this.r=r;}
  public void process(double[] p){
    double t = c.calculateTotal(p); r.save(t);
  }
}
```

```python
# Python
class OrderCalculator:
    def calculate_total(self, prices): return sum(prices)

class OrderRepository:
    def save(self, total): ...

class OrderService:
    def __init__(self, calc, repo):
        self._calc = calc; self._repo = repo
    def process(self, prices):
        t = self._calc.calculate_total(prices)
        self._repo.save(t)
```

---

## 5. Deep Dive: OCP (Open/Closed Principle)

> “Open for extension, closed for modification.” – Bertrand Meyer

**Key Concepts**

* Depend on abstractions, not concrete types
* Extend behavior via new subclasses or implementations
* Avoid `if`/`switch` chains on type

### Code Examples

#### Natural Language

Sum areas of shapes, new shapes added without touching the summing logic.

#### Violating OCP

```csharp
// C#
public class AreaCalculator
{
    public double TotalArea(IEnumerable<object> shapes)
    {
        double total=0;
        foreach(var s in shapes)
        {
            if(s is Circle c) total+=Math.PI*c.Radius*c.Radius;
            else if(s is Rectangle r) total+=r.Width*r.Height;
        }
        return total;
    }
}
```

```java
// Java
public double totalArea(List<Object> shapes){
  double t=0;
  for(Object s: shapes){
    if(s instanceof Circle) t+=...;
    else if(s instanceof Rectangle) t+=...;
  }
  return t;
}
```

```python
# Python
def total_area(shapes):
    t=0
    for s in shapes:
        if isinstance(s, Circle): t+=math.pi*s.radius**2
        elif isinstance(s, Rectangle): t+=s.width*s.height
    return t
```

#### Respecting OCP

```csharp
// C#
public interface IShape { double Area(); }
public class Circle : IShape { /*...*/ public double Area() => ...; }
public class Rectangle : IShape { /*...*/ public double Area() => ...; }
public class AreaCalculator
{
    public double TotalArea(IEnumerable<IShape> shapes)
      => shapes.Sum(s=>s.Area());
}
```

```java
// Java
public interface Shape { double area(); }
public class Circle implements Shape { public double area(){...} }
public class Rectangle implements Shape { public double area(){...} }
public class AreaCalculator {
  public double totalArea(List<Shape> shapes){
    return shapes.stream().mapToDouble(Shape::area).sum();
  }
}
```

```python
# Python
from abc import ABC, abstractmethod
class Shape(ABC):
    @abstractmethod
    def area(self): ...
class Circle(Shape): ...
class Rectangle(Shape): ...
def total_area(shapes: list[Shape]) -> float:
    return sum(s.area() for s in shapes)
```

---

## 6. Deep Dive: LSP (Liskov Substitution Principle)

> “Objects of a superclass shall be replaceable with objects of its subclasses without breaking the application.” – Barbara Liskov

**Key Concepts**

* Subclasses must honor base class contracts (no stronger preconditions, no weaker postconditions)
* Avoid unexpected exceptions or behavior changes

### Code Examples

#### Natural Language

A `Bird` abstraction with `Fly()`: penguins cannot fly.

#### Violating LSP

```csharp
// C#
public abstract class Bird { public abstract void Fly(); }
public class Sparrow : Bird { public override void Fly() { /* ok */ } }
public class Penguin : Bird { public override void Fly() 
  => throw new NotSupportedException(); }
```

```java
// Java
abstract class Bird { abstract void fly(); }
class Sparrow extends Bird{ void fly(){...} }
class Penguin extends Bird{ void fly(){throw new...;} }
```

```python
# Python
class Bird: def fly(self): raise NotImplementedError
class Sparrow(Bird): def fly(self): print("flying")
class Penguin(Bird): def fly(self): raise Exception("no fly")
```

#### Respecting LSP

```csharp
// C#
public abstract class Bird { public string Name{get;} protected Bird(string n)=>Name=n; }
public interface IFlyable { void Fly(); }
public class Sparrow : Bird, IFlyable { public void Fly()=>...; }
public class Penguin : Bird { /* no Fly() */ }
```

```java
// Java
abstract class Bird{ String name; Bird(String n){name=n;} }
interface Flyable{ void fly(); }
class Sparrow extends Bird implements Flyable{ public void fly(){...} }
class Penguin extends Bird{ /* no fly() */ }
```

```python
# Python
from abc import ABC,abstractmethod
class Bird(ABC): ...
class Flyable(ABC): @abstractmethod def fly(self): ...
class Sparrow(Bird,Flyable): def fly(self): ...
class Penguin(Bird): pass
```

---

## 7. Deep Dive: ISP (Interface Segregation Principle)

> “Clients should not be forced to depend on methods they do not use.” – Robert C. Martin

**Key Concepts**

* Break large “fat” interfaces into client‑specific, focused interfaces
* Avoid forcing implementers to stub unused methods

### Code Examples

#### Natural Language

A multifunction device: print, scan, fax.

#### Violating ISP

```csharp
// C#
public interface IMachine {
  void Print(Document d);
  void Scan(Document d);
  void Fax(Document d);
}
public class Printer : IMachine {
  public void Print(Document d){...}
  public void Scan(Document d)=>throw new NotImplementedException();
  public void Fax(Document d)=>throw new NotImplementedException();
}
```

```java
// Java
interface Machine{ void print(D); void scan(D); void fax(D); }
class Printer implements Machine{ 
  public void print(D d){...}
  public void scan(D d){throw new...;}
  public void fax(D d){throw new...;} 
}
```

```python
# Python
class Machine:
    def print(self,d):...
    def scan(self,d):...
    def fax(self,d):...
class Printer(Machine):
    def print(self,d):...
    def scan(self,d): raise NotImplementedError
    def fax(self,d): raise NotImplementedError
```

#### Respecting ISP

```csharp
// C#
public interface IPrinter { void Print(Document d); }
public interface IScanner { void Scan(Document d); }
public interface IFaxer { void Fax(Document d); }
public class Printer : IPrinter { public void Print(Document d){...} }
```

```java
// Java
interface Printer{ void print(D d); }
interface Scanner{ void scan(D d); }
interface Faxer{ void fax(D d); }
class PrinterImpl implements Printer{ public void print(D d){...} }
```

```python
# Python
from abc import ABC,abstractmethod
class Printer(ABC): @abstractmethod
    def print(self,d): ...
class Scanner(ABC): @abstractmethod
    def scan(self,d): ...
class Faxer(ABC): @abstractmethod
    def fax(self,d): ...
class PrinterImpl(Printer):
    def print(self,d): print("printing")
```

---

## 8. Deep Dive: DIP (Dependency Inversion Principle)

> 1. High‑level modules should not depend on low‑level modules. Both depend on abstractions.
> 2. Abstractions should not depend on details. Details depend on abstractions. – Robert C. Martin

**Key Concepts**

* Inversion of Control via Dependency Injection
* High‑level logic depends on interfaces, not concretes

### Code Examples

#### Natural Language

Order processor sends notifications via email or SMS.

#### Violating DIP

```csharp
// C#
public class EmailSender { public void Send(string to,string m){...} }
public class OrderProcessor {
    private EmailSender _es = new EmailSender();
    public void Process(Order o){ _es.Send(o.Email, "…"); }
}
```

```java
// Java
class EmailSender{ void send(String to,String m){...}}
class OrderProcessor{
  private EmailSender es=new EmailSender();
  void process(Order o){ es.send(o.getEmail(),"..."); }
}
```

```python
# Python
class EmailSender:
    def send(self,to,msg): ...
class OrderProcessor:
    def __init__(self): self._es=EmailSender()
    def process(self,o): self._es.send(o.email,"…")
```

#### Respecting DIP

```csharp
// C#
public interface INotifier { void Notify(string r,string m); }
public class EmailNotifier:INotifier{ public void Notify(string r,string m){...} }
public class OrderProcessor {
    private readonly INotifier _n;
    public OrderProcessor(INotifier n){_n=n;}
    public void Process(Order o){ _n.Notify(o.Email,"…"); }
}
```

```java
// Java
interface Notifier{ void notify(String r,String m); }
class EmailNotifier implements Notifier{ public void notify(...){...} }
class OrderProcessor{
  private Notifier n;
  OrderProcessor(Notifier n){this.n=n;}
  void process(Order o){ n.notify(o.getEmail(),"..."); }
}
```

```python
# Python
from abc import ABC,abstractmethod
class Notifier(ABC):
    @abstractmethod
    def notify(self,to,msg): ...
class EmailNotifier(Notifier):
    def notify(self,to,msg): ...
class OrderProcessor:
    def __init__(self,notifier:Notifier): self._notifier=notifier
    def process(self,o): self._notifier.notify(o.email,"…")
```

---

## 9. Combined Example: All Five Principles

Below are three implementations (C#, Java, Python) of a simple order processing flow that illustrate **all five SOLID principles** together. Comments in the code point out where each principle is applied.

---

### C# (.NET)

```csharp
using System;
using System.Collections.Generic;

#region Domain Model
public class Order
{
    public int Id { get; }
    public string CustomerEmail { get; }
    public List<double> ItemPrices { get; }
    public double Total { get; private set; }

    public Order(int id, string customerEmail, List<double> itemPrices)
    {
        Id = id;
        CustomerEmail = customerEmail;
        ItemPrices = itemPrices;
    }

    public void SetTotal(double total)
    {
        Total = total;
    }
}
#endregion

#region Interfaces (ISP, DIP)
// ISP: clients depend only on the methods they need
public interface IOrderTotalCalculator
{
    double CalculateTotal(Order order);
}

public interface IShippingCostStrategy
{
    double CalculateShippingCost(Order order);
}

public interface IOrderRepository
{
    void Save(Order order);
}

public interface INotifier
{
    void Notify(string recipient, string message);
}
#endregion

#region Implementations (SRP, OCP, LSP)
// SRP: each class has one responsibility
// OCP & LSP: new calculators or shipping strategies can be added without modifying existing code
public class SimpleOrderTotalCalculator : IOrderTotalCalculator
{
    public double CalculateTotal(Order order)
    {
        double sum = 0.0;
        foreach (double price in order.ItemPrices)
        {
            sum += price;
        }
        return sum;
    }
}

public class FlatRateShippingStrategy : IShippingCostStrategy
{
    private const double Rate = 5.0;
    public double CalculateShippingCost(Order order) => Rate;
}

public class Repository : IOrderRepository
{
    public void Save(Order order)
    {
        // Persist order to database...
        Console.WriteLine($"Order {order.Id} saved with total {order.Total:C}.");
    }
}

public class EmailNotifier : INotifier
{
    public void Notify(string recipient, string message)
    {
        // Send email...
        Console.WriteLine($"Email to {recipient}: {message}");
    }
}
#endregion

#region Orchestration (DIP, SRP)
// DIP: depends on abstractions, not concretes
public class OrderProcessor
{
    private readonly IOrderTotalCalculator _calculator;
    private readonly IShippingCostStrategy _shippingStrategy;
    private readonly IOrderRepository _repository;
    private readonly INotifier _notifier;

    public OrderProcessor(
        IOrderTotalCalculator calculator,
        IShippingCostStrategy shippingStrategy,
        IOrderRepository repository,
        INotifier notifier)
    {
        _calculator = calculator;
        _shippingStrategy = shippingStrategy;
        _repository = repository;
        _notifier = notifier;
    }

    public void Process(Order order)
    {
        // SRP: this class orchestrates only
        double total = _calculator.CalculateTotal(order);
        double shipping = _shippingStrategy.CalculateShippingCost(order);

        order.SetTotal(total + shipping);

        _repository.Save(order);
        _notifier.Notify(order.CustomerEmail, $"Your order #{order.Id} has been processed. Total: {order.Total:C}");
    }
}
#endregion

// Usage:
// var processor = new OrderProcessor(new SimpleOrderTotalCalculator(),
//     new FlatRateShippingStrategy(), new Repository(), new EmailNotifier());
// processor.Process(new Order(1, "customer@example.com", new List<double>{10,20,30}));

```

```java
import java.util.List;

#region Domain Model
public class Order {
    private final int id;
    private final String customerEmail;
    private final List<Double> itemPrices;
    private double total;

    public Order(int id, String customerEmail, List<Double> itemPrices) {
        this.id = id;
        this.customerEmail = customerEmail;
        this.itemPrices = itemPrices;
    }

    public int getId() { return id; }
    public String getCustomerEmail() { return customerEmail; }
    public List<Double> getItemPrices() { return itemPrices; }
    public double getTotal() { return total; }
    public void setTotal(double total) { this.total = total; }
}
#endregion

#region Interfaces (ISP, DIP)
public interface OrderTotalCalculator {
    double calculateTotal(Order order);
}

public interface ShippingCostStrategy {
    double calculateShippingCost(Order order);
}

public interface OrderRepository {
    void save(Order order);
}

public interface Notifier {
    void notify(String recipient, String message);
}
#endregion

#region Implementations (SRP, OCP, LSP)
public class SimpleOrderTotalCalculator implements OrderTotalCalculator {
    public double calculateTotal(Order order) {
        double sum = 0.0;
        for (double price : order.getItemPrices()) {
            sum += price;
        }
        return sum;
    }
}

public class FlatRateShippingStrategy implements ShippingCostStrategy {
    private static final double RATE = 5.0;
    public double calculateShippingCost(Order order) {
        return RATE;
    }
}

public class Repository implements OrderRepository {
    public void save(Order order) {
        // Persist order...
        System.out.printf("Order %d saved with total %.2f.%n", order.getId(), order.getTotal());
    }
}

public class EmailNotifier implements Notifier {
    public void notify(String recipient, String message) {
        // Send email...
        System.out.printf("Email to %s: %s%n", recipient, message);
    }
}
#endregion

#region Orchestration (DIP, SRP)
public class OrderProcessor {
    private final OrderTotalCalculator calculator;
    private final ShippingCostStrategy shippingStrategy;
    private final OrderRepository repository;
    private final Notifier notifier;

    public OrderProcessor(
        OrderTotalCalculator calculator,
        ShippingCostStrategy shippingStrategy,
        OrderRepository repository,
        Notifier notifier)
    {
        this.calculator = calculator;
        this.shippingStrategy = shippingStrategy;
        this.repository = repository;
        this.notifier = notifier;
    }

    public void process(Order order) {
        double total = calculator.calculateTotal(order);          // SRP: orchestration only
        double shipping = shippingStrategy.calculateShippingCost(order);

        order.setTotal(total + shipping);

        repository.save(order);
        notifier.notify(order.getCustomerEmail(),
            "Your order #" + order.getId() + " has been processed. Total: " + order.getTotal());
    }
}
#endregion

// Usage:
// OrderProcessor p = new OrderProcessor(
//     new SimpleOrderTotalCalculator(),
//     new FlatRateShippingStrategy(),
//     new Repository(),
//     new EmailNotifier());
// p.process(new Order(1, "customer@example.com", List.of(10.0, 20.0, 30.0)));

```

```python
from abc import ABC, abstractmethod
from typing import List

#region Domain Model
class Order:
    def __init__(self, id: int, customer_email: str, item_prices: List[float]) -> None:
        self.id = id
        self.customer_email = customer_email
        self.item_prices = item_prices
        self.total = 0.0

    def set_total(self, total: float) -> None:
        self.total = total
#endregion

#region Interfaces (ISP, DIP)
class IOrderTotalCalculator(ABC):
    @abstractmethod
    def calculate_total(self, order: Order) -> float:
        ...

class IShippingCostStrategy(ABC):
    @abstractmethod
    def calculate_shipping_cost(self, order: Order) -> float:
        ...

class IOrderRepository(ABC):
    @abstractmethod
    def save(self, order: Order) -> None:
        ...

class INotifier(ABC):
    @abstractmethod
    def notify(self, recipient: str, message: str) -> None:
        ...
#endregion

#region Implementations (SRP, OCP, LSP)
class SimpleOrderTotalCalculator(IOrderTotalCalculator):
    def calculate_total(self, order: Order) -> float:
        return sum(order.item_prices)

class FlatRateShippingStrategy(IShippingCostStrategy):
    RATE = 5.0
    def calculate_shipping_cost(self, order: Order) -> float:
        return self.RATE

class Repository(IOrderRepository):
    def save(self, order: Order) -> None:
        # Persist order...
        print(f"Order {order.id} saved with total {order.total:.2f}.")

class EmailNotifier(INotifier):
    def notify(self, recipient: str, message: str) -> None:
        # Send email...
        print(f"Email to {recipient}: {message}")
#endregion

#region Orchestration (DIP, SRP)
class OrderProcessor:
    def __init__(
        self,
        calculator: IOrderTotalCalculator,
        shipping_strategy: IShippingCostStrategy,
        repository: IOrderRepository,
        notifier: INotifier
    ) -> None:
        self._calculator = calculator        # DIP: depends on abstractions
        self._shipping_strategy = shipping_strategy
        self._repository = repository
        self._notifier = notifier

    def process(self, order: Order) -> None:
        # SRP: only orchestrates workflow
        total = self._calculator.calculate_total(order)
        shipping = self._shipping_strategy.calculate_shipping_cost(order)

        order.set_total(total + shipping)

        self._repository.save(order)
        self._notifier.notify(
            order.customer_email,
            f"Your order #{order.id} has been processed. Total: {order.total:.2f}"
        )
#endregion

# Usage:
# processor = OrderProcessor(
#     SimpleOrderTotalCalculator(),
#     FlatRateShippingStrategy(),
#     Repository(),
#     EmailNotifier()
# )
# processor.process(Order(1, "customer@example.com", [10.0, 20.0, 30.0]))
```

---

## 10. Summary

* **SOLID** ensures software is modular, extensible, and maintainable.
* Each principle addresses a different design concern:

  * SRP: one reason to change
  * OCP: extend without modifying
  * LSP: safe subtype replacement
  * ISP: focused interfaces
  * DIP: depend on abstractions
* When practiced together, they create code that is easy to read, test, evolve, and refactor.

---

## 11. Sources

* Oloruntoba, Samuel & Walia, Anish Singh. “SOLID: The First 5 Principles of Object Oriented Design,” DigitalOcean, Apr 23 2024.
* Krishna, Ashutosh. “What Is SOLID? Principles for Better Software Design,” freeCodeCamp.
* Nordelo, Eric. “SOLID: The First Object‑Oriented Design Principles through Examples,” Jun 2 2021.
* “SOLID: 5 Principles of Software Design,” TheTechTangent.
* “Why SOLID Principles Are Still the Foundation for Modern Software Architecture,” Stack Overflow Blog, Nov 1 2021.
* Milo, Martin. “SOLID Principles: Explained on Practical Examples,” martinmilo.com.
* Kimani, Robert. “An Engineering Leader’s Guide to SOLID Principles,” LeadDev, Mar 6 2025.
