# Android vs iOS — Concepts & Terminology

If you know Android development, you already understand most of the concepts in iOS development. The platforms solve the same problems — they just use different names and slightly different approaches.

## App entry points

| Concept          | Android                                     | iOS                                |
| ---------------- | ------------------------------------------- | ---------------------------------- |
| App definition   | [`AndroidManifest.xml`](https://developer.android.com/guide/topics/manifest/manifest-intro) + [`Application`](https://developer.android.com/reference/android/app/Application) class | `@main` / [`App`](https://developer.apple.com/documentation/swiftui/app) struct (SwiftUI)   |
| Main entry point | `MainActivity`                              | `ContentView` (first SwiftUI view) |
| App lifecycle    | [`Application.onCreate()`](https://developer.android.com/reference/android/app/Application#onCreate())                    | `App.body` scene                   |

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

| Concept              | Android                           | iOS                                         |
| -------------------- | --------------------------------- | ------------------------------------------- |
| A single screen      | [`Fragment`](https://developer.android.com/guide/fragments) / [`@Composable`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/Composable)        | `View` (UIKit) / `View` struct (SwiftUI)    |
| Navigation container | [`NavController`](https://developer.android.com/reference/androidx/navigation/NavController) (Jetpack)         | [`NavigationStack`](https://developer.apple.com/documentation/swiftui/navigationstack) (SwiftUI)                 |
| Navigate to screen   | `navController.navigate("route")` | `.navigationDestination` + `NavigationLink` |
| Back stack           | Managed by `NavController`        | Managed by `NavigationStack`                |
| Tab bar              | `BottomNavigationBar`             | [`TabView`](https://developer.apple.com/documentation/swiftui/tabview)                                   |
| Modal/bottom sheet   | [`ModalBottomSheet`](https://developer.android.com/develop/ui/compose/components/bottom-sheets)                | [`.sheet()`](https://developer.apple.com/documentation/swiftui/view/sheet(ispresented:ondismiss:content:)) modifier                         |

## UI building blocks

| Concept              | Android (Compose)     | iOS (SwiftUI)         |
| -------------------- | --------------------- | --------------------- |
| Vertical list        | [`LazyColumn`](https://developer.android.com/develop/ui/compose/lists)          | `List` / `LazyVStack` |
| Horizontal list      | [`LazyRow`](https://developer.android.com/develop/ui/compose/lists)             | `LazyHStack`          |
| Text                 | `Text()`              | `Text()`              |
| Button               | `Button()`            | `Button()`            |
| Image                | `Image()`             | `Image()`             |
| Text input           | `TextField()`         | `TextField()`         |
| Layout — vertical    | `Column`              | `VStack`              |
| Layout — horizontal  | `Row`                 | `HStack`              |
| Layout — overlapping | `Box`                 | `ZStack`              |
| Padding/spacing      | `.padding()` modifier | `.padding()` modifier |

Compose and SwiftUI are strikingly similar in philosophy — both are declarative, both use modifiers/chaining. If you're comfortable with Compose, SwiftUI will feel familiar within a few hours.

## State management

| Concept                  | Android (Compose)               | iOS (SwiftUI)                            |
| ------------------------ | ------------------------------- | ---------------------------------------- |
| Local UI state           | `remember { mutableStateOf() }` | [`@State`](https://developer.apple.com/documentation/swiftui/state)                                 |
| Shared state (ViewModel) | [`ViewModel`](https://developer.android.com/reference/androidx/lifecycle/ViewModel) + [`StateFlow`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/)       | [`@StateObject`](https://developer.apple.com/documentation/swiftui/stateobject) / [`@ObservableObject`](https://developer.apple.com/documentation/combine/observableobject)     |
| Observe state in UI      | [`collectAsState()`](https://developer.android.com/develop/ui/compose/state#state-in-composables)              | [`@ObservedObject`](https://developer.apple.com/documentation/swiftui/observedobject) / [`@EnvironmentObject`](https://developer.apple.com/documentation/swiftui/environmentobject) |
| Side effects             | [`LaunchedEffect`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#LaunchedEffect(kotlin.Any,kotlin.coroutines.SuspendFunction1))                | `.task {}` / `.onAppear {}`              |

The mental model is the same: state lives somewhere, the UI reacts to it. The wiring looks different but the idea is identical.

## ViewModels

Android's [`ViewModel`](https://developer.android.com/reference/androidx/lifecycle/ViewModel) has a direct equivalent in iOS. In modern SwiftUI (iOS 17+), you use the [`@Observable`](https://developer.apple.com/documentation/observation/observable()) macro:

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

In a KMP project, you can put the _logic_ in the `shared` module and only keep the platform-specific wiring in each ViewModel.

## Lifecycle

| Concept                 | Android                    | iOS                         |
| ----------------------- | -------------------------- | --------------------------- |
| App comes to foreground | [`onResume()`](https://developer.android.com/reference/android/app/Activity#onResume())               | `scenePhase == .active`     |
| App goes to background  | [`onPause()`](https://developer.android.com/reference/android/app/Activity#onPause())                | `scenePhase == .background` |
| View appears            | [`LaunchedEffect(Unit)`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#LaunchedEffect(kotlin.Any,kotlin.coroutines.SuspendFunction1))     | `.onAppear {}`              |
| View disappears         | [`DisposableEffect`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#DisposableEffect(kotlin.Any,kotlin.Function1)) cleanup | `.onDisappear {}`           |
| Low memory warning      | [`onLowMemory()`](https://developer.android.com/reference/android/app/Application#onLowMemory())            | `didReceiveMemoryWarning`   |

## Permissions

| Concept             | Android                             | iOS                                           |
| ------------------- | ----------------------------------- | --------------------------------------------- |
| Declare permissions | [`AndroidManifest.xml`](https://developer.android.com/guide/topics/manifest/manifest-intro)               | [`Info.plist`](https://developer.apple.com/documentation/bundleresources/information-property-list)                                  |
| Request at runtime  | [`rememberLauncherForActivityResult`](https://developer.android.com/develop/ui/compose/permissions) | [`AVAudioSession.requestRecordPermission`](https://developer.apple.com/documentation/avfaudio/avaudiosession/requestrecordpermission(_:)) etc. |
| Permission model    | Grouped, user can revoke            | Per-feature, managed by system                |

## Data storage

| Concept                   | Android                           | iOS                         |
| ------------------------- | --------------------------------- | --------------------------- |
| Key-value storage         | [`SharedPreferences`](https://developer.android.com/reference/android/content/SharedPreferences) / [`DataStore`](https://developer.android.com/topic/libraries/architecture/datastore) | [`UserDefaults`](https://developer.apple.com/documentation/foundation/userdefaults)              |
| Local database            | [`Room`](https://developer.android.com/training/data-storage/room)                            | [`CoreData`](https://developer.apple.com/documentation/coredata) / [`SwiftData`](https://developer.apple.com/documentation/swiftdata)    |
| File storage              | [`Context.filesDir`](https://developer.android.com/reference/android/content/Context#getFilesDir())                | `FileManager` / app sandbox |
| Keychain / secure storage | [`EncryptedSharedPreferences`](https://developer.android.com/reference/androidx/security/crypto/EncryptedSharedPreferences)      | [`Keychain`](https://developer.apple.com/documentation/security/keychain_services)                  |

## In-app purchases

|                          | Android                               | iOS                               |
| ------------------------ | ------------------------------------- | --------------------------------- |
| Framework                | [Google Play Billing Library](https://developer.android.com/google/play/billing)           | [StoreKit 2](https://developer.apple.com/documentation/storekit)                        |
| Purchase type (one-time) | `ProductType.INAPP`                   | `.nonConsumable`                  |
| Restore purchases        | `queryPurchasesAsync()`               | `Transaction.currentEntitlements` |
| Sandbox testing          | Internal test track / license testers | Sandbox Apple ID                  |

The concepts are the same (products, purchases, entitlements) but the APIs are completely different and platform-specific. Neither can be shared via KMP.

## Packaging and distribution

|                | Android                     | iOS                           |
| -------------- | --------------------------- | ----------------------------- |
| App package    | `.apk` / [`.aab`](https://developer.android.com/guide/app-bundle)             | `.ipa`                        |
| Store          | [Google Play](https://play.google.com/console)                 | [App Store](https://developer.apple.com/app-store/)                     |
| Beta testing   | Internal/Closed/Open tracks | [TestFlight](https://developer.apple.com/testflight/)                    |
| Sideloading    | Allowed (with settings)     | Restricted (EU only with DMA) |
| Review process | Mostly automated, fast      | Manual review, 1-3 days       |

## Key iOS concepts with no direct Android equivalent

- **[Scenes](https://developer.apple.com/documentation/uikit/app_and_environment/scenes)** — iOS supports multiple windows of the same app (iPad). Handled via `Scene` in SwiftUI.
- **[App Groups](https://developer.apple.com/documentation/xcode/configuring-app-groups)** — share data between your app and extensions (widgets, share extensions). No Android equivalent.
- **[Entitlements](https://developer.apple.com/documentation/bundleresources/entitlements)** — a signed list of capabilities (push notifications, iCloud, etc.) granted by Apple. Roughly equivalent to permissions, but granted at build time, not runtime.
- **[Provisioning profiles](https://developer.apple.com/documentation/xcode/distributing-your-app-to-registered-devices)** — certificates that prove "this build was made by this developer for these devices." Android uses keystore signing but without the provisioning complexity.
- **[Info.plist](https://developer.apple.com/documentation/bundleresources/information-property-list)** — XML config file declaring app capabilities, required permissions, URL schemes, etc. Roughly combines `AndroidManifest.xml` and some Gradle config.

## Further reading

- [Develop in Swift — developer.apple.com](https://developer.apple.com/tutorials/develop-in-swift/)
- [Articles about Android development — proandroiddev.com](https://proandroiddev.com)
- [Human Interface Guidelines — developer.apple.com](https://developer.apple.com/design/human-interface-guidelines/)
