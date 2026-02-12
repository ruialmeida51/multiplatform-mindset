# Module 04: Kotlin Language Features

## Overview

Kotlin provides powerful language features that make multiplatform development more productive and safe. This module
covers essential Kotlin concepts you'll use daily.

## Learning Objectives

- Master Kotlin's type system and null safety
- Understand data classes and sealed hierarchies
- Learn functional programming constructs in Kotlin
- Use Kotlin's scope functions effectively
- Create and use extension functions

### Data Classes

----

Data classes are designed to hold data. They automatically derive the following methods from the properties declared in
the primary constructor:

- `equals()` / `hashCode()`
- `toString()` (e.g., "User(name=John, age=42)")
- `componentN()` functions (for destructuring)
- `copy()` -> creates a new instance inferred from the original with only the updated properties modified

```kotlin
data class User(val name: String, val age: Int)

fun main() {
    val user = User("Alice", 29)
    println(user) // User(name=Alice, age=29)

    // Copying: create a new instance with some properties modified
    val olderUser = user.copy(age = 30)

    // Destructuring: unpack properties into variables
    val (name, age) = user
    println("$name is $age years old")
}
```

```kotlin
data class DataClass(
    val param1: String,
    var param2: String,
)

fun main() {
    val immutableClass = DataClass(
        param1 = "1",
        param2 = "2"
    )

    immutableClass.param2 = "3"
    // = ? (It prints 3, changes in runtime affect all instances of this object)
    print(immutableClass.param2)
}
```

### Sealed Hierarchies

----

Sealed classes and interfaces represent restricted class hierarchies. They are useful when a value can have one of the
types from a limited set, but cannot have any other type. They are effectively "Enums on steroids" that can hold state.

#### Usage with `when`

When used in a `when` expression, the compiler ensures exhaustiveness.

```kotlin
sealed interface UiState {
    data object Loading : UiState
    data class Success(val data: List<String>) : UiState
    data class Error(val message: String) : UiState
}

fun render(state: UiState) {
    // Compile-time error if a case is missing and no else branch is provided
    when (state) {
        UiState.Loading -> println("Loading...")
        is UiState.Success -> println("Data: ${state.data}")
        is UiState.Error -> println("Error: ${state.message}")
    }
}
```

### Null Safety

----

Kotlin distinguishes between nullable and non-nullable types to prevent `NullPointerException`.

- **`String`**: Cannot hold `null`.
- **`String?`**: Can hold `null`.

#### Safety Operators

```kotlin
val text: String? = null

// 1. Safe Call Operator (?.)
// Returns null if text is null, otherwise returns length
val length: Int? = text?.length

// 2. Elvis Operator (?:)
// Returns the value on the left if not null, otherwise the value on the right
val safeLength: Int = text?.length ?: 0

// 3. Not-null Assertion (!!)
// Converts any value to a non-null type and throws an exception if the value is null
// Use with caution!
val forcedLength = text!!.length // Throws NPE if text is null
```

### Immutability: `val` vs `var`

----

- **`val` (Value)**: Immutable reference. Once assigned, it cannot be reassigned.
- **`var` (Variable)**: Mutable reference. Can be reassigned.

#### Reference Immutability vs Object Immutability

Declaring a variable as `val` does not mean the object itself is immutable, only that the *reference* to it cannot
change.

```kotlin
data class MutableContainer(var count: Int)

fun main() {
    val container = MutableContainer(0)

    // Valid: Modifying the object's internal state
    container.count = 1

    // Invalid: Reassigning the reference
    container = MutableContainer(2) // Compile Error
}
```

### Scope Functions

----

Kotlin provides five scope functions that execute a block of code within the context of an object. The difference lies
in:

1. How the object is available inside the block (`this` vs `it`).
2. What the expression returns (the object itself vs the lambda result).

