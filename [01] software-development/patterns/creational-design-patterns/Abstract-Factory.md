# Abstract Factory Pattern

## Short Description

**Abstract Factory** provides an interface for creating families of related or dependent objects without specifying their concrete classes.
It ensures that objects from the same family (e.g., “Fantasy assets”) are used together, preserving visual and behavioral consistency.

---

## Comprehensive Overview

The **Abstract Factory** pattern is one of the creational design patterns defined by the *Gang of Four (GoF)*.
It solves the problem of creating related objects without binding the client to concrete classes.
Instead of calling `new` directly, the client relies on an abstract factory that produces objects belonging to the same “family.”

For example, in a game engine:

* The **FantasyFactory** might create `ElfCharacter`, `MagicSword`, and `EnchantedForest`.
* The **SciFiFactory** might create `CyborgCharacter`, `LaserGun`, and `SpaceStation`.

The game engine doesn’t care which world is loaded — it interacts only with abstract interfaces (`ICharacter`, `IWeapon`, `IEnvironment`), allowing seamless switching between asset sets.

This pattern is crucial for maintaining *cross-product consistency* — ensuring that products created together look and behave as part of the same theme.

---

## Problems this topic solves

1. **Cross-family consistency** — ensures that related products (e.g., character, weapon, environment) match each other.
2. **Decoupling from concrete implementations** — clients depend only on abstract interfaces.
3. **Configurable product families** — you can switch the entire product family (Fantasy ↔ Sci-Fi) at runtime.
4. **Extensibility for new families** — adding new families (e.g., “Post-Apocalyptic”) doesn’t require changing existing code.

---

## Limitations — what it does *not* solve

* Adding **new product types** (like a new “Vehicle” type) requires changing *all* factories and interfaces.
* Can introduce **many extra classes and interfaces** (factory + multiple product abstractions).
* Overkill if there’s only one family or few products.
* Complex hierarchies can make the code harder to follow without clear naming and organization.

---

## When to use

* You have multiple **interchangeable families** of related products (themes, platforms, brands).
* You must guarantee that products from one family work correctly together.
* You want to easily **switch families** at runtime (config, environment, or player choice).

---

## When to avoid

* You have only one family or rarely switch between them.
* The number of **product types** changes frequently — maintenance becomes costly.
* Simpler creational patterns (Factory Method, Prototype) would suffice.

---

## Implementation Examples

The following examples show a **Game Asset Factory** that can produce either *Fantasy* or *Sci-Fi* assets.
The client (`GameWorld`) uses only abstract interfaces, and can load any factory at runtime.

---

### C# Example

```csharp
using System;

// ===== Abstract Products =====
public interface ICharacter
{
    void Attack();
}

public interface IWeapon
{
    void Use();
}

public interface IEnvironment
{
    void Load();
}

// ===== Concrete Fantasy Products =====
public class ElfCharacter : ICharacter
{
    public void Attack() => Console.WriteLine("Elf shoots an arrow!");
}

public class MagicSword : IWeapon
{
    public void Use() => Console.WriteLine("Swinging a glowing magic sword!");
}

public class EnchantedForest : IEnvironment
{
    public void Load() => Console.WriteLine("Entering the enchanted forest...");
}

// ===== Concrete Sci-Fi Products =====
public class CyborgCharacter : ICharacter
{
    public void Attack() => Console.WriteLine("Cyborg fires plasma cannon!");
}

public class LaserGun : IWeapon
{
    public void Use() => Console.WriteLine("Pew pew! Laser gun engaged.");
}

public class SpaceStation : IEnvironment
{
    public void Load() => Console.WriteLine("Docking at orbital space station...");
}

// ===== Abstract Factory =====
public interface IGameAssetFactory
{
    ICharacter CreateCharacter();
    IWeapon CreateWeapon();
    IEnvironment CreateEnvironment();
}

// ===== Concrete Factories =====
public class FantasyFactory : IGameAssetFactory
{
    public ICharacter CreateCharacter() => new ElfCharacter();
    public IWeapon CreateWeapon() => new MagicSword();
    public IEnvironment CreateEnvironment() => new EnchantedForest();
}

public class SciFiFactory : IGameAssetFactory
{
    public ICharacter CreateCharacter() => new CyborgCharacter();
    public IWeapon CreateWeapon() => new LaserGun();
    public IEnvironment CreateEnvironment() => new SpaceStation();
}

// ===== Client =====
public class GameWorld
{
    private readonly ICharacter _character;
    private readonly IWeapon _weapon;
    private readonly IEnvironment _environment;

    public GameWorld(IGameAssetFactory factory)
    {
        _character = factory.CreateCharacter();
        _weapon = factory.CreateWeapon();
        _environment = factory.CreateEnvironment();
    }

    public void LoadWorld()
    {
        _environment.Load();
        _character.Attack();
        _weapon.Use();
    }
}

// ===== Usage =====
// IGameAssetFactory factory = new SciFiFactory();
// var world = new GameWorld(factory);
// world.LoadWorld();
```

