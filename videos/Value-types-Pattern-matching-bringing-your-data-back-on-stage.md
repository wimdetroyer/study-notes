https://www.youtube.com/watch?v=tWonozjIE-s


# Part One: Introduction + Demo

- **OOP is used in Java**
    - Adding methods in interface breaks the "I" in SOLID.
    - Encapsulation is used to define a public API, hiding internals.
    - Traditional OOP approach: Interfaces are used together with polymorphism, where behavior is tied to the class.
    
- **Turning it around with DOP (Data-Oriented Programming)**
    - DOP is interesting because the compiler can assist more than in the traditional approach.
    - In DOP, data is just data and behavior is not tied to it.
    - To implement DOP, use these Java 21 features:
    
        - **Marker Interfaces** 
            - Used to categorize data types (e.g., `City`, `Department`, `Country` implement the `Populated` interface).
            - The interface is sealed, permitting only specific data types. This enables exhaustive switch expression matching.
        
        - **Records** 
            - Data is modeled as records, which are final but can implement marker interfaces.
        
        - **Switch Expression** 
            - Handle null values with `case null`.
            - Check data exhaustively by ensuring every type is covered.
            - The `yield` statement is used for controlling the flow within the switch, avoiding ambiguity with the `return` keyword.
            - Fine-tune control over branch execution using the `when` keyword.
        
        - **Pattern Matching for Switch** 
            - Efficiently identify data types without `instanceof`.
            - Deconstruct records with `Record Patterns` (JEP 440).
                - **Why use Record Patterns?** 
                    - Changing record components triggers a compiler error, enforcing immutability.
                    - Though verbose, `unnamed patterns` can reduce verbosity. Use `_` for unused variables.
                - Combine default branches by using unnamed patterns: `case Type1 _, Type2 _ -> "default";`
        
        - **Behavior** 
            - Modeled separately, often as static methods.

---

# Part Two: Timeline for Further Development

- **Caution with DOP** 
    - DOP isn’t always the solution. Data isn’t always more important than behavior.
    
- **Wadler's Expression of the Problem:**
    - Polymorphism excels at adding new subtypes but struggles with new operations.
    - Pattern matching excels at adding new operations but doesn’t support adding new subtypes.
    - Balancing both is challenging (even research languages struggle).
    
- **Java 21 and Beyond** 
    - Matchers and related features are incoming but not yet part of Java 21.

---

# Part Three: Design Patterns

- **Primitive Types vs Using Objects**
    - Using objects for types is more readable than just picking descriptive variable names.
        - **Performance Hit:** 
            - Using objects (pointer chasing) might impact performance, but readability outweighs performance losses, which are negligible.
        - **Example:**
            - Instead of `Map<String, Integer> popByCity = ...;`, use `Map<City, Population> popByCity = ...;`, where `City` and `Population` are records.

- **Project Valhalla** 
    - Aims to make abstractions like this cost-free.
    - **Value Class**: 
        - Part of the solution: value classes abandon identity (no memory address) for performance.
