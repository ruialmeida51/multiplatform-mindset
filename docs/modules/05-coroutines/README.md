# Module 05: Coroutines Fundamentals

## Overview

Kotlin Coroutines provide a powerful way to write asynchronous code that looks sequential. This module covers the
fundamentals of coroutines, including suspension, cancellation, and execution contexts.

## Learning Objectives

- Understand what coroutines are and why they're useful
- Learn how coroutines can be paused (suspended) and cancelled
- Master coroutine scopes and dispatchers
- Work with coroutine contexts and jobs

### What are Coroutines?

----

Coroutines are "lightweight threads" that allow you to write asynchronous, non-blocking code in a sequential style. They
solve the problem of long-running tasks (like network calls or database operations) freezing the UI or wasting system
resources.

- **Asynchronous Programming**: Instead of waiting for a task to finish, the program can continue doing other work.
- **Coroutines vs Threads**:
    - **Threads** are managed by the OS, are expensive to create, and have a significant memory footprint.
    - **Coroutines** are managed by the Kotlin Coroutines library, are very cheap (you can run 100,000+ of them on a
      single thread), and don't block the underlying thread when suspended.
- **Benefits**:
    - Better readability (no "callback hell").
    - Efficient resource usage.
    - Built-in support for cancellation.

### Suspension

----

Suspension is the "superpower" of coroutines. When a coroutine calls a `suspend` function, it "pauses" its execution
without blocking the thread it's running on.

- **Suspend Functions**: Marked with the `suspend` keyword. They can only be called from other suspend functions or a
  coroutine.
- **How Suspension Works**: The coroutine saves its state, releases the thread, and when the result is ready, it
  resumes from where it left off (possibly on a different thread).
- **Suspend vs Blocking**:
    - **Blocking**: The thread stays "busy" doing nothing while waiting. No other code can run on it.
    - **Suspending**: The thread is "freed" to do other work while waiting.

```kotlin
// ✅ Suspending
suspend fun fetchUser(id: String): User {
    delay(1000) // Does not block the thread
    return User(id, "Alice")
}

// ❌ Blocking
fun fetchUserBlocking(id: String): User {
    Thread.sleep(1000) // Blocks the thread!
    return User(id, "Alice")
}
```

### Cancellation

----

Coroutines are **cooperative**, meaning they don't just "die" instantly when cancelled; they must check for
cancellation regularly.

- **Cooperative Cancellation**: Functions like `delay()` and `yield()` are "cancellable." If you have a long-running
  loop, you should check `isActive`.
- **Cancellation Exceptions**: When a coroutine is cancelled, it throws a `CancellationException`. This is a normal part
  of the lifecycle and shouldn't be caught to "prevent" cancellation.
- **Best Practices**:
    - Always use cancellable suspend functions for async work.
    - Check for cancellation in heavy CPU-bound loops using `ensureActive()` or `yield()`.

```kotlin
val job = scope.launch {
    try {
        while (isActive) {
            // Do work...
        }
    } finally {
        // Clean up resources even if cancelled
        withContext(NonCancellable) {
            cleanup()
        }
    }
}
job.cancel() // Request cancellation
```

### Execution Threads and Dispatchers

----

Dispatchers determine which thread or thread pool a coroutine runs on.

- **Dispatchers.Main**: UI thread (Android, iOS Main Queue). Use for updating UI.
- **Dispatchers.IO**: Optimized for I/O tasks (Network, Disk). Large thread pool.
- **Dispatchers.Default**: Optimized for CPU-intensive tasks (Sorting, Image processing). Thread pool size = Number of
  CPU cores.
- **Dispatchers.Unconfined**: Runs in whatever the current caller's thread is, but only until the first suspension
  point. Use with caution! Generally not used in production code apart from very specific cases (like testing or certain
  libraries).

**Context Switching**: You can change the dispatcher within a coroutine using `withContext`.

```kotlin
suspend fun loadAndShowUser() {
    val user = withContext(Dispatchers.IO) {
        userRepository.getUser() // Fetch on IO thread
    }
    // Automatically resumes on the original dispatcher (e.g., Main)
    showUser(user)
}
```

### Coroutine Scope and Structured Concurrency

----

A `CoroutineScope` keeps track of all coroutines launched within it.

- **Structured Concurrency**: A parent scope won't finish until all its children finish. If a scope is cancelled, all
  its children are cancelled. This prevents "leaking" coroutines.
- **Creating Scopes**:
    - `viewModelScope`: (Android) Automatically cancelled when ViewModel is cleared.
    - `lifecycleScope`: (Android) Tied to Activity/Fragment lifecycle.
    - `CoroutineScope(context)`: Manual scope creation.

### Jobs

----

A `Job` is a handle to a coroutine. You get it when you call `launch`.

- **Lifecycle**: `New` -> `Active` -> `Completing` -> `Completed`.
- **Job Hierarchy**: Coroutines have a parent-child relationship.
- **Waiting**: You can use `job.join()` to wait for a coroutine to finish or other operators to manipulate the coroutine
  lifecycle  (e.g., `cancel()`, `isActive`).

### CoroutineContext

----

A `CoroutineContext` is a set of elements that define the behavior of a coroutine.

- **Elements**: `Job`, `Dispatcher`, `CoroutineName`, `CoroutineExceptionHandler`.
- **Combining Contexts**: You can use the `+` operator.
  `val context = Dispatchers.IO + Job() + CoroutineName("MyCoroutine")`
- **Inheritance**: A child coroutine inherits the context of its parent, but can override specific elements.

## Resources

- [Kotlin Documentation: Coroutines Guide](https://kotlinlang.org/docs/coroutines-guide.html)
- [Android Developers: Coroutines on Android](https://developer.android.com/kotlin/coroutines)
- [Kotlin Conf: Coroutines by Roman Elizarov](https://www.youtube.com/watch?v=_hfBv0a09Jc)

## Next Steps

Continue to [Module 06: Jobs and Supervisor Patterns](../06-jobs-supervisor/README.md)
