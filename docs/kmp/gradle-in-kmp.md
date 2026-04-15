# Gradle in a KMP Project

If you've built Android apps, you already know [Gradle](https://gradle.org) as the build system. In a KMP project, Gradle takes on a larger role — it manages not just the Android build, but also the compilation of shared Kotlin code and the integration with Xcode for iOS.

## What Gradle does in a KMP project

1. **Compiles shared Kotlin code** into the right output for each platform
2. **Manages dependencies** across modules (`shared`, `composeApp`, `iosApp`)
3. **Produces an iOS framework** (`.framework`) from the `shared` module that Xcode can consume
4. **Runs tasks** like tests, code generation, and packaging

## The file structure

A KMP project has multiple `build.gradle.kts` files — one per module:

```
SessionClick/
├── build.gradle.kts          ← root build file (project-wide config)
├── settings.gradle.kts       ← declares which modules exist
├── gradle.properties         ← global properties (versions, flags)
├── shared/
│   └── build.gradle.kts      ← shared module config (the most important one)
└── composeApp/
    └── build.gradle.kts      ← Android app config
```

The `iosApp/` folder is an Xcode project — it has no `build.gradle.kts`. Gradle produces a framework that Xcode picks up, but Xcode has its own build system for the rest.

## settings.gradle.kts

This is the entry point. It tells Gradle which modules are part of the project:

```kotlin
rootProject.name = "SessionClick"

include(":shared")
include(":composeApp")
```

## The shared module build file

This is where KMP is actually configured:

```kotlin
kotlin {
    androidTarget()        // compile for Android
    iosX64()               // compile for iOS simulator (Intel Mac)
    iosArm64()             // compile for real iOS device
    iosSimulatorArm64()    // compile for iOS simulator (Apple Silicon Mac)

    sourceSets {
        commonMain.dependencies {
            // dependencies available on all platforms
            implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.7.0")
        }
        androidMain.dependencies {
            // Android-only dependencies
        }
        iosMain.dependencies {
            // iOS-only dependencies (rare — usually using Apple frameworks directly)
        }
    }
}
```

## How Gradle produces the iOS framework

When you build the project, Gradle compiles the `shared` module into a native iOS framework called `shared.framework`. Xcode is configured to run a Gradle task as part of its build phase to get this framework.

This is why you must have a working Gradle setup before the iOS app can build — Xcode depends on it.

```
Gradle builds shared Kotlin code
        ↓
  shared.framework
        ↓
  Xcode links it into the iOS app
```

## gradle.properties

This file holds global configuration. In KMP projects, two entries are important:

```properties
kotlin.code.style=official
android.useAndroidX=true
```

You may also see KMP-specific flags here, like enabling the new Kotlin/Native memory model.

## Version catalogs (libs.versions.toml)

Modern KMP projects use a [version catalog](https://docs.gradle.org/current/userguide/version_catalogs.html) to centralise dependency versions:

```
gradle/
└── libs.versions.toml
```

```toml
[versions]
kotlin = "2.0.0"
kotlinx-serialization = "1.7.0"

[libraries]
kotlinx-serialization-json = { module = "org.jetbrains.kotlinx:kotlinx-serialization-json", version.ref = "kotlinx-serialization" }

[plugins]
kotlin-multiplatform = { id = "org.jetbrains.kotlin.multiplatform", version.ref = "kotlin" }
```

Instead of hardcoding version strings in each `build.gradle.kts`, you reference them as `libs.kotlinx.serialization.json`. This makes keeping versions consistent across modules much easier.

## Common Gradle tasks in KMP

| Task | What it does |
|---|---|
| `./gradlew build` | Build all modules |
| `./gradlew :shared:build` | Build only the shared module |
| `./gradlew :composeApp:assembleDebug` | Build Android debug APK |
| `./gradlew :shared:iosArm64Binaries` | Compile shared code for real iOS device |
| `./gradlew test` | Run all tests |
| `./gradlew :shared:commonTest` | Run only shared module tests |

## Further reading

- [Gradle basics for Android — developer.android.com](https://developer.android.com/build)
- [KMP Gradle DSL reference — kotlinlang.org](https://kotlinlang.org/docs/multiplatform-dsl-reference.html)
- [Gradle version catalogs — docs.gradle.org](https://docs.gradle.org/current/userguide/version_catalogs.html)
