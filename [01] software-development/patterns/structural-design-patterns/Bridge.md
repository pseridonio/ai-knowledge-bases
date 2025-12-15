# Bridge Pattern

## Short Description

The **Bridge** pattern decouples an abstraction from its implementation, allowing both to evolve independently.
It’s often used to separate high-level logic (e.g., “Document”) from low-level system details (e.g., “Storage Provider”), avoiding rigid inheritance hierarchies.

---

## Comprehensive Overview

In traditional inheritance structures, if you have multiple dimensions of variation — for example:

* Types of **Documents** (`PDFDocument`, `WordDocument`, `ReportDocument`)
* Types of **Storage** (`LocalStorage`, `CloudStorage`, `EncryptedStorage`)

—you might end up creating combinations like:
`PDFCloudDocument`, `WordLocalDocument`, `ReportEncryptedDocument`, etc.
This quickly becomes unmanageable — a **class explosion problem**.

The **Bridge** pattern solves this by separating these hierarchies:

* **Abstraction** — defines the high-level interface (e.g., `Document`).
* **Implementor** — defines the low-level interface (e.g., `StorageProvider`).

Now, instead of inheritance, the abstraction *has a reference* to the implementor.
That allows you to combine any abstraction with any implementation at runtime — for example, a `PDFDocument` stored in `EncryptedCloudStorage`.

**Analogy:**
Think of a TV remote and a TV. The remote (abstraction) can control different TVs (implementations). You can upgrade either side without breaking the other.

---

## Problems this topic solves

* **Rigid inheritance hierarchies:** Avoids class explosion from multiple independent dimensions.
* **Tight coupling between abstraction and implementation.**
* **Difficult maintenance:** Enables independent evolution of high-level and low-level components.
* **Runtime flexibility:** You can switch implementations dynamically.

---

## Limitations — what it does *not* solve

* Adds an **extra layer of indirection**, increasing complexity for simple systems.
* Requires careful interface design for both abstraction and implementor.
* Not useful if you have only one implementation dimension.
* Can overcomplicate systems that don’t require runtime flexibility.

---

## When to use

* You have **multiple dimensions of variation** (e.g., different devices and renderers).
* You want to **combine abstractions and implementations freely**.
* Both abstraction and implementation may need to evolve independently.
* You need **runtime flexibility** to switch implementations.

---

## When to avoid

* You have a simple hierarchy — inheritance is enough.
* Abstraction and implementation are permanently linked.
* The additional abstraction layer adds more confusion than benefit.

---

## Implementation Examples

We’ll build a **Document Management System (DMS)**:

* **Abstraction:** `Document` defines operations like `save()` and `open()`.
* **Implementors:** `StorageProvider` defines operations like `write()` and `read()`.
  Different combinations (e.g., `PDFDocument` + `CloudStorage`) can coexist without new subclasses.

---

### C# Example

```csharp
using System;

// ===== Implementor =====
public interface IStorageProvider
{
    void Write(string filename, string content);
    string Read(string filename);
}

// ===== Concrete Implementors =====
public class LocalStorage : IStorageProvider
{
    public void Write(string filename, string content)
        => Console.WriteLine($"[Local] Saving {filename}: {content}");
    public string Read(string filename)
        => $"[Local] Reading file: {filename}";
}

public class CloudStorage : IStorageProvider
{
    public void Write(string filename, string content)
        => Console.WriteLine($"[Cloud] Uploading {filename}: {content}");
    public string Read(string filename)
        => $"[Cloud] Downloading file: {filename}";
}

public class EncryptedStorage : IStorageProvider
{
    public void Write(string filename, string content)
        => Console.WriteLine($"[Encrypted] Writing {filename} (encrypted data)");
    public string Read(string filename)
        => $"[Encrypted] Reading file: {filename} (decrypted)";
}

// ===== Abstraction =====
public abstract class Document
{
    protected IStorageProvider storageProvider;

    protected Document(IStorageProvider provider)
    {
        storageProvider = provider;
    }

    public abstract void Save(string content);
    public abstract void Open();
}

// ===== Refined Abstractions =====
public class PDFDocument : Document
{
    public PDFDocument(IStorageProvider provider) : base(provider) { }

    public override void Save(string content)
        => storageProvider.Write("report.pdf", content);

    public override void Open()
        => Console.WriteLine(storageProvider.Read("report.pdf"));
}

public class WordDocument : Document
{
    public WordDocument(IStorageProvider provider) : base(provider) { }

    public override void Save(string content)
        => storageProvider.Write("document.docx", content);

    public override void Open()
        => Console.WriteLine(storageProvider.Read("document.docx"));
}

// ===== Usage =====
// Document doc = new PDFDocument(new CloudStorage());
// doc.Save("Quarterly report data");
// doc.Open();
```

---

### Java 21 Example

