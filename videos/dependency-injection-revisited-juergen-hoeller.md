# Dependency Injection Revisited by Juergen Hoeller @ Spring I/O 2025

**Video URL:** https://www.youtube.com/watch?v=AvZEoxH_wGo

## Key Concepts

### Java Typed Handles
- **What is a handle?** - Java 'typed handle' concept
- Compare with `ObjectProvider` in Spring
- ObjectProvider is like Spring but you decide when to initialize
- No proxy involved in ObjectProvider

### Programmatic Bean Definitions
- **BeanRegistrar** in Spring Framework 7.0
- Designed for more 'complex' use cases
- Not easily available in the declarative way
- **Advantage:** All Java language features are available:
  - For loops
  - Complex conditionals
  - Full programmatic control

## Notes
- ObjectProvider gives you more control over initialization timing
- BeanRegistrar provides programmatic alternative to annotations for complex scenarios