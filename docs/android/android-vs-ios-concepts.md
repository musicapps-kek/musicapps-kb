# Android vs iOS — Concepts & Terminology

If you know Android development, you already understand most of the concepts in iOS development. The platforms solve the same problems — they just use different names and slightly different approaches.

## App entry points

| Concept | Android | iOS |
|---|---|---|
| App definition | `AndroidManifest.xml` + `Application` class | `@main` / `App` struct (SwiftUI) |
| Main entry point | `MainActivity` | `ContentView` (first SwiftUI view) |
| App lifecycle | `Application.onCreate()` | `App.body` scene |

In SwiftUI, the app entry point is a struct conforming to the `App` protocol:

```swift
@main
struct SessionClickApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

This is roughly equivalent to your `Application` class + `MainActivity` combined.

## Screens and navigation

| Concept | Android | iOS |
|---|---|---|
| A single screen | `Fragment` / `@Composable` | `View` (UIKit) / `View` struct (SwiftUI) |
| Navigation container | `NavController` (Jetpack) | `NavigationStack` (SwiftUI) |
| Navigate to screen | `navController.navigate("route")` | `.navigationDestination` + `NavigationLink` |
| Back stack | Managed by `NavController` | Managed by `NavigationStack` |
| Tab bar | `BottomNavigationBar` | `TabView` |
| Modal/bottom sheet | `ModalBottomSheet` | `.sheet()` modifier |

## UI building blocks

| Concept | Android (Compose) | iOS (SwiftUI) |
|---|---|---|
| Vertical list | `LazyColumn` | `List` / `LazyVStack` |
| Horizontal list | `LazyRow` | `LazyHStack` |
| Text | `Text()` | `Text()` |
| Button | `Button()` | `Button()` |
| Image | `Image()` | `Image()` |
| Text input | `TextField()` | `TextField()` |
| Layout — vertical | `Column` | `VStack` |
| Layout — horizontal | `Row` | `HStack` |
| Layout — overlapping | `Box` | `ZStack` |
| Padding/spacing | `.padding()` modifier | `.padding()` modifier |

Compose and SwiftUI are strikingly similar in philosophy — both are declarative, both use modifiers/chaining. If you're comfortable with Compose, SwiftUI will feel familiar within a few hours.

## State management

| Concept | Android (Compose) | iOS (SwiftUI) |
|---|---|---|
| Local UI state | `remember { mutableStateOf() }` | `@State` |
| Shared state (ViewModel) | `ViewModel` + `StateFlow` | `@StateObject` / `@ObservableObject` |
| Observe state in UI | `collectAsState()` | `@ObservedObject` / `@EnvironmentObject` |
| Side effects | `LaunchedEffect` | `.task {}` / `.onAppear {}` |

The mental model is the same: state lives somewhere, the UI reacts to it. The wiring looks different but the idea is identical.

## ViewModels

Android's `ViewModel` has a direct equivalent in iOS. In modern SwiftUI (iOS 17+), you use the `@Observable` macro:

```swift
// iOS (SwiftUI)
@Observable
class MetronomeViewModel {
    var bpm: Int = 120
    var isPlaying: Bool = false

    func togglePlay() { isPlaying.toggle() }
}
```

```kotlin
// Android (Compose)
class MetronomeViewModel : ViewModel() {
    var bpm by mutableStateOf(120)
    var isPlaying by mutableStateOf(false)

    fun togglePlay() { isPlaying = !isPlaying }
}
```

In a KMP project, you can put the *logic* in the `shared` module and only keep the platform-specific wiring in each ViewModel.

## Lifecycle

| Concept | Android | iOS |
|---|---|---|
| App comes to foreground | `onResume()` | `scenePhase == .active` |
| App goes to background | `onPause()` | `scenePhase == .background` |
| View appears | `LaunchedEffect(Unit)` | `.onAppear {}` |
| View disappears | `DisposableEffect` cleanup | `.onDisappear {}` |
| Low memory warning | `onLowMemory()` | `didReceiveMemoryWarning` |

## Permissions

| Concept | Android | iOS |
|---|---|---|
| Declare permissions | `AndroidManifest.xml` | `Info.plist` |
| Request at runtime | `rememberLauncherForActivityResult` | `AVAudioSession.requestRecordPermission` etc. |
| Permission model | Grouped, user can revoke | Per-feature, managed by system |

## Data storage

| Concept | Android | iOS |
|---|---|---|
| Key-value storage | `SharedPreferences` / `DataStore` | `UserDefaults` |
| Local database | `Room` | `CoreData` / `SwiftData` |
| File storage | `Context.filesDir` | `FileManager` / app sandbox |
| Keychain / secure storage | `EncryptedSharedPreferences` | `Keychain` |

## In-app purchases

| | Android | iOS |
|---|---|---|
| Framework | Google Play Billing Library | StoreKit 2 |
| Purchase type (one-time) | `ProductType.INAPP` | `.nonConsumable` |
| Restore purchases | `queryPurchasesAsync()` | `Transaction.currentEntitlements` |
| Sandbox testing | Internal test track / license testers | Sandbox Apple ID |

The concepts are the same (products, purchases, entitlements) but the APIs are completely different and platform-specific. Neither can be shared via KMP.

## Packaging and distribution

| | Android | iOS |
|---|---|---|
| App package | `.apk` / `.aab` | `.ipa` |
| Store | Google Play | App Store |
| Beta testing | Internal/Closed/Open tracks | TestFlight |
| Sideloading | Allowed (with settings) | Restricted (EU only with DMA) |
| Review process | Mostly automated, fast | Manual review, 1-3 days |

## Key iOS concepts with no direct Android equivalent

- **Scenes** — iOS supports multiple windows of the same app (iPad). Handled via `Scene` in SwiftUI.
- **App Groups** — share data between your app and extensions (widgets, share extensions). No Android equivalent.
- **Entitlements** — a signed list of capabilities (push notifications, iCloud, etc.) granted by Apple. Roughly equivalent to permissions, but granted at build time, not runtime.
- **Provisioning profiles** — certificates that prove "this build was made by this developer for these devices." Android uses keystore signing but without the provisioning complexity.
- **Info.plist** — XML config file declaring app capabilities, required permissions, URL schemes, etc. Roughly combines `AndroidManifest.xml` and some Gradle config.

## Further reading

- [SwiftUI tutorials — developer.apple.com](https://developer.apple.com/tutorials/swiftui)
- [Jetpack Compose vs SwiftUI comparison — proandroiddev.com](https://proandroiddev.com)
- [Human Interface Guidelines — developer.apple.com](https://developer.apple.com/design/human-interface-guidelines/)
