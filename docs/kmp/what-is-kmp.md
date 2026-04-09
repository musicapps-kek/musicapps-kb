# What is Kotlin Multiplatform?

Kotlin Multiplatform (KMP) lets you write shared Kotlin code that runs on multiple platforms — Android, iOS, desktop, web — while keeping platform-specific code where it belongs.

It is **not** a cross-platform UI framework. You still write native UI on each platform (Jetpack Compose on Android, SwiftUI on iOS). KMP is about sharing the *logic* underneath the UI.

## The core idea

In any app, a large portion of the code has nothing to do with the platform:

- Data models (a `Song` with a name and a BPM is the same on Android and iOS)
- Business logic (calculating the interval between metronome ticks)
- Data persistence and serialization (reading/writing JSON setlists)
- Rules (freemium gate: max 3 setlists in free tier)

KMP lets you write this code once in Kotlin, and use it from both Android and iOS.

## How it fits into a project

A KMP project has (at minimum) three modules:

```
shared/
├── commonMain/     ← pure Kotlin, no platform APIs
├── androidMain/    ← Android-specific implementations
└── iosMain/        ← iOS-specific implementations

composeApp/         ← Android app (Jetpack Compose UI)
iosApp/             ← iOS app (Xcode project, SwiftUI UI)
```

Code in `commonMain` can be used by both `composeApp` and `iosApp`. Code in `androidMain` or `iosMain` is only compiled for that platform.

## The expect/actual mechanism

When you need platform-specific behaviour but want to call it from shared code, KMP uses `expect` and `actual`:

```kotlin
// commonMain — declares the interface
expect fun currentTimeMillis(): Long

// androidMain — Android implementation
actual fun currentTimeMillis(): Long = System.currentTimeMillis()

// iosMain — iOS implementation
actual fun currentTimeMillis(): Long = 
    NSDate().timeIntervalSince1970.toLong() * 1000
```

Shared code calls `currentTimeMillis()` without knowing which platform it's on. The compiler wires up the right implementation.

## What KMP is NOT

- **Not Kotlin Multiplatform Mobile (KMM)** — KMM was the old name for the mobile-focused subset. It's now just called KMP.
- **Not Compose Multiplatform** — that's a separate JetBrains project that shares UI across platforms using Compose. KMP shares logic; Compose Multiplatform shares UI. They can be combined, but they're distinct.
- **Not a replacement for native development** — you still need Xcode for iOS and Android Studio for Android. KMP reduces duplication, it doesn't eliminate platform work.

## What is shared in SessionClick

| Code | Where |
|---|---|
| `Song`, `Setlist` data models | `commonMain` |
| BPM calculation, tick interval logic | `commonMain` |
| JSON export/import | `commonMain` |
| Freemium rules (max songs/setlists) | `commonMain` |
| Audio engine | `androidMain` / `iosMain` (platform-specific) |
| UI | `composeApp` (Compose) / `iosApp` (SwiftUI) |
| In-app purchase | `composeApp` (Google Play Billing) / `iosApp` (StoreKit) |

## Further reading

- [Kotlin Multiplatform overview — kotlinlang.org](https://kotlinlang.org/docs/multiplatform.html)
- [KMP project structure — kotlinlang.org](https://kotlinlang.org/docs/multiplatform-discover-project.html)
