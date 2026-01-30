---
description: Performance optimization guidelines for databases, caching, and algorithms
---

# Performance Rules

## Code Performance

### Database
- Use pagination for list queries
- Avoid N+1 queries (use JOIN FETCH or batch loading)
- Index frequently queried columns
- Use projections when full entity not needed

### Caching
- Cache frequently accessed, rarely changed data
- Set appropriate TTL
- Invalidate on writes
- Use cache-aside pattern

### Memory
- Avoid loading large collections into memory
- Use streaming for large datasets
- Close resources properly (try-with-resources)
- Be mindful of object creation in loops

## Algorithm Complexity

- Prefer O(1) or O(log n) when possible
- Document O(n²) or worse with justification
- Consider data structure choice (HashMap vs TreeMap)

## Lazy Loading

- Load data only when needed
- Use pagination for UI lists
- Implement infinite scroll over load-all

## Profiling Before Optimizing

- Measure before optimizing
- Identify actual bottlenecks
- Don't optimize prematurely
- Document performance improvements

### Profiling Tools

**Java:**
- VisualVM - Free, comes with JDK
- JProfiler - Commercial, comprehensive
- Async-profiler - Low overhead sampling profiler
- Built-in: `jstack`, `jmap`, `jstat`

**Node.js:**
- Clinic.js - Comprehensive performance suite
- 0x - Flamegraph generator
- Node.js built-in `--prof` flag
- Chrome DevTools profiler

**Frontend:**
- Chrome DevTools Performance tab
- Lighthouse (Core Web Vitals)
- WebPageTest
- React DevTools Profiler

## Frontend Performance

### Core Web Vitals

| Metric | Target | Description |
|--------|--------|-------------|
| **LCP** | < 2.5s | Largest Contentful Paint |
| **FID** | < 100ms | First Input Delay |
| **CLS** | < 0.1 | Cumulative Layout Shift |
| **FCP** | < 1.8s | First Contentful Paint |
| **TTFB** | < 600ms | Time to First Byte |

### Optimization Techniques

- **Images**: Use WebP, lazy loading, responsive images
- **JavaScript**: Code split, tree shake, defer non-critical
- **CSS**: Minimize, inline critical CSS
- **Caching**: Service workers, CDN, browser caching
- **Fonts**: Preload critical fonts, use `font-display: swap`

## Lazy Loading

- Load data only when needed
- Use pagination for UI lists
- Implement infinite scroll over load-all
- Lazy load images with loading="lazy"
- Lazy load components with React.lazy() or dynamic imports

### Code Splitting Examples

```typescript
// React lazy loading
const HeavyChart = lazy(() => import('./HeavyChart'));
const AdminPanel = lazy(() => import('./AdminPanel'));

// Dynamic import with condition
async function loadHeavyModule() {
  if (shouldLoadModule) {
    const module = await import('./heavyModule');
    return module.default;
  }
}
```
