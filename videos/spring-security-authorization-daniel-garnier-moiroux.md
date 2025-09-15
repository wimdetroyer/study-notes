# Authorization in Spring Security: permissions, roles and beyond by Daniel Garnier-Moiroux @ Spring I/O

**Video URL:** https://www.youtube.com/watch?v=-x8-s3QnhMQ

## Core Concepts

### Profile Page Security
- **Use Case:** Profile page accessible only if principal equals username in URL path
- **Implementation:** Request matching with variables

### AuthorizationDecision
- **Definition:** Simple wrapper around a boolean
- **Purpose:** Decides if an Authenticated user has access to a resource

### Authorization Context
- **User-based:** Authorization by the Authentication (the user)
- **Context-based:** Authorization from the context (where did the request originate from?)

## Spring Security 6.3+ Features

### Meta Annotations for Business Rules
- **Since:** Spring Security 6.3
- **Feature:** Write reusable meta annotations comprising business rules
- **Example:** `@AllowableDomains(domains = {"foo.com", "bar.com"})`
- **Requirements:**
  - Annotation parameters can be passed
  - Spring Security bean needed: `AnnotationTemplateExpressionDefaults`

### Collection Filtering
- **@PreFilter / @PostFilter:** For collections
- **Compare with:** @PreAuthorize, @PostAuthorize (for single objects)

### Field-Level Security
- **Combination:**
  - Security annotation like `@PreAuthorize` on object fields
  - `@AuthorizeReturnObject` on method returning said object
- **Result:** Granular field-level masking

### Error Handling
- **AuthorizationDeniedHandler:** Beautiful handling of `AuthorizationDeniedException`
- **Replaces:** Blank page with exception stacktrace

### Authentication Details
- **Purpose:** Store additional details during login flow
- **Examples:**
  - Which authentication flow was used (form login, HTTP basic)
  - Available alongside Principal, Authorities, etc.
- **Usage:** Enhanced context for authorization decisions