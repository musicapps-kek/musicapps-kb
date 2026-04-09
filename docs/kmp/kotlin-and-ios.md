# How Kotlin and iOS Work Together

Kotlin is a JVM-based language — so how does it end up running on an iPhone? The answer is Kotlin/Native, a compiler backend that turns Kotlin code into native binaries without a JVM.

## The compilation pipeline

```
Kotlin source code (commonMain / iosMain)
        ↓
  Kotlin/Native compiler
        ↓
  LLVM IR (same intermediate format as C/C++/Swift)
        ↓
  Native ARM64 binary code
        ↓
  Packaged as shared.framework
        ↓
  Linked into the iOS app by Xcode
```

The resulting code runs directly on the device — no JVM, no interpreter. Performance is comparable to Swift or C++.

## What Xcode sees: the framework

From Xcode's perspective, the shared Kotlin code is just a regular Apple framework (`shared.framework`). It doesn't know or care that it was written in Kotlin.

Swift code in the iOS app imports it like any other framework:

```swift
import shared

let song = Song(name: "Superstition", bpm: 100)
```

Kotlin classes, functions, and properties are automatically exposed to Swift. Kotlin/Native generates Objective-C headers (`.h` files) from your Kotlin code, which Swift can consume via its Objective-C interop layer.

## Kotlin → Swift type mapping

Kotlin/Native translates Kotlin types to their nearest equivalent:

| Kotlin | Swift |
|---|---|
| `String` | `String` |
| `Int` | `Int32` |
| `Long` | `Int64` |
| `Boolean` | `Bool` |
| `List<T>` | `[T]` (Array) |
| `Map<K,V>` | `[K:V]` (Dictionary) |
| `data class` | Regular class (no destructuring in Swift) |
| `sealed class` | `enum`-like pattern (limited in Swift) |
| `suspend fun` | Kotlin coroutine → special Swift async handling needed |

The translation is mostly seamless for simple types. More complex Kotlin patterns (sealed classes, coroutines, generics) require some care.

## Coroutines and Swift concurrency

Kotlin coroutines (`suspend` functions) don't map directly to Swift's `async/await`. This is one of the more painful edges of KMP.

The most common solution is **SKIE** (Swift Kotlin Interface Enhancer), a Gradle plugin that automatically wraps Kotlin coroutines so they appear as native Swift `async` functions:

```kotlin
// Kotlin (commonMain)
suspend fun loadSetlists(): List<Setlist>
```

```swift
// Swift — with SKIE, this just works
let setlists = try await repository.loadSetlists()
```

Without SKIE, you'd need to wrap coroutines manually in callbacks, which is tedious.

## Calling Apple APIs from Kotlin (iosMain)

In `iosMain`, you can call Apple's native frameworks directly from Kotlin. Kotlin/Native ships with bindings for the entire Apple SDK:

```kotlin
// iosMain
import platform.AVFAudio.AVAudioEngine
import platform.Foundation.NSDate

actual fun currentTimeMillis(): Long =
    (NSDate().timeIntervalSince1970 * 1000).toLong()
```

Apple framework names and types keep their original names. It feels slightly foreign in Kotlin, but it works.

## Memory management

Older versions of Kotlin/Native used a strict ownership model that made sharing objects between threads painful. Since Kotlin 1.7.20, the **new memory manager** (now the default) uses a model much closer to the JVM — standard garbage collection, no strict thread ownership. This eliminates most of the old pain points.

## What this means for SessionClick

| Component | Where it lives | How it reaches iOS |
|---|---|---|
| `Song`, `Setlist` models | `commonMain` | Compiled into `shared.framework`, imported in Swift |
| JSON serialization | `commonMain` | Same |
| BPM / tick logic | `commonMain` | Same |
| Audio engine | `iosMain` | Uses `AVAudioEngine` directly, compiled into framework |
| SwiftUI views | `iosApp` (Swift only) | Calls shared framework via `import shared` |
| StoreKit (IAP) | `iosApp` (Swift only) | Pure Swift, no Kotlin involved |

## Further reading

- [Kotlin/Native overview — kotlinlang.org](https://kotlinlang.org/docs/native-overview.html)
- [SKIE — Swift-friendly KMP — skie.touchlab.co](https://skie.touchlab.co)
- [KMP iOS integration guide — kotlinlang.org](https://kotlinlang.org/docs/multiplatform-ios-integration.html)
