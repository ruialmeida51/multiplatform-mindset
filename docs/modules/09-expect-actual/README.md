🚧 IN CONSTRUCTION & REVIEW

---

# Module 09: expect/actual for KMP/CMP

---

🚧 IN CONSTRUCTION & REVIEW

## Overview
The expect/actual mechanism is the cornerstone of Kotlin Multiplatform, allowing you to write platform-specific implementations while maintaining a common interface. This module explains how to use this powerful feature.

## Learning Objectives
- Understand the expect/actual mechanism
- Learn when to use platform-specific implementations
- Create common interfaces with platform-specific code
- Apply expect/actual to real-world scenarios

## Topics Covered

### 1. Understanding expect/actual

----

The `expect`/`actual` mechanism is Kotlin's way of handling platform-specific code.

- **What is it?**: It allows you to define a "contract" in the `commonMain` source set using the `expect` keyword, and
  provide platform-specific implementations in the corresponding platform source sets using the `actual` keyword.
- **How it Works**: The compiler ensures that every `expect` declaration has a matching `actual` implementation in every
  target platform you've defined in your project.
- **Compilation Process**: During compilation, the compiler "merges" the common code with the specific platform's
  actual implementations to create a final binary for that platform.

### 2. Expected Declarations

----

You can use `expect` with various Kotlin constructs:

- **Functions**: `expect fun getPlatformName(): String`
- **Classes**: `expect class PlatformHttpClient()`
- **Properties**: `expect val platformVersion: Int`
- **Objects**: `expect object PlatformConfig`

### 3. Actual Implementations

----

For each platform (Android, iOS, etc.), you must provide the implementation:

```kotlin
// In commonMain
expect fun getPlatformName(): String

// In androidMain
actual fun getPlatformName(): String = "Android ${android.os.Build.VERSION.SDK_INT}"

// In iosMain
actual fun getPlatformName(): String = UIDevice.currentDevice.systemName()
```

### 4. Common Patterns

----

- **Platform Info**: Getting OS version, device model, or unique IDs.
- **File System**: Accessing specific directories (like `Documents` on iOS vs `Internal Storage` on Android).
- **Network Configuration**: Configuring SSL pinning or proxy settings that vary by platform.
- **UI Components**: Sometimes you need a platform-specific view (like a `MapView` or `WebView`) that isn't yet
  available in Compose Multiplatform.

### 5. Best Practices

----

- **Minimize Platform-Specific Code**: Try to keep as much logic as possible in `commonMain`. Use `expect`/`actual` only
  when there is no other way to access a platform API.
- **Interfaces over expect/actual**: Often, a regular interface in `commonMain` and an implementation passed via
  Dependency Injection (like Koin) is cleaner than `expect`/`actual`. Use `expect`/`actual` when you need the compiler to
  guarantee the existence of the implementation or when you need to use platform-specific types in the signature.
- **Organize by Feature**: Keep your `expect` and `actual` files organized in the same package structure across source
  sets.

### 6. Advanced: Type Aliases

----

Sometimes you want to use an existing platform class as an `actual` implementation.

```kotlin
// commonMain
expect class AtomicInt(value: Int) {
    fun increment(): Int
}

// jvmMain
actual typealias AtomicInt = java.util.concurrent.atomic.AtomicInteger
```

## Resources

- [Kotlin Documentation: Connect to platform-specific APIs](https://kotlinlang.org/docs/multiplatform-connect-to-apis.html)
- [Article: Expect/Actual is a last resort](https://touchlab.co/expect-actual-is-a-last-resort/)
- [KMP Samples: Platform-specific code](https://github.com/Kotlin/kmp-snippets/tree/master/platform-specific-code)

## Next Steps
Continue to [Module 10: KMP/CMP Internals](../10-kmp-internals/README.md)
