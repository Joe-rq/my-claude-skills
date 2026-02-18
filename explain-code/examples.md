# Example Output

## Example 1: React useEffect Hook

**Analogy**: Think of `useEffect` like a "side-effect manager" in a factory. When a worker (component) finishes making a product (render), the side-effect manager steps in to handle cleanup and setup tasks—like sweeping the floor (cleanup) or preparing tools for the next batch (setup).

**Diagram**:
```
┌─────────────────────────────────────────┐
│         Component Render              │
│    (Worker makes the product)         │
└──────────────────┬──────────────────┘
                   │
                   ▼
         ┌─────────────────────┐
         │   useEffect Hook    │
         │   (Side-effect     │
         │    Manager)        │
         └────────┬────────────┘
                  │
    ┌─────────────┴─────────────┐
    │                           │
    ▼                           ▼
┌─────────┐              ┌──────────┐
│ Setup   │              │ Cleanup  │
│ (Prepare│              │ (Sweep   │
│  tools) │              │  floor)  │
└─────────┘              └──────────┘
```

**Walkthrough**:
1. Component first renders with initial state
2. After render completes, `useEffect` runs the setup function
3. When component unmounts or dependencies change, cleanup runs first, then setup again

**Gotcha**: Don't forget the dependency array! Omitting it causes infinite loops because the effect runs after every render.

## Example 2: Python Decorator

**Analogy**: A decorator is like a gift-wrapping station. You hand over your gift (function), the station wraps it in fancy paper (extra behavior), and returns the wrapped version. The gift inside is unchanged—but now it comes with a bow on top.

**Diagram**:
```
         Original Function
               │
               ▼
   ┌───────────────────────┐
   │     Decorator          │
   │  ┌─────────────────┐  │
   │  │  Before logic    │  │  ← e.g. logging, auth check
   │  ├─────────────────┤  │
   │  │  original_func() │  │  ← your actual function runs
   │  ├─────────────────┤  │
   │  │  After logic     │  │  ← e.g. timing, cleanup
   │  └─────────────────┘  │
   └───────────────────────┘
               │
               ▼
        Wrapped Function
      (same name, extra powers)
```

**Code**:
```python
def timer(func):
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"{func.__name__} took {time.time() - start:.2f}s")
        return result
    return wrapper

@timer
def fetch_data():
    ...
```

**Walkthrough**:
1. `@timer` is syntactic sugar for `fetch_data = timer(fetch_data)`
2. `timer` receives the original `fetch_data` as `func`
3. It defines `wrapper`, which records start time, calls the real function, then prints elapsed time
4. `timer` returns `wrapper`—so now `fetch_data` actually points to `wrapper`
5. Every time you call `fetch_data()`, you're really calling `wrapper()`, which calls the original inside

**Gotcha**: The wrapped function loses its original `__name__` and `__doc__`. Use `@functools.wraps(func)` on the wrapper to preserve them.

## Example 3: JavaScript async/await

**Analogy**: Imagine ordering food at a restaurant. You place your order (start an async operation) and then read your phone (continue execution) instead of standing at the kitchen door. When the waiter says "your dish is ready" (Promise resolves), you pick it up and eat. `await` is you choosing to pause and wait at the counter instead.

**Diagram**:
```
Main Thread          Promise (Kitchen)
    │
    ├─ fetch(url)  ──────→  🍳 Cooking...
    │                              │
    ├─ await ·····················→│
    │  (paused)                    │
    │                              ▼
    │                        ✅ Response ready
    │←─────────────────────── data returned
    │
    ├─ process(data)
    ▼
```

**Code**:
```javascript
async function getUser(id) {
  const response = await fetch(`/api/users/${id}`);
  const user = await response.json();
  return user;
}
```

**Walkthrough**:
1. `async` marks the function as returning a Promise
2. `fetch()` starts a network request and immediately returns a Promise (food is being cooked)
3. `await` pauses this function until the Promise resolves—but does NOT block the main thread; other code can run
4. When the response arrives, execution resumes and `response` holds the result
5. `response.json()` is also async (parsing the body), so we `await` again
6. The function returns `user`, which the caller receives as a Promise

**Gotcha**: `await` only pauses inside the `async` function—it doesn't freeze the whole program. But if you `await` in a loop sequentially, you lose parallelism. Use `Promise.all()` when requests are independent.
