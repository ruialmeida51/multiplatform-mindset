# Module 03: MVVM and Unidirectional Data Flow

## Overview

This module focuses on the Model-View-ViewModel (MVVM) architectural pattern, which is the standard approach for this
guide. We'll also explore unidirectional data flow and briefly reference other patterns like MVI.

## Learning Objectives

- Understand the MVVM architectural pattern
- Learn about unidirectional data flow (UDF)
- Separate concerns between View, ViewModel, and Model layers
- Explore alternatives like MVI (Model-View-Intent)

### MVVM Architecture (Model-View-ViewModel)

---

MVVM is a design pattern that separates the user interface (UI) from the business logic and data. It was invented by
Microsoft but adopted widely by Google for Android development.

#### The Layers

1. **Model**:
    * The source of truth.
    * Holds data and business logic (e.g., Repositories, Database, Network).
    * *Completely ignorant of the UI.*

2. **View** (The UI):
    * The Activity, Fragment, or Composable.
    * Displays data observed from the ViewModel.
    * Forwards user interactions (clicks, text input) to the ViewModel.
    * *Does not contain business logic.*

3. **ViewModel**:
    * Acts as the bridge between View and Model.
    * Holds the **UiState**.
    * Exposes streams of data (StateFlow/LiveData) for the View to observe.
    * Survives configuration changes (like screen rotation on Android).

#### Diagram

<img src="/images/udf_diagram.png"/>

#### Code Example

```kotlin
// 1. Model
class UserRepositoryImplementation : UserRepository {
    override fun getUser(): User {
        // Imagine this fetches from a database or network
        return User(name = "Alice")
    }
}

interface UserRepository {
    fun getUser(): User
}

class GetUserUseCase(private val userRepository: UserRepository) {
    fun getUser(): User = userRepository.getUser()
}

// 2. ViewModel
class UserViewModel(private val getUser: GetUserUseCase) : ViewModel() {

    // Internal mutable state
    private val _uiState = MutableStateFlow(UserUiState())

    // Public immutable state exposed to View
    val uiState: StateFlow<UserUiState> = _uiState.asStateFlow()

    fun loadUser() {
        val user = getUser()
        _uiState.update { it.copy(name = user.name, isLoading = false) }
    }
}

// 3. View (Conceptual - e.g., Jetpack Compose)
@Composable
fun UserScreen(viewModel: UserViewModel) {
    // Observer state
    val state by viewModel.uiState.collectAsStateWithLifecycle()

    if (state.isLoading) {
        CircularProgressIndicator()
    } else {
        Text(text = "Hello, ${state.name}")
    }

    // Send event
    Button(onClick = { viewModel.loadUser() }) {
        Text("Refresh")
    }
}
```

### Unidirectional Data Flow (UDF)

---

UDF is a design pattern where data flows in **only one direction**.

#### The Cycle

1. **State** flows down from ViewModel to View.
2. **Events** flow up from View to ViewModel (user clicks, inputs).
3. ViewModel handles the event, updates the data, and emits a **New State**.

**State -> View -> Event -> ViewModel -> New State -> View...**

#### Why UDF?

* **Predictability**: You always know where the state comes from (the ViewModel).
* **Debuggability**: If the UI looks wrong, you just check the State in the ViewModel.
* **Decoupling**: The View handles *how* to show data, not *how* to calculate it.

### State Management

---

How do we represent the screen's state? Usually, with a single **Data Class**.

❌ **Bad**: Multiple variables in ViewModel.

```kotlin
val isLoading = MutableStateFlow(false)
val userName = MutableStateFlow("")
val error = MutableStateFlow<String?>(null)
// Hard to ensure consistency. What if isLoading is true AND error is not null?
```

✅ **Good**: Single Source of Truth.

```kotlin
data class UserUiState(
    val isLoading: Boolean = true,
    val name: String = "",
    val error: String? = null
)

// In ViewModel
_uiState.update { it.copy(isLoading = false, name = "Bob") }
```

### Alternatives

---

#### MVI (Model-View-Intent)

* Very similar to MVVM with UDF.
* **Intent**: Represents "User intention" (like `LoadData`, `SubmitForm`).
* Strictly functional: `calculate(oldState, intent) -> newState`.
* Popular in KMP communities, often requires more boilerplate than simple MVVM.

#### MVP (Model-View-Presenter)

* Older pattern.
* The **Presenter** knows about the View interface and calls methods like `view.showLoading()`.
* **Downside**: Tightly coupled to the View interface. Harder to test than ViewModel.

## Resources

- [Android Architecture Guide: UI Layer](https://developer.android.com/topic/architecture/ui-layer)
- [Guide to App Architecture](https://developer.android.com/topic/architecture)

## Next Steps

Continue to [Module 04: Kotlin Language Features](../04-kotlin-fundamentals/README.md)
