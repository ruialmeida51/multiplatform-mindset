🚧 IN CONSTRUCTION & REVIEW

---

# Module 10: KMP/CMP Internals

---

🚧 IN CONSTRUCTION & REVIEW

## Overview
Understanding how Kotlin Multiplatform and Compose Multiplatform work under the hood helps you make better architectural decisions and debug issues more effectively.

## Learning Objectives
- Understand how KMP compiles to different platforms
- Learn about the Kotlin/Native memory model
- Explore how Compose Multiplatform renders on different platforms
- Understand interoperability with platform-specific code

## Topics Covered

### 1. Kotlin Multiplatform Architecture

----

KMP isn't a "framework" like Flutter; it's a set of compiler features.

- **Compilation Targets**: Each platform is a "target." For Android, it's JVM bytecode. For iOS, it's native LLVM
  binaries. For Web, it's JavaScript or WebAssembly.
- **Source Sets**: KMP uses "Source Sets" to organize code. `commonMain` is where shared code lives. `androidMain`,
  `iosMain`, etc., are where platform-specific code lives.
- **Module Structure**: A typical KMP project has a hierarchical structure where platform-specific source sets "depend
  on" `commonMain`.

### 2. Kotlin/Native (iOS)

----

Kotlin/Native is the technology that allows Kotlin to run on platforms where no JVM is available (like iOS).

- **Memory Model**: The new Kotlin/Native memory model (stable since Kotlin 1.7.20) has removed many of the previous
  restrictions (like freezing). It now works much like the JVM, with a shared heap and a garbage collector.
- **Garbage Collection**: Kotlin/Native uses a tracing garbage collector that runs in the background.
- **C Interop**: Kotlin/Native can directly call C and Objective-C libraries. It automatically generates Kotlin
  bindings for Apple's frameworks (like UIKit, Foundation).

### 3. Kotlin/JVM (Android)

----

On Android, Kotlin compiles to JVM bytecode. It's 100% interoperable with Java. This means KMP on Android has zero
performance overhead compared to standard Kotlin/Android.

### 4. Compose Multiplatform Rendering

----

How does CMP draw UI on so many platforms?

- **Skia**: On most platforms (Desktop, iOS, Web), Compose Multiplatform uses **Skia**, a high-performance 2D graphics
  library (the same one used by Chrome and Flutter).
- **Rendering**: CMP doesn't map to native components (like `UIButton` or `android.widget.Button`). Instead, it draws
  the pixels directly to a canvas provided by the platform.
- **Performance**: Because it uses Skia and leverages the GPU, CMP achieves a smooth 60/120 FPS experience.

### 5. Interoperability

----

- **iOS (Objective-C/Swift)**: Kotlin code is exported as a Framework that Swift can import. Note: Swift sees Kotlin
  classes through an Objective-C header, which can sometimes lead to "weird" naming or missing features (like sealed
  classes appearing as regular classes in Swift).
- **Android (JNI)**: While mostly hidden from you, KMP can interact with low-level Android C++ code via JNI if needed.

### 6. Build System: Gradle

----

Gradle is the engine that ties everything together. The `kotlin-multiplatform` plugin allows you to configure targets,
source sets, and dependencies in one `build.gradle.kts` file.

```kotlin
kotlin {
    androidTarget()
    iosArm64()
    iosSimulatorArm64()

    sourceSets {
        commonMain.dependencies {
            implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.8.0")
        }
        androidMain.dependencies { ... }
        iosMain.dependencies { ... }
    }
}
```

## Resources

- [Kotlin Documentation: Multiplatform project structure](https://kotlinlang.org/docs/multiplatform-project-structure.html)
- [JetBrains Blog: Compose Multiplatform for iOS is Alpha](https://blog.jetbrains.com/kotlin/2023/05/compose-multiplatform-for-ios-is-in-alpha/)
- [Kotlin/Native Memory Management](https://kotlinlang.org/docs/native-memory-manager.html)

## Next Steps
Continue to [Module 11: Dependency Injection](../11-dependency-injection/README.md)
