🚧 IN CONSTRUCTION & REVIEW

---

# Module 11: Dependency Injection

---

🚧 IN CONSTRUCTION & REVIEW

## Overview
Dependency Injection is a crucial pattern for building testable, maintainable applications. This module covers IoC (Inversion of Control), various DI frameworks, and best practices for multiplatform projects.

## Learning Objectives
- Understand Inversion of Control (IoC) and Dependency Injection
- Learn the benefits of using DI in your projects
- Compare different DI solutions (Dagger, Koin, Hilt, etc.)
- Choose the right DI framework for your multiplatform project
- Distinguish between DI containers and Service Locators

## Topics Covered

### 1. Inversion of Control (IoC) and Dependency Injection

----

- **Inversion of Control (IoC)**: A design principle where the control of object creation and lifecycle is transferred
  from the object itself to a container or framework.
- **Dependency Injection (DI)**: A pattern that implements IoC. Instead of an object creating its dependencies, they are
  "injected" into it (usually via the constructor).

**Benefits**:
- **Testability**: You can easily swap real implementations with mocks/fakes in tests.
- **Decoupling**: Classes don't need to know how to create their dependencies.
- **Maintainability**: Changing an implementation only needs to be done in one place (the DI configuration).

### 2. DI Fundamentals

----

- **Constructor Injection**: (Recommended) Dependencies are passed through the constructor.
- **Property Injection**: Dependencies are assigned to properties after object creation.
- **Interface-based Dependencies**: Always depend on interfaces, not concrete classes.

```kotlin
// ✅ Constructor Injection
class UserViewModel(private val repository: UserRepository) : ViewModel()
```

### 3. DI Frameworks Overview

----

- **Koin**: A lightweight, Kotlin-native dependency injection framework (actually a Service Locator). Very popular in
  KMP because it's easy to set up and has great multiplatform support.
- **Hilt / Dagger 2**: Standard for Android. Compile-time safe but depends heavily on code generation (KSP/KAPT), which
  can be more complex to set up in KMP.
- **Manual DI**: Creating and passing dependencies manually. Good for very small projects but doesn't scale well.

### 4. Koin Deep Dive

----

- **Service Locator vs DI Container**: Koin is technically a Service Locator. The main difference is that in a true DI
  container (like Dagger), dependencies are "pushed" into classes. In a Service Locator, classes "request" dependencies
  from a central registry.
- **Setup**:
```kotlin
val appModule = module {
    single<UserRepository> { UserRepositoryImpl(get(), get()) }
    factory { GetUserUseCase(get()) }
    viewModel { UserViewModel(get()) }
}
```
- **KMP Support**: Koin is written in pure Kotlin, so it works seamlessly across Android, iOS, Desktop, and Web. You
  can define your modules in `commonMain` and start Koin in the platform-specific entry points.

### 5. Choosing a DI Solution

----

| Feature             | Koin                   | Hilt / Dagger          |
|:--------------------|:-----------------------|:-----------------------|
| **KMP Support**     | Excellent              | Limited (Android only) |
| **Learning Curve**  | Low                    | High                   |
| **Safety**          | Runtime                | Compile-time           |
| **Setup Speed**     | Fast                   | Slow                   |

**Recommendation**: Use **Koin** for Kotlin Multiplatform projects unless you have a very large team and strict
requirements for compile-time safety that justify the complexity of Dagger/Hilt.

### 6. Best Practices

----

- **Scope Management**: Use `single` for singletons (e.g., Repositories) and `factory` for objects that should be
  re-created every time (e.g., UseCases).
- **Module Organization**: Split your DI modules by feature or by layer (e.g., `dataModule`, `domainModule`, `uiModule`).
- **Avoid Common Pitfalls**: Don't use the Service Locator as a "God Object" by passing the Koin instance itself into
  your classes.

## Resources

- [Koin Official Documentation](https://insert-koin.io/)
- [Koin Multiplatform Sample](https://github.com/InsertKoinIO/koin-getting-started/tree/main/kotlin-multiplatform)
- [Article: Dependency Injection in KMP](https://medium.com/@pablichjenkov/dependency-injection-in-kotlin-multiplatform-koin-vs-manual-di-8b3684a0d845)

## Next Steps
Continue to [Module 12: Contributing](../12-contributing/README.md)
