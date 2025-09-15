# The Simplest Way to Build Resilient Applications by Giselle van Dongen @ Spring I/O 2025

**Video URL:** https://www.youtube.com/watch?v=lYcEa8APBTQ

## Key Concepts

### Form Factor
- **Question explored:** What is a 'form factor'?

### Restate - Durable Execution Engine
- **Definition:** A server that sits between requests and backend code of microservices
- **Functionality:**
  - Stores snapshots of code execution across services and time
  - Uses stored information for retries when failures occur
  - Provides durability and resilience to distributed systems

## Architecture Pattern
- Intermediary layer for microservices
- Stateful execution tracking
- Automatic retry mechanisms based on execution state