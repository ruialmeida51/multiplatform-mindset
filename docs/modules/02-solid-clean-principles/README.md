# Module 02: SOLID and CLEAN Principles

## Overview

Understanding software design principles is crucial for building maintainable, scalable applications. This module covers
SOLID principles, CLEAN architecture concepts, and the fundamentals of coupling and cohesion.

## Learning Objectives

- Master SOLID principles (Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency
  Inversion)
- Understand CLEAN architecture principles
- Learn about coupling and cohesion in software design

### SOLID Principles Overview

---

SOLID is an acronym for five design principles intended to make software designs more understandable, flexible, and
maintainable.

#### S - Single Responsibility Principle (SRP)

**"A class should have one, and only one, reason to change."**

Instead of a "God Class" that does everything (fetching data, parsing it, saving to DB, updating UI), break it down.

**Example:**

❌ **Bad:** A `UserHandler` that handles validation, database saving, and email notifications.

✅ **Good:** Split into `UserValidator`, `UserRepository`, and `EmailService`.

```kotlin
// ❌ Violation
class UserHandler {
    fun register(user: User) {
        if (user.email.isEmpty()) throw Error("Invalid") // Validation logic
        database.save(user) // DB logic
        smtpClient.send("Welcome!", user.email) // Notification logic
    }
}

// ✅ Correct
class UserRegistrationUseCase(
    private val validator: UserValidator,
    private val repository: UserRepository,
    private val emailService: EmailService
) {
    fun register(user: User) {
        validator.validate(user)
        repository.save(user)
        emailService.sendWelcomeEmail(user)
    }
}
```

#### O - Open/Closed Principle (OCP)

**"Software entities should be open for extension, but closed for modification."**

You should be able to add new functionality without changing existing code.

**Example:**
If you want to calculate area for a new shape, you shouldn't have to modify the `AreaCalculator` class.

```kotlin
interface Shape {
    fun area(): Double
}

class Circle(val radius: Double) : Shape {
    override fun area() = Math.PI * radius * radius
}

class Rectangle(val w: Double, val h: Double) : Shape {
    override fun area() = w * h
}

// this function never needs to change, even if we add 'Triangle' later
fun printArea(shape: Shape) {
    println("Area: ${shape.area()}")
}
```

#### L - Liskov Substitution Principle (LSP)

**"Subtypes must be substitutable for their base types."**

If class B inherits from class A, we should be able to use B anywhere we use A without breaking the program.

**Example:**
A `Penguin` is a `Bird`, but if `Bird` has a `fly()` method, `Penguin` violates LSP because it cannot fly.

```kotlin
open class Bird {
    open fun move() {}
}

class Sparrow : Bird() {
    override fun move() {
        fly()
    }
    fun fly() { /*...*/
    }
}

class Penguin : Bird() {
    override fun move() {
        swim()
    } // Penguin moves by swimming, still a valid 'move'
    fun swim() { /*...*/
    }
}
```

#### I - Interface Segregation Principle (ISP)

**"Clients should not be forced to depend upon interfaces that they do not use."**

It's better to have many specific interfaces than one general-purpose interface.

```kotlin
// ❌ Violation
interface InterfaceMegaProtocol {
    fun onClick()
    fun onLongClick()
    fun onScroll()
    fun onSwipe()
}

// ✅ Correct
interface ClickListener {
    fun onClick()
}
interface ScrollListener {
    fun onScroll()
}
```

#### D - Dependency Inversion Principle (DIP)

**"Depend on abstractions, not on concretes."**

High-level modules (business logic) should not depend on low-level modules (database, network). Both should depend on
abstractions (interfaces).

```kotlin
// ❌ Violation: High-level 'Repository' depends on low-level 'CheckDatabase'
class UserRepository {
    private val db = SQLDatabase() // Hard dependency
}

// ✅ Correct: Depends on abstraction
interface Database {
    fun save(data: Any)
}

class UserRepository(private val db: Database) { // Injected
    fun saveUser(user: User) {
        db.save(user)
    }
}
```

### 2. CLEAN Architecture

---

Clean Architecture (by Robert C. Martin) guides us to separate concerns into layers. The most critical rule is the *
*Dependency Rule**:

> Source code dependencies can only point inwards. Nothing in an inner circle can know anything at all about something
> in an outer circle.

#### The Layers (Inner to Outer)

1. **Entities (Enterprise Business Rules)**
    * Pure business objects (e.g., `User`, `Loan`).
    * Least likely to change when external things (UI, DB) change.
    * No Android/iOS code here. Pure Kotlin.

2. **Use Cases (Application Business Rules)**
    * Specific actions the application can do (e.g., `GetUserProfile`, `TransferMoney`).
    * Orchestrates data flow to/from Entities.

3. **Interface Adapters (Presenters, ViewModels, Gateways)**
    * Converts data from Use Case format to UI format.
    * Implements the interfaces defined by Use Cases (like Repositories).

4. **Frameworks & Drivers (UI, DB, Network)**
    * The "glue" code. Android Activities, iOS ViewControllers, Room Database, Retrofit.

#### Example Flow

Consider a "Login" feature:

1. **UI (Outer)**: User clicks "Login Button". Calls `LoginViewModel`.
2. **Presentation**: `LoginViewModel` calls `LoginUseCase`.
3. **Domain (Inner)**: `LoginUseCase` asks `UserRepository` interface to find user.
4. **Data (Outer implementation)**: `UserRepositoryImpl` (in Data layer) calls specific API.

<img src="/images/manual_clean_diagram.png" width="600"/>

### 3. Coupling and Cohesion

---

These properties measure quality of design.

#### Cohesion (We want HIGH Cohesion)

* **Definition**: How closely related the parts of a class/module are.
* **High Cohesion**: A class `Calculator` only contains math functions (`add`, `sub`). Easy to understand.
* **Low Cohesion**: A class `Utility` contains `calculateTax`, `sendEmail`, `formatDate`. Hard to maintain.

#### Coupling (We want LOOSE Coupling)

* **Definition**: How dependent one module is on another.
* **Loose Coupling**: Components interact via interfaces. Changing one doesn't break the other.
* **Tight Coupling**: Class A directly instantiates Class B and accesses its fields. Changing B breaks A.

```kotlin
// ❌ Tight Coupling
class OrderService {
    val logger = FileLogger() // Hard-coded dependency
    fun placeOrder() {
        logger.log("Order placed")
    }
}

// ✅ Loose Coupling
class OrderService(val logger: Logger) { // Can be FileLogger, ConsoleLogger, or FakeLogger
    fun placeOrder() {
        logger.log("Order placed")
    }
}
```

### A simple modularisation structure

---

In a smaller project or when starting out, we often split the app into three core layers. This structure is easy to
navigate but can become a "bottleneck" as the team grows, because everyone is working in the same three modules.

The "Big Three" Structure:


- `:domain` (The Heart)
    - Contents: Entities, Use Cases, and Repository Interfaces.
    - Rule: Zero dependencies on other modules. Pure Kotlin/Java.

- `:data` (The Implementation)
    - Contents: Repository Implementations, Room Database, Retrofit/API services, Mappers.
    - Rule: Depends on :domain.

- `:ui` (or `:presentation`) (The Face)
    - Contents: ViewModels, Activities/Fragments, Compose UI, Dependency Injection setup.
    - Rule: Depends on :domain (to call Use Cases) and :data (to initialize the app).


### Feature based modularisation

---

As an app grows "Layer-Based" modularization fails because the modules become massive. Instead, we split the app by
Features (e.g., Auth, Profile, Chat).

A structure example:

- `:core:something` Shared logic, theme, and common utilities used by everyone.
- `:feature:auth:` Everything related to login/signup (Data, Domain, and UI for this feature only).
- `:feature:profile:` Everything related to the user profile.
- `:app:` The "shell" that ties all features together, our launcher.

#### Example

```
project-root/
├── app/                  # Main entry point, navigation graph
├── core/                 # Design system, Network engine, Extensions
│   ├── ui-components/
│   ├── coroutines-ktx/   
├── features/
│   ├── profile/          # User details, UpdateProfileUseCase    
│   │   ├── data/            
│   │   ├── domain/        
│   │   ├── presentation/  
│   ├── feed/             # Post list, PostRepository, LikeUseCase  
│   │   ├── data/            
│   │   ├── domain/        
│   │   ├── presentation/  
│   └── profile/          # User details, UpdateProfileUseCase     
│   │   ├── data/            
│   │   ├── domain/        
│   │   ├── presentation/    
```

I'd say there are two approaches available to us. We can take the by-the-book approach and keep the repositories as
a proxy between the data sources. Or eliminate that proxy completely and have the use cases communicate with the data
sources directly. Examples below.

```kotlin
// Data: depends on domain
interface UserRemoteSource {
    fun getUser(): User
}

class UserRemoteSourceImplementation(
    private val httpClient: HttpClient,
) : UserRemoteSource {
    override fun getUser(): User {
        // Some network operation
        return httpClient.get("myGetUserUrl")
    }
}

interface UserLocalSource {
    fun save(user: User): Boolean
}

class UserLocalSourceImplementation(
    private val databaseClient: Database
) : UserLocalSource {
    override fun save(user: User): Boolean {
        databaseClient.upsert(user)
    }
}

class UserRepositoryImplementation(
    private val localSource: UserLocalSource,
    private val remoteSource: UserRemoteSource,
) : UserRepository {
    override fun getUser(): User {
        return remoteSource
            .getUser()
            .also { fetchedUser -> localSource.save(fetchedUser) }
    }
}

// Domain: no dependencies
interface UserRepository {
    fun getUser(): User
}

class GetUserUseCase(private val userRepository: UserRepository) {
    fun getUser(): User = userRepository.getUser()
}

// Presentation: depends on domain
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

The CLEAN setup with repositories mediating between domain and data introduces unnecessary indirection:

- Repositories act as pass-through layers with no real logic.
- Use cases become anemic, delegating almost entirely to repositories.
- The boundary between when (domain) and how (data) to perform operations becomes unclear.

By simplifying this model, we aim to:

- Make domain logic meaningful again (the when).
- Remove redundant repository layers.
- Strengthen testability and developer ergonomics.
- Reduce boilerplate and mental overhead per feature.

So, an example with the simplified model:

```kotlin
// Data: depends on domain
class UserRemoteSourceImplementation(
    private val httpClient: HttpClient,
) : UserRemoteSource {
    override fun getUser(): User {
        // Some network operation
        return httpClient.get("myGetUserUrl")
    }
}

class UserLocalSourceImplementation(
    private val databaseClient: Database
) : UserLocalSource {
    override fun save(user: User): Boolean {
        databaseClient.upsert(user)
    }
}

// Domain: no dependencies
interface UserRemoteSource {
    fun getUser(): User
}

interface UserLocalSource {
    fun save(user: User): Boolean
}

class GetUserUseCase(
    private val userLocalSource: UserLocalSource,
    private val userRemoteSource: UserRemoteSource,
) {
    fun getUser(): User = remoteSource
        .getUser()
        .also { fetchedUser -> localSource.save(fetchedUser) }

}

// Presentation: depends on domain
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

## Resources

* [Philipp Lackner: Clean Code in 20 seconds](https://www.youtube.com/shorts/ySt16uOgpC4?feature=share)
* [Philipp Lackner: Should you Focus on Clean Architecture](https://www.youtube.com/shorts/gVZUjF_pabI?feature=share)

## Next Steps

Continue to [Module 03: MVVM and Unidirectional Data Flow](../03-mvvm-architecture/README.md)
