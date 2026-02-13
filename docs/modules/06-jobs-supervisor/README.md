# Module 06: Jobs and Supervisor Patterns

## Overview
This module dives deeper into Jobs, exploring how to manage coroutine lifecycles and handle failures gracefully with SupervisorJob and supervisorScope.

## Learning Objectives
- Understand Job in detail and its role in coroutines
- Learn about exception handling in coroutines
- Master SupervisorJob and supervisorScope
- Handle child coroutine failures effectively

### Job Deep Dive

----

A `Job` is the fundamental unit of coroutine management. It defines the lifecycle and relationship of coroutines.

- **Job States**:
    - **New**: Coroutine created but not yet started (e.g., using `CoroutineStart.LAZY`).
    - **Active**: Coroutine is running or suspended.
    - **Completing**: Coroutine finished its work but is waiting for its children to finish.
    - **Completed**: Coroutine and all its children have finished successfully.
    - **Cancelling**: `cancel()` was called, but cleanup is still happening.
    - **Cancelled**: Coroutine was cancelled or failed with an exception.

- **Parent-Child Relationships**: By default, coroutines are strictly hierarchical.
    - If a parent is cancelled, all children are cancelled.
    - If a child fails (with an exception other than `CancellationException`), the parent is cancelled.

### Exception Handling in Coroutines

----

Exception handling in coroutines is different from regular code because of structured concurrency.

- **Exception Propagation**: When a coroutine fails, it cancels its parent, which then cancels all its other children. This is "fail-fast" behavior.
- **CoroutineExceptionHandler**: A context element used to catch *uncaught* exceptions in a scope. It's only useful when placed in the root of a scope (like `launch`). It won't work inside `async` (where exceptions are encapsulated in the `Deferred` result).

```kotlin
val handler = CoroutineExceptionHandler { _, exception ->
    println("Caught $exception")
}

scope.launch(handler) {
    throw RuntimeException("Boom!")
}
```

### SupervisorJob

----

Sometimes, you don't want a single child's failure to bring down the whole system. This is where `SupervisorJob`
comes in.

- **What is SupervisorJob?**: A special type of `Job` where a child's failure does not affect the parent or other
  siblings.
- **When to Use**:
    - A screen with multiple independent UI components (e.g., a dashboard). If one component's data fetch fails, the
      rest should still work.
    - Background services handling multiple independent tasks.

```kotlin
val supervisor = SupervisorJob()
val scope = CoroutineScope(Dispatchers.Main + supervisor)

scope.launch {
    // If this fails, the scope and other coroutines remain active
    throw Exception("Child 1 failed")
}
scope.launch {
    // This will still run!
    println("Child 2 is still alive")
}
```

### supervisorScope

----

`supervisorScope` is the scoped version of `SupervisorJob`. It creates a sub-scope where children can fail
independently.

- **Scope vs Job**: Use `supervisorScope` when you want to wrap a specific block of work in a supervisor behavior
  within a regular coroutine.
- **Best Practice**: Always handle exceptions *inside* the children of a `supervisorScope` (using `try-catch`), or
  the exception will still propagate to the caller of `supervisorScope`.

```kotlin
suspend fun loadDashboard() = supervisorScope {
    launch { fetchWeather() } // If this fails...
    launch { fetchNews() }    // ...this still runs
}
```

## Resources

- [Kotlin Documentation: Exception Handling](https://kotlinlang.org/docs/exception-handling.html)
- [Android Developers: Coroutines - Patterns for work that shouldn't be cancelled](https://medium.com/androiddevelopers/coroutines-patterns-for-work-that-shouldnt-be-cancelled-e26c40f142ad)
- [Article: SupervisorJob vs Job](https://proandroiddev.com/kotlin-coroutines-job-vs-supervisorjob-3269b8288da6)

## Next Steps
Continue to [Module 07: Flows and Reactive Programming](../07-flows/README.md)