---

### Java 21 Example

```java
// ===== Abstract Products =====
interface Character {
    void attack();
}

interface Weapon {
    void use();
}

interface Environment {
    void load();
}

// ===== Fantasy Products =====
class ElfCharacter implements Character {
    public void attack() { System.out.println("Elf shoots an arrow!"); }
}

class MagicSword implements Weapon {
    public void use() { System.out.println("Swinging a glowing magic sword!"); }
}

class EnchantedForest implements Environment {
    public void load() { System.out.println("Entering the enchanted forest..."); }
}

// ===== Sci-Fi Products =====
class CyborgCharacter implements Character {
    public void attack() { System.out.println("Cyborg fires plasma cannon!"); }
}

class LaserGun implements Weapon {
    public void use() { System.out.println("Pew pew! Laser gun engaged."); }
}

class SpaceStation implements Environment {
    public void load() { System.out.println("Docking at orbital space station..."); }
}

// ===== Abstract Factory =====
interface GameAssetFactory {
    Character createCharacter();
    Weapon createWeapon();
    Environment createEnvironment();
}

// ===== Concrete Factories =====
class FantasyFactory implements GameAssetFactory {
    public Character createCharacter() { return new ElfCharacter(); }
    public Weapon createWeapon() { return new MagicSword(); }
    public Environment createEnvironment() { return new EnchantedForest(); }
}

class SciFiFactory implements GameAssetFactory {
    public Character createCharacter() { return new CyborgCharacter(); }
    public Weapon createWeapon() { return new LaserGun(); }
    public Environment createEnvironment() { return new SpaceStation(); }
}

// ===== Client =====
class GameWorld {
    private final Character character;
    private final Weapon weapon;
    private final Environment environment;

    public GameWorld(GameAssetFactory factory) {
        this.character = factory.createCharacter();
        this.weapon = factory.createWeapon();
        this.environment = factory.createEnvironment();
    }

    public void loadWorld() {
        environment.load();
        character.attack();
        weapon.use();
    }
}

// ===== Usage =====
// GameAssetFactory factory = new FantasyFactory();
// GameWorld world = new GameWorld(factory);
// world.loadWorld();
```

---

### Python 13 Example

