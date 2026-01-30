---
name: performance
description: Performance optimization guidelines for databases, caching, memory, and algorithms. Apply when optimizing code or reviewing performance.
mode: model_decision
---

# Performance Rules

## Database Performance

- Use pagination for list queries (default: 20 items)
- Avoid N+1 queries (use JOIN FETCH or batch loading)
- Index frequently queried columns
- Use projections when full entity not needed

## Caching Strategy

- Cache frequently accessed, rarely changed data
- Set appropriate TTL (Time To Live)
- Invalidate cache on writes
- Use cache-aside pattern

## Memory Management

- Avoid loading large collections into memory
- Use streaming for large datasets
- Close resources properly (try-with-resources)
- Be mindful of object creation in loops

## Algorithm Complexity

- Prefer O(1) or O(log n) when possible
- Document O(n²) or worse with justification
- Consider data structure choice (HashMap vs TreeMap)

## Frontend Performance (Core Web Vitals)

| Metric | Target | Description |
|--------|--------|-------------|
| **LCP** | < 2.5s | Largest Contentful Paint |
| **FID** | < 100ms | First Input Delay |
| **CLS** | < 0.1 | Cumulative Layout Shift |
| **FCP** | < 1.8s | First Contentful Paint |

## Profiling Before Optimizing

- Measure before optimizing
- Identify actual bottlenecks
- Don't optimize prematurely

**Profiling Tools:**
- Java: VisualVM, JProfiler, Async-profiler
- Node.js: Clinic.js, Chrome DevTools
- Frontend: Lighthouse, Chrome DevTools Performance tab

## Lazy Loading

- Load data only when needed
- Use pagination for UI lists
- Implement infinite scroll over load-all
- Lazy load images with loading="lazy"
- Lazy load components with dynamic imports
