# Module 07: Flows and Reactive Programming

## Overview

Kotlin Flows provide a reactive programming approach for handling streams of data. This module covers the Observer
pattern, different types of flows, and how to work with hot and cold flows.

## Learning Objectives

- Understand the Observer pattern and reactive programming
- Learn the difference between hot and cold flows
- Master Flow, SharedFlow, StateFlow, and Channel
- Choose the right flow type for your use case

### Reactive Programming and Observer Pattern

----

Reactive programming is a declarative programming paradigm concerned with data streams and the propagation of change.

- **Observer Pattern**: A provider (Subject) maintains a list of dependents (Observers) and notifies them automatically
  of any state changes.
- **Benefits**:
    - Decouples data producers from consumers.
    - Handles asynchronous data streams gracefully.
    - Makes UI updates automatic when data changes.

### Hot vs Cold Flows

----

Understanding the temperature of a stream is crucial for choosing the right tool.

- **Cold Flows**: Only start producing data when someone starts collecting them. Each collector gets its own independent
  stream of data. Think of it like a **YouTube video** (starts from the beginning for each person).
    - Example: `flow { ... }`
- **Hot Flows**: Produce data even if no one is listening. Data is shared among all collectors. Think of it like a *
  *Radio station** (everyone hears the same thing at the same time).
    - Examples: `StateFlow`, `SharedFlow`, `Channel`.

### Flow (Cold)

----

The basic `Flow` is cold. It's used for one-off requests or streams that should be re-executed for every consumer.

```kotlin
fun getUsers(): Flow<User> = flow {
    emit(User("Alice"))
    delay(1000)
    emit(User("Bob"))
}
```

- **Collecting**: Use `collect` to start receiving values.
- **Flow Context**: Flows are "context-preserving." They run in the context of the collector. To change the producer's
  context, use the `flowOn` operator. e.g

```kotlin
getUsers()
    .flowOn(Dispatchers.IO) // Runs calls from here upward in the IO dispatcher
    .collect { user -> println(user) }
```

### SharedFlow (Hot)

----

A `SharedFlow` is a hot flow that can have multiple collectors. It's great for sending "events" to multiple parts of the
app.

- **Replay**: Can store a number of previous values for new subscribers.
- **Buffer**: Can handle slow subscribers by buffering values.

```kotlin
val events = MutableSharedFlow<String>(replay = 1)
events.emit("Event 1")
```

### StateFlow (Hot)

----

`StateFlow` is a specialized `SharedFlow` designed specifically for **State Management**.

- **Initial Value**: Always has a starting value.
- **Conflation**: Only emits the *latest* value. If the state changes to the same value, it won't emit again.
- **Usage**: Perfect for ViewModels to expose UI state.

```kotlin
private val _uiState = MutableStateFlow(UserUiState())
val uiState: StateFlow<UserUiState> = _uiState.asStateFlow()
```

### 6. Channel

----

`Channel` is a way to communicate between coroutines. Unlike flows, which are about "broadcasting" data, channels are
more about "transferring" data (usually one-to-one). Meaning, if we have two consumers but only one emission from the
producer, only one of the consumers will receive it. Likewise, if we have two events and two consumers, only one of the
consumers will receive each event, as opposed to `SharedFlow` where both consumers would receive both events.

- **Types**: `Rendezvous` (default), `Buffered`, `Conflated`, `Unlimited`.
- **Send/Receive**: Producers `send` and consumers `receive`.

### 7. Flow Operators

----

Operators allow you to transform and manipulate streams. A few common ones:

- **Transformation**: `map`, `transform`, `scan`.
- **Filtering**: `filter`, `take`, `drop`.
- **Combination**: `zip`, `combine`, `flattenMerge`.
- **Terminal**: `collect`, `first`, `toList`, `reduce`.

```kotlin
getUsers()
    .filter { it.age > 18 } // From here down, we're only working with users older than 18
    .map { it.name } // Transform User to String (name)
    .collect { name -> println(name) } // Collect the names (String) and print them
```

### 8. Best Practices

----

- **Choose Wisely**:
    - Use `Flow` for one-off data streams and data that should be re-executed for each consumer.
    - Use `StateFlow` for state management where you want to always have the latest value available.
    - Use `SharedFlow` for event broadcasting to multiple consumers, especially when you want to replay the latest value
      to new subscribers.
    - Use `Channel` for one-to-one communication between coroutines, especially when you want to transfer data without
      the need for replaying or sharing.
- **Lifecycle**: On Android, use `collectAsStateWithLifecycle` to collect flows in a lifecycle-aware manner, meaning
  collection will automatically start and stop based on the lifecycle state of the component (e.g., Activity, Fragment).
- **Error Handling**: Use the `catch` operator to handle exceptions in the stream without breaking the whole coroutine.

## Resources

- [Kotlin Documentation: Flow](https://kotlinlang.org/docs/flow.html)
- [Android Developers: Kotlin flows on Android](https://developer.android.com/kotlin/flow)

## Next Steps

Continue to [Module 08: Compose Fundamentals](../08-compose-fundamentals/README.md)
