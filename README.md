# TrueOrigin Android SDK

Web-to-app install attribution for Android. The SDK reads Google Play's install referrer
on the first launch after install, TrueOrigin matches it to the ad click that led there,
and your app gets the campaign.

Requirements: Android 7.0 or later (minSdk 24), compileSdk 30 or later, Kotlin 1.9 or
later. For React Native and Expo, use the `@trueorigin/react-native` package instead.

## Install

The SDK is on Maven Central. In your app module's `build.gradle.kts`, with
`mavenCentral()` and `google()` among your repositories:

```kotlin
dependencies {
    implementation("dev.trueorigin:trueorigin-android:<version>")
}
```

`<version>` is the [latest release](https://github.com/trueorigin-dev/trueorigin-android/releases/latest)
here. The SDK brings Google's Install Referrer library and `kotlinx-coroutines-core` with it.

### Without Maven Central

Download `trueorigin.aar` and `trueorigin.aar.sha256` from the
[latest release](https://github.com/trueorigin-dev/trueorigin-android/releases/latest)
and check the AAR against its SHA-256:

```bash
shasum -a 256 -c trueorigin.aar.sha256
```

Put it in your app module's `libs/` folder and add it with the two libraries it uses,
since the bare AAR has no POM:

```kotlin
dependencies {
    implementation(files("libs/trueorigin.aar"))
    implementation("com.android.installreferrer:installreferrer:2.2")
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.8.1")
}
```

## Configure

Your SDK key is in the TrueOrigin dashboard, in Settings. Your app's Android URL there
must be its Google Play URL: the referrer Google Play passes on is what matches an
install to its tap.

```kotlin
import dev.trueorigin.TrueOrigin

class App : Application() {
    override fun onCreate() {
        super.onCreate()
        TrueOrigin.configure(this, apiKey = "to_…")
    }
}
```

## Read the attribution

```kotlin
val attribution = TrueOrigin.attribution()   // in a coroutine
when (attribution.status) {
    Attribution.Status.MATCHED -> showOnboarding(attribution.campaign)
    Attribution.Status.UNMATCHED -> {}   // organic, or installed without a tracking link
    else -> {}                           // keep an else branch: new statuses may come
}
```

`TrueOrigin.onAttribution { attribution -> … }` is the callback form, called once on the
main thread. Both return right away on later launches: the result is stored.
`TrueOrigin.currentAttribution` reads the stored result without waiting.

## RevenueCat

After configuring both SDKs, attach the install identity before purchases:

```kotlin
Purchases.sharedInstance.setAttributes(TrueOrigin.revenueCatAttributes())

// optional campaign attributes for targeting, once attribution resolves
TrueOrigin.onAttribution { attribution ->
    Purchases.sharedInstance.setAttributes(attribution.revenueCatAttributes)
}
```

Repeat the first call on every app start and after each successful RevenueCat
`logIn`/`logOut`. It returns right away and always includes `trueorigin_install_id`,
even while the attribution is pending. It does not change the RevenueCat App User ID.

## Privacy

The SDK does not read the advertising ID or `ANDROID_ID` and asks for no permission
beyond `INTERNET` (the install referrer library adds its own permission to bind to
Google Play). Once, on the first launch, it sends Google Play's install referrer, the
device model and manufacturer, Android version and API level, screen size and density,
the app's package name, version, version code and installer, the SDK version, a random
install id and the time of the first launch; TrueOrigin also sees the device's IP address
and the approximate location derived from it. The full list is in section 5.2 of the
[privacy policy](https://trueorigin.dev/privacy). The install id lives in the app's
no-backup storage and is removed with the app.

Questions: hello@trueorigin.dev

## License

Proprietary, see [LICENSE](LICENSE).
