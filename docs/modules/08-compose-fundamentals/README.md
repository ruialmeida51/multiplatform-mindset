🚧 IN CONSTRUCTION & REVIEW

---

# Module 08: Compose Fundamentals 

---

🚧 IN CONSTRUCTION & REVIEW

## Overview
Jetpack Compose and Compose Multiplatform represent a paradigm shift in UI development. This comprehensive module covers declarative UI, composition, recomposition, and best practices for building efficient Compose UIs.

## Learning Objectives
- Understand declarative vs imperative UI programming
- Master composition and recomposition
- Learn about stability and skipping mechanics
- Manage state effectively in Compose
- Understand lifecycles and side effects
- Optimize performance and avoid unnecessary recompositions

## Topics Covered

### 1. Declarative vs Imperative UI

----

- **Imperative UI** (e.g., XML Views): You manually manage the state of the UI. If a user logs in, you find the TextView
  and call `setText("Hello")`. You must carefully keep the UI in sync with the data.
- **Declarative UI** (e.g., Compose): You describe *what* the UI should look like for a given state. When the state
  changes, the framework automatically rebuilds (recomposes) the UI parts that changed.

**Benefits**:
- Less code and fewer bugs (state is the single source of truth).
- More intuitive UI logic.
- Easier to build complex, dynamic layouts.

### 2. Composition and Recomposition

----

- **Composition**: The process of building the UI tree by calling Composable functions.
- **Recomposition**: When the data used by a Composable changes, Compose re-runs that function to update the UI. Compose
  is "smart" — it only recomposes the parts of the UI that actually changed.

```kotlin
@Composable
fun UserGreeting(name: String) {
    Text("Hello $name!") // This part recomposes only if 'name' changes
}
```

### 3. Stability and Skipping

----

Compose optimizes performance using **Stability**.

- **Stable Types**: Types whose properties are immutable or whose changes Compose can track (e.g., `String`, `Int`, `@Immutable` data classes).
- **Skipping**: If a Composable's parameters haven't changed and they are all *Stable*, Compose will **skip** that Composable during recomposition.
- **Impact**: Unstable types (like `List` or mutable classes) can cause unnecessary recompositions because Compose
  can't be sure if they changed.

### 4. State Management

----

State is any value that can change over time and affect the UI.

- **`remember`**: Stores a value in the Composition. It's preserved during recomposition but lost during configuration
  changes (like rotation).
- **`rememberSaveable`**: Similar to `remember`, but survives configuration changes.
- **State Hoisting**: Moving state up to a caller to make a Composable stateless and reusable.

```kotlin
@Composable
fun SearchBar(query: String, onQueryChange: (String) -> Unit) { // Hoisted State
    TextField(value = query, onValueChange = onQueryChange)
}
```

### 5. Lifecycles and Side Effects

----

Composables have their own lifecycle: **Enter Composition -> Recompose (0+ times) -> Leave Composition**.

**Side Effects** are actions that happen outside the scope of a Composable (like showing a Toast or starting a timer).

- **`LaunchedEffect`**: Runs a suspend function when a key changes.
- **`rememberCoroutineScope`**: Gives you a scope to launch coroutines in response to user events (e.g., button clicks).
- **`DisposableEffect`**: Used for effects that need cleanup (e.g., registering/unregistering a listener).
- **`derivedStateOf`**: Used when you want to calculate a state based on other states (optimization).

### 6. Flow Collection in Compose

----

- **`collectAsStateWithLifecycle()`**: (Recommended) Collects a Flow and converts it to State, while automatically
  stopping collection when the app is in the background.

```kotlin
@Composable
fun UserProfile(viewModel: UserViewModel) {
    val state by viewModel.uiState.collectAsStateWithLifecycle()
    Text(state.name)
}
```

### 7. CompositionLocal

----

`CompositionLocal` is a way to pass data through the Composition tree implicitly, without passing it as a parameter to
every Composable.

- **Use Cases**: Themes, User details, Analytics handlers.
- **Providers**: `CompositionLocalProvider(LocalUser provides user) { ... }`

### 8. Theming

----

Compose uses a system of `MaterialTheme` by default, allowing you to define `Color`, `Typography`, and `Shape` for your
entire app.

```kotlin
MaterialTheme(
    colorScheme = myColors,
    typography = myTypography
) {
    // App Content
}
```

## Resources

- [Official Jetpack Compose Documentation](https://developer.android.com/jetpack/compose)
- [Compose Multiplatform Documentation](https://www.jetbrains.com/lp/compose-multiplatform/)
- [Jetpack Compose Pathway](https://developer.android.com/courses/pathways/jetpack-compose-fundamentals)
- [Codelab: State in Jetpack Compose](https://developer.android.com/codelabs/jetpack-compose-state)

## Next Steps
Continue to [Module 09: expect/actual for KMP/CMP](../09-expect-actual/README.md)