```java
// ===== Implementor =====
interface StorageProvider {
    void write(String filename, String content);
    String read(String filename);
}

// ===== Concrete Implementors =====
class LocalStorage implements StorageProvider {
    public void write(String filename, String content) {
        System.out.println("[Local] Saving " + filename + ": " + content);
    }
    public String read(String filename) {
        return "[Local] Reading file: " + filename;
    }
}

class CloudStorage implements StorageProvider {
    public void write(String filename, String content) {
        System.out.println("[Cloud] Uploading " + filename + ": " + content);
    }
    public String read(String filename) {
        return "[Cloud] Downloading file: " + filename;
    }
}

class EncryptedStorage implements StorageProvider {
    public void write(String filename, String content) {
        System.out.println("[Encrypted] Writing " + filename + " (encrypted)");
    }
    public String read(String filename) {
        return "[Encrypted] Reading " + filename + " (decrypted)";
    }
}

// ===== Abstraction =====
abstract class Document {
    protected StorageProvider provider;
    protected Document(StorageProvider provider) { this.provider = provider; }

    abstract void save(String content);
    abstract void open();
}

// ===== Refined Abstractions =====
class PDFDocument extends Document {
    public PDFDocument(StorageProvider provider) { super(provider); }
    void save(String content) { provider.write("report.pdf", content); }
    void open() { System.out.println(provider.read("report.pdf")); }
}

class WordDocument extends Document {
    public WordDocument(StorageProvider provider) { super(provider); }
    void save(String content) { provider.write("document.docx", content); }
    void open() { System.out.println(provider.read("document.docx")); }
}

// ===== Usage =====
// Document doc = new WordDocument(new EncryptedStorage());
// doc.save("Confidential data");
// doc.open();
```

---

### Python 13 Example

```python
from abc import ABC, abstractmethod

# ===== Implementor =====
class StorageProvider(ABC):
    @abstractmethod
    def write(self, filename: str, content: str) -> None: ...
    @abstractmethod
    def read(self, filename: str) -> str: ...

# ===== Concrete Implementors =====
class LocalStorage(StorageProvider):
    def write(self, filename: str, content: str) -> None:
        print(f"[Local] Saving {filename}: {content}")

    def read(self, filename: str) -> str:
        return f"[Local] Reading file: {filename}"

class CloudStorage(StorageProvider):
    def write(self, filename: str, content: str) -> None:
        print(f"[Cloud] Uploading {filename}: {content}")

    def read(self, filename: str) -> str:
        return f"[Cloud] Downloading file: {filename}"

class EncryptedStorage(StorageProvider):
    def write(self, filename: str, content: str) -> None:
        print(f"[Encrypted] Writing {filename} (encrypted data)")

    def read(self, filename: str) -> str:
        return f"[Encrypted] Reading {filename} (decrypted data)"

# ===== Abstraction =====
class Document(ABC):
    def __init__(self, provider: StorageProvider) -> None:
        self.provider = provider

    @abstractmethod
    def save(self, content: str) -> None: ...
    @abstractmethod
    def open(self) -> None: ...

# ===== Refined Abstractions =====
class PDFDocument(Document):
    def save(self, content: str) -> None:
        self.provider.write("report.pdf", content)

    def open(self) -> None:
        print(self.provider.read("report.pdf"))

class WordDocument(Document):
    def save(self, content: str) -> None:
        self.provider.write("document.docx", content)

    def open(self) -> None:
        print(self.provider.read("document.docx"))

# ===== Usage =====
# doc = PDFDocument(CloudStorage())
# doc.save("Annual Report Data")
# doc.open()
```

---

## Instructions for AIs (Knowledge Source Usage)

**When to apply the Bridge pattern:**

* When there are **two orthogonal dimensions** (abstraction and implementation).
* When inheritance leads to class explosion.
* When you want to switch implementations dynamically at runtime.
* When abstraction and implementation should evolve independently.

**When not to use it:**

* When only one abstraction or implementation exists.
* When the abstraction and implementation will *always* change together.
* When the additional layer adds unnecessary complexity.

**Relation to other patterns:**

| Pattern              | Relationship                                                                                     |
| -------------------- | ------------------------------------------------------------------------------------------------ |
| **Adapter**          | Adapter makes existing classes compatible; Bridge designs them to be independent from the start. |
| **Strategy**         | Bridge focuses on structural decoupling; Strategy focuses on algorithm selection.                |
| **Abstract Factory** | Bridge can use factories to instantiate implementors.                                            |
| **Composite**        | Bridge can coexist with Composite for hierarchical abstractions.                                 |

**Cares & considerations:**

* Define clear roles for abstraction and implementor.
* Use interfaces and composition, not inheritance.
* Favor dependency injection to pass implementations.
* Document possible abstraction–implementation combinations.
* Don’t merge logic — each side should focus on its own responsibility.

**Common pitfalls:**

* Overengineering — using Bridge for a single abstraction.
* Mixing abstraction and implementation code in the same class.
* Creating circular dependencies between abstraction and implementor.
* Forgetting to keep the interface stable when implementations evolve.

---

## References and Bibliography

* Gamma, Erich et al. *Design Patterns: Elements of Reusable Object-Oriented Software*. Addison-Wesley, 1994.
* Refactoring.Guru — [Bridge Pattern](https://refactoring.guru/design-patterns/bridge)
* Microsoft Docs — *Bridge Design Pattern in C#.*
* Martin Fowler — *Patterns of Enterprise Application Architecture.*
* Oracle Java Docs — *Bridge Pattern Examples.*
* Real Python — *Bridge Pattern in Python.*

---

## Quick Summary / Mnemonic

**“Divide and connect.”**
Bridge separates *what you do* (abstraction) from *how you do it* (implementation).
You can change either side freely — like switching from local to cloud storage without rewriting your documents.

---
