# Module 01: Understanding KMP and CMP

## Overview

This module introduces Kotlin Multiplatform (KMP) and Compose Multiplatform (CMP), exploring what problems they solve
and why they might be the right choice for your project.

## Learning Objectives

- Understand what Kotlin Multiplatform (KMP) is and the problems it solves
- Learn about Compose Multiplatform (CMP) and declarative UI
- Compare KMP/CMP with other cross-platform solutions (Flutter, React Native, etc.)
- Understand when to choose KMP/CMP for your project

## Topics Covered

- [What is Kotlin Multiplatform and Compose Multiplatform?](#1-what-is-kotlin-multiplatform-and-compose-multiplatform)
- [KMP/CMP vs Alternatives](#2-kmpcmp-versus-alternatives)
- [When to use and not to use KMP/CMP](#3-when-to-use-and-not-use-kmpcmp)

### 1. What is Kotlin Multiplatform and Compose Multiplatform?

----

#### Kotlin Multiplatform

Traditionally, each platform would require its own codebase which would, in many cases, duplicate the same business
logic and domain models and/or entities. For example, a payment validation function might be implemented separately in
Swift (iOS), Kotlin (Android), and TypeScript (Web), each with potential inconsistencies. This creates several dilemmas:

- **Synchronization issues**: Multiple teams repeating logic can lead to difficulty ensuring all implementations are
  identical. A simple misunderstanding can result in divergent behavior across platforms.

- **Consistency challenges**: Different implementation speeds make it hard to maintain consistent behavior across all
  platforms.

- **Duplicated effort**: High development cost from writing the same logic multiple times, wasting valuable developer
  focus and mental clarity.

KMP appears as a tool that centralizes business logic and compiles it to multiple targets (JVM bytecode for Android,
native binaries for iOS, JavaScript for web). Other apps can consume this as an SDK that encapsulates domain and data
layer logic, leaving only the UI to be implemented per platform. This deduplicates everything from the UI layer down.

#### Compose Multiplatform

Compose Multiplatform comes to solve the missing piece of the previous statement, where KMP would only touch data and
domain layers, Compose Multiplatform aims to complete this loop and tackle the UI layer - what if we could share UI
across implementations as well? The problems this is trying to solve are a very similar, if not equal, to the same
issues Kotlin Multiplatform aims to address.

Compose Multiplatform provides a way for developers to write UI code only once and compile it to multiple targets.

----

### 2. KMP/CMP Versus Alternatives

----

If you thought that there are a bunch of other alternatives to Kotlin and Compose Multiplatform, you'd be correct. So
why would you choose this tool as opposed to any other? To answer this question we need to take a step back and
understand what options were available before:

_Disclaimer: to my knowledge, there are obviously more such tools, like Skia or Valdi from snapchat, but I'll
stick to the most popular ones to my knowledge and at the time of writing for now_

- We could write shared code with, for example, C++ or Rust and compile it for both iOS and Android, loading them
  through the respective native binary loaders (JNI for Android, for example). Besides being written in a possible
  separate language than the client we're building for, it requires a lot of manual setup and can be cumbersome to work
  with since Swift, for example, cannot read C++ code directly - so we'd need an intermediate layer to communicate with
  the Swift code written, most likely, in Objective-C.

- Flutter and React Native do not provide a clear way to share **only** domain or data logic without commiting the
  entire UI layer along with it, so they are out of the question for this approach.

- Or.. do it natively on each target.

So, what does Kotlin Multiplatform do differently? This is the only tool where we can share data and/or domain business
rules as a standard native library that we add to our projects - since it compiles to our local language: Swift for iOS,
Kotlin/Java for Android, Java for the JVM and other types of native binaries for other operating systems such as Linux
or Windows. This means, for the developer, a KMP library is just another native library added to the project he can
access directly without hassle - no bridges or virtualization in the middle.

This massively improves the developer experience when consuming these shared codebases. Another advantage to Kotlin
Multiplatform (and also Compose Multiplatform) is the interoperability. Now, you could technically do this with C++ too,
but here it’s just... easier. You don't have to jump through hoops to talk to the underlying system. If you need to use
a specific iOS API for biometrics or a weird Android-only sensor, you just call it. You aren't "trapped" inside the
framework.

What about Compose Multiplatform, specifically? If we look at Compose Multiplatform on its own, it's an entirely
different beast when compared to Flutter or React Native.

- **React Native** writes UI in JavaScript (or TypeScript), but your phone doesn't run JavaScript natively. So there's a
  "bridge" - a translator sitting between your JS code and the actual native UI components. Every time you want to
  update a button or scroll a list, that message has to cross the bridge. This translation takes time, and when you're
  doing it thousands of times per second (like during animations), that adds up. Although, there are some steps taken
  by the React Native team to reduce this performance gap with the new JSI (javascript interface).

- **Flutter** takes a completely different approach. Instead of using native UI components, it draws everything itself
  directly onto a native canvas. It renders pixels from scratch for every button, text field, and animation. This means
  it's fast (no bridge!) and looks identical on every platform. The downside is that your app won't automatically feel
  "native" to each platform - a Flutter button looks like a Flutter button, not an iOS button or an Android button
  (unless you explicitly style it that way).

**Compose Multiplatform** borrows Flutter's rendering approach - it also draws to a canvas, giving you the same
performance benefits and cross-platform consistency. But here's the key difference: you're not locked in. With CMP, you
can adopt it gradually, screen by screen, just like with KMP. Already have a native iOS app? You can write one new
screen in Compose Multiplatform and leave the rest native. Want to keep your native navigation? Go for it. Need a
specific screen to be fully native for performance or platform-specific reasons? Write it natively. Everything plays
nicely together. Although, to be fair, we still have the same downside as flutter, where a Compose Multiplatform button
will look the same across platforms. This is easily solved, however, with the `expect` `actual` mechanism, that we'll
talk further down the line in these modules, where we can provide a specific implementation for each targeted platform.

And what if we join KMP and CMP - we get a great combo. Why?

Think about it this way: **KMP is the brain, CMP is the body**. KMP handles all the thinking - your business logic, data
models, API calls, validation rules. CMP handles the body - what users see and touch. Together, they solve the "I hate
repeating myself" problem at every level of your app. You're sharing everything from end to end with the flexibility of
adopting and sharing as much as you need and want to. There's no bridge like React Native, and the code compiles down
to actual platform binaries. An Android user gets an Android binary, an iOS user gets an iOS binary. Your shared Kotlin
code becomes actual Swift interop code on iOS and actual Kotlin/JVM code on Android.

For example, your payments screen needs to integrate deeply with Apple Pay or Google Pay using platform-specific APIs
write that one natively. Or maybe you share the logic with KMP but write the UI natively on each platform. You get to
choose, feature by feature, screen by screen, what makes sense to share and what makes sense to keep native.
It's flexible in a way the other frameworks just aren't.

----

### 3. When to use and not use KMP/CMP

To be added.

## Resources

_Content to be added_

## Next Steps

Continue to [Module 02: SOLID and CLEAN Principles](../02-solid-clean-principles/README.md)
