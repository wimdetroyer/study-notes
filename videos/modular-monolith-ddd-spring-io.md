# Crafting a Self-Documenting Modular Monolith with DDD Principles @ Spring I/O

**Video URL:** https://www.youtube.com/watch?v=O3Cytc-sJSc

## Key Concepts

### Career Path Consideration
- **Question explored:** Enterprise architect path as a career?
- **Certification:** TOGAF certification

### jMolecules Framework

#### @UseCase Annotation
- **From:** jMolecules framework
- **Applied to:** Application layer
- **Purpose:** Self-documenting architecture

### Spring Modulith

#### ApplicationModuleListener
- **Pattern:** Eventual consistency
- **Implementation:** Event sent through a NEW transaction
- **Benefit:** Decoupling between modules

#### Event Handling Options
- **Complex use cases:** ApplicationModuleListener with eventual consistency
- **Simple use cases:** 'Vanilla' Spring application event listeners

## Architecture Patterns
- Self-documenting modular monolith
- Domain-Driven Design (DDD) principles
- Event-driven decoupling within monolith
- Transaction boundary management for consistency