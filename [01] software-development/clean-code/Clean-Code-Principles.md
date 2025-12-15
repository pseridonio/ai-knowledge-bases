# Knowledge Base: **Clean Code** in C# (.NET)

> **Objective:** This knowledge base provides definitions, principles, and benefits of *Clean Code* so that a generative AI can align its code generation and responses with these concepts.

---

## 1. Context for the AI

> **Initialization Prompt for the Model:**  
> “You are an expert C#/.NET (and polyglot) software engineering assistant. Whenever you generate or refactor code, apply the Clean Code principles defined in this knowledge base.”

---

## 2. What Is *Clean Code*?

1. **Readability First**  
   - Code should “read like a book”: class, method, and variable names must be self‑explanatory, avoiding superfluous comments.  
2. **Simplicity and Clarity**  
   - Programs should consist of single‑responsibility functions and classes that are small and cohesive.  
3. **Expressiveness**  
   - The code’s structure and language constructs should directly reflect the developer’s intentions without clever or obscure patterns.  
4. **Continuous Discipline**  
   - Boy Scout Rule: when you modify code, always leave it “a little bit better” than you found it.

---

## 3. Benefits of Applying *Clean Code*

| Benefit                            | Details                                                                 |
|------------------------------------|-------------------------------------------------------------------------|
| **Ease of Maintenance**            | Less time spent understanding, locating, and fixing bugs.               |
| **Long‑Term Cost Reduction**       | Clean code prevents costly refactors and rework down the line.          |
| **Higher Team Productivity**       | Faster onboarding and fewer hours spent debugging obscure code.         |
| **Reduced Error Proneness**        | Cohesion and simplicity expose logical inconsistencies early.           |
| **Easier Evolution & Refactoring** | Modularity makes safe refactors (e.g., method extraction) more reliable.|

---

## 4. Key References

- **Robert C. Martin (“Uncle Bob”)**  
  - *Clean Code: A Handbook of Agile Software Craftsmanship* (2008)  
- **Steve McConnell**  
  - *Code Complete* (1993)  
- **Martin Fowler**  
  - *Refactoring: Improving the Design of Existing Code* (1999)  
- **Jason Taylor**  
  - *Clean Architecture with .NET Core* (templates and articles)  
- **Mohammed (Moh_Moh701)**  
  - *C# Clean Code: SOLID Principles* (practical examples)  
- **Ziggy Rafiq**  
  - *Clean Code and Best Practices with C# 13* (modern C# features)  
- **Steve Smith**  
  - *Writing Clean Code in ASP.NET Core with Dependency Injection*  

---

## 5. How to Use This Base for Code Generation

1. **Before generating any code**, the assistant should load this document into memory and apply its principles.  
2. **Naming Validation**  
   - Every identifier must follow the “Good Name Rule”: clear and without abbreviations (e.g., `CalculateInvoiceTotal()` instead of `CalcInv()`).  
3. **Function and Class Size**  
   - If a method exceeds ~20–25 lines, suggest refactoring into smaller single‑responsibility methods.  
4. **Comments**  
   - Use comments only to explain “why,” not “what” (the code itself should be expressive enough).  
5. **Design Review**  
   - Apply SOLID and look for “code smells” (overly long methods, giant classes, duplicated logic).

---

## 6. **Sample Prompts for the AI**

- **New Feature Generation**  
  > “Apply Clean Code: use clear names, keep methods short, and enforce SOLID when implementing this data‑reading service.”  
- **Refactoring**  
  > “Refactor this method to adhere to Clean Code principles: extract responsibilities into smaller methods and rename variables for clarity.”  
- **Pull Request Review**  
  > “Analyze this diff and highlight any Clean Code violations: code smells, poor naming, or methods with dual responsibilities.”

---

> **Final Note:**  
> This base is the master guide for any code generation, refactoring, or review in C#/.NET. Keep it always accessible and update it as new standards and best practices emerge.  

--- 

## Sample codes of what not to do and what to do

### What This Code Does (in Natural Language)

Calculates the total of a list of product prices, including a 10% tax rate. It takes an array of prices, applies the tax to each item, and sums the results to return the final total.

---

### “Bad” Example (Violates Clean Code Principles)

#### C# Example