```python
from abc import ABC, abstractmethod

# ===== Abstract Products =====
class Character(ABC):
    @abstractmethod
    def attack(self) -> None: ...

class Weapon(ABC):
    @abstractmethod
    def use(self) -> None: ...

class Environment(ABC):
    @abstractmethod
    def load(self) -> None: ...

# ===== Fantasy Products =====
class ElfCharacter(Character):
    def attack(self) -> None:
        print("Elf shoots an arrow!")

class MagicSword(Weapon):
    def use(self) -> None:
        print("Swinging a glowing magic sword!")

class EnchantedForest(Environment):
    def load(self) -> None:
        print("Entering the enchanted forest...")

# ===== Sci-Fi Products =====
class CyborgCharacter(Character):
    def attack(self) -> None:
        print("Cyborg fires plasma cannon!")

class LaserGun(Weapon):
    def use(self) -> None:
        print("Pew pew! Laser gun engaged.")

class SpaceStation(Environment):
    def load(self) -> None:
        print("Docking at orbital space station...")

# ===== Abstract Factory =====
class GameAssetFactory(ABC):
    @abstractmethod
    def create_character(self) -> Character: ...
    @abstractmethod
    def create_weapon(self) -> Weapon: ...
    @abstractmethod
    def create_environment(self) -> Environment: ...

# ===== Concrete Factories =====
class FantasyFactory(GameAssetFactory):
    def create_character(self) -> Character: return ElfCharacter()
    def create_weapon(self) -> Weapon: return MagicSword()
    def create_environment(self) -> Environment: return EnchantedForest()

class SciFiFactory(GameAssetFactory):
    def create_character(self) -> Character: return CyborgCharacter()
    def create_weapon(self) -> Weapon: return LaserGun()
    def create_environment(self) -> Environment: return SpaceStation()

# ===== Client =====
class GameWorld:
    def __init__(self, factory: GameAssetFactory) -> None:
        self.character = factory.create_character()
        self.weapon = factory.create_weapon()
        self.environment = factory.create_environment()

    def load_world(self) -> None:
        self.environment.load()
        self.character.attack()
        self.weapon.use()

# ===== Usage =====
# factory = FantasyFactory()
# world = GameWorld(factory)
# world.load_world()
```

---

## Instructions for AIs (Knowledge Source Usage)

**When an AI coding assistant should use Abstract Factory:**

* When the problem involves **families of related products** (UI elements, themes, platforms, game assets).
* When product families must remain consistent or interchangeable at runtime.
* When decoupling creation logic from usage improves maintainability or testability.

**When it should not:**

* When only one product family exists.
* When adding new product types is more common than adding new families.
* When excessive indirection hurts readability more than it helps flexibility.

**How it relates to other patterns:**

* **Factory Method:** Abstract Factory often uses Factory Methods internally.
* **Builder:** focuses on building one complex product; Abstract Factory creates *families* of related products.
* **Prototype:** can be used by a factory to clone existing prototypes instead of instantiating new objects.
* **Dependency Injection:** both decouple object creation, but Abstract Factory focuses on product families, DI focuses on configuration and lifecycle.

**Cares and considerations:**

* Keep naming consistent: each family and product type should be clearly labeled.
* Group related factories logically (e.g., “FantasyFactory”, “SciFiFactory”).
* Document which families are supported and what each product type does.
* Don’t abuse the pattern for single-product cases — use simpler factories instead.
* Keep interfaces cohesive: don’t mix unrelated responsibilities in one factory.

**Common pitfalls:**

* Adding a new product type requires editing all factories — plan for that.
* Overuse can make code hierarchies large and intimidating.
* Using reflection or configuration to select factories can obscure control flow if not documented.

---

## References and Bibliography

* Gamma, Erich et al. *Design Patterns: Elements of Reusable Object-Oriented Software*. Addison-Wesley, 1994.
* Refactoring.Guru — Abstract Factory Pattern. [https://refactoring.guru/design-patterns/abstract-factory](https://refactoring.guru/design-patterns/abstract-factory)
* Martin Fowler — *Inversion of Control Containers and the Dependency Injection Pattern*. [https://martinfowler.com/articles/injection.html](https://martinfowler.com/articles/injection.html)
* Oracle Java Tutorials — Design Patterns Overview.
* Microsoft Docs — “Abstract Factory Design Pattern in C#”.
* Real Python — “Design Patterns in Python: Abstract Factory.”

---

## Quick Summary / Mnemonic

**“Families stay together.”**
Use Abstract Factory when you want entire *families of products* to be created and used consistently without the client knowing the details.

---

Would you like me to now generate this into a downloadable `.md` file (like we did for Builder)?