| Function    | Object Ref | Returns        | Common Use Case                                                |
|:------------|:-----------|:---------------|:---------------------------------------------------------------|
| **`apply`** | `this`     | Context Object | Configuration needed for object initialization.                |
| **`also`**  | `it`       | Context Object | Side effects (logging, printing) without breaking call chains. |
| **`let`**   | `it`       | Lambda Result  | Null-safety checks (`?.let { ... }`) or converting objects.    |
| **`run`**   | `this`     | Lambda Result  | Object configuration and computing a result.                   |
| **`with`**  | `this`     | Lambda Result  | Grouping function calls on an object (non-extension).          |

#### Examples

```kotlin
// apply: Configure and return object
val person = Person().apply {
    name = "Dave"
    age = 25
}

// also - a side effect
doSomethingFunction().also { result ->
    logger.log("The result is $result")
}

// let: Null check
val name: String? = "Kotlin"
name?.let {
    // executed only if name is not null
    println("Length is ${it.length}")
}

// run: Compute result from object context
val isAdult = person.run {
    age >= 18
}

// with - group the context for a particular object
with(person) {
    val nameAndAge = "$name $age"
}
```

### Extension Functions

Extensions allow you to add functionality to existing classes without inheriting from them. If you come from a Java
background, they're kind of translated to the same thing as a global *insert visibility* static function. 

```kotlin
// Extending the standard String class
fun String.addExclamation(): String {
    return "$this, good sir!"
}

fun main() {
    println("Hello".addExclamation()) // Output: Hello, good sir!
}
```

### Control Flow: Ranges, Loops, and Labels

----

#### Ranges
Ranges allow you to define an interval of values. They are commonly used with `for` loops and `when` expressions.

- **Inclusive Range (`..`)**: Includes both the start and end values.
  ```kotlin
  val range = 1..10 // 1, 2, 3, ..., 10
  ```
- **Exclusive Range (`until`)**: Includes the start but excludes the end value.
  ```kotlin
  val range = 1 until 10 // 1, 2, 3, ..., 9
  ```
- **Descending Range (`downTo`)**: Counts backwards from the start to the end.
  ```kotlin
  val range = 10 downTo 1 // 10, 9, 8, ..., 1
  ```
- **Custom Step (`step`)**: Defines the increment between values in the range.
  ```kotlin
  val range = 1..10 step 2 // 1, 3, 5, 7, 9
  ```

#### Loops
Kotlin provides standard looping constructs similar to other languages but with a cleaner syntax for iteration.

- **For Loop**: Used to iterate over anything that provides an iterator (ranges, arrays, collections).
  ```kotlin
  val myList = listOf("1", "2", "3")
  
  for (item in myList) {
      println(item)
  }

  for (i in 1..5) {
      println("Iteration $i")
  }
  
  myList.forEach { item -> 
      doOperation(item) 
  }
  ```
- **While and Do-While**: Standard conditional loops.
  ```kotlin
  while (condition) {
      // execution
  }

  do {
      // execution
  } while (condition)
  ```

#### Labels and Qualified Returns
Labels allow you to control the flow of execution in nested loops or lambda expressions.

- **Qualified Break/Continue**: You can mark a loop with a label and use `break` or `continue` to target that specific loop.
  ```kotlin
  outer@ for (i in 1..3) {
      for (j in 1..3) {
          if (i == 2 && j == 2) break@outer
          println("i=$i, j=$j")
      }
  }
  ```
- Labels are defined using an identifier followed by the `@` sign (e.g., `loop@`).


## Resources

- [Kotlin Documentation: Data Classes](https://kotlinlang.org/docs/data-classes.html)
- [Kotlin Documentation: Sealed Classes](https://kotlinlang.org/docs/sealed-classes.html)
- [Kotlin Documentation: Null Safety](https://kotlinlang.org/docs/null-safety.html)
- [Kotlin Scope Functions Guide](https://kotlinlang.org/docs/scope-functions.html)
- [Kotlin Documentation: Control Flow](https://kotlinlang.org/docs/control-flow.html)
- [Kotlin Documentation: Returns and Jumps](https://kotlinlang.org/docs/returns.html)

## Next Steps

Continue to [Module 05: Coroutines Fundamentals](../05-coroutines/README.md)