```csharp
public class p  
{  
    // Calculate something with tax
    public double c(double[] a)  
    {  
        double r = 0;  
        for (int i = 0; i < a.Length; i++)  
        {  
            // 1.1 é imposto
            r += a[i] * 1.1;  
        }  
        return r;  
    }  
}
```

#### Java 21 Example

```java
public class P {
    // Calculate something with tax
    public double c(double[] a) {
        double r = 0;
        for (int i = 0; i < a.length; i++) {
            // 1.1 is tax
            r += a[i] * 1.1;
        }
        return r;
    }
}
```

#### Python 13 Example

```python
class P:
    # Calculate something with tax
    def c(self, a):
        r = 0
        for price in a:
            # 1.1 is tax
            r += price * 1.1
        return r
```

**Problems with This Code**

1. **Meaningless Names**
   * Class `p`, method `c`, parameter `a`, and variable `r` convey no intent.  

2. **Redundant Comments**
   * Comments describe “what” is happening rather than “why.”  

3. **Magic Number**
   * The literal `1.1` appears without context.  
   
4. **Mixed Responsibilities**
   * Iteration and tax calculation are tangled in one method.

---

### “Good” Example (Adheres to Clean Code Principles)

#### C# Example
```csharp
public class InvoiceCalculator
{
    private const double TaxRate = 0.10;

    /// <summary>
    /// Calculates the total of all product prices, including tax.
    /// </summary>
    /// <param name="productPrices">Array of unit prices for products.</param>
    /// <returns>Total amount with tax applied.</returns>
    public double CalculateTotalWithTax(double[] productPrices)
    {
        double total = 0.0;

        foreach (double price in productPrices)
        {
            total += CalculatePriceWithTax(price);
        }

        return total;
    }

    /// <summary>
    /// Applies the tax rate to a single unit price.
    /// </summary>
    /// <param name="price">Price before tax.</param>
    /// <returns>Price after tax.</returns>
    private double CalculatePriceWithTax(double price)
    {
        return price + (price * TaxRate);
    }
}
```

#### Java 21 Example
```java
public class InvoiceCalculator {
    private static final double TAX_RATE = 0.10;

    /**
     * Calculates the total of all product prices, including tax.
     * @param productPrices Array of unit prices for products.
     * @return Total amount with tax applied.
     */
    public double calculateTotalWithTax(double[] productPrices) {
        double total = 0.0;
        for (double price : productPrices) {
            total += calculatePriceWithTax(price);
        }
        return total;
    }

    /**
     * Applies the tax rate to a single unit price.
     * @param price Price before tax.
     * @return Price after tax.
     */
    private double calculatePriceWithTax(double price) {
        return price + (price * TAX_RATE);
    }
}
```

#### Python 13 Example
```python
class InvoiceCalculator:
    TAX_RATE = 0.10

    def calculate_total_with_tax(self, product_prices):
        """
        Calculates the total of all product prices, including tax.

        Args:
            product_prices (list[float]): List of unit prices for products.

        Returns:
            float: Total amount with tax applied.
        """
        total = 0.0
        for price in product_prices:
            total += self.calculate_price_with_tax(price)
        return total

    def calculate_price_with_tax(self, price):
        """
        Applies the tax rate to a single unit price.

        Args:
            price (float): Price before tax.

        Returns:
            float: Price after tax.
        """
        return price + (price * self.TAX_RATE)
```

---

### Why This Code Is “Clean”

1. **Descriptive Names**

   * `InvoiceCalculator` clearly states the class’s purpose.
   * Method names like `CalculateTotalWithTax` and `CalculatePriceWithTax` are self‑explanatory.
   * Parameter and variable names (`productPrices`, `price`, `total`, `TaxRate`) convey their roles.
2. **Single Responsibility**

   * One method handles iteration and accumulation; another handles tax calculation.
3. **No Magic Numbers**

   * The constant `TaxRate` documents the 10% rate.
4. **Self‑Documenting Code**

   * XML comments provide high‑level intent (“why”), not details of “what” the code already shows.
5. **Appropriate Method Size**

   * Each method is concise and focused on one task (<20 lines).

---

> **Tip for a Generative AI**
>
> * Always define constants for fixed values (tax rates, limits, etc.).
> * Separate iteration logic from calculation logic.
> * Use clear, descriptive names instead of obscure abbreviations.
> * Comment only when the “why” isn’t obvious from the code itself.
