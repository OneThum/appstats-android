# AppStats Android SDK

[![Build](https://github.com/OneThum/AppStats-Android/actions/workflows/build.yml/badge.svg)](https://github.com/OneThum/AppStats-Android/actions/workflows/build.yml)
[![Release](https://jitpack.io/v/OneThum/AppStats-Android.svg)](https://jitpack.io/#OneThum/AppStats-Android)

Privacy-first analytics SDK for Android apps. Native peer to the
[AppStats Swift SDK](https://github.com/OneThum/AppStats-iOS), conforming to the
same [SDK Protocol Specification](../docs/SDK_PROTOCOL.md).

- Auto-tracks: app lifecycle, screens, crashes
- Custom events with primitive properties
- Offline-resilient: in-memory + on-disk queue, deflate-compressed batches
- Tiny dependency footprint: OkHttp, kotlinx.serialization, AndroidX lifecycle

---

## Requirements

| Item | Version |
|---|---|
| `minSdk` | 24 (Android 7.0) |
| Kotlin | 1.9+ |
| Java target | 17 |

## Installation

### From JitPack (current)

```kotlin
// settings.gradle.kts
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        maven("https://jitpack.io")
    }
}

// app/build.gradle.kts
dependencies {
    implementation("com.github.OneThum:AppStats-Android:0.1.1")
}
```

### Repository & JitPack artifact id (May 2026)

The public GitHub repository is **`OneThum/AppStats-Android`** (older links to `OneThum/appstats-android` redirect).  
On JitPack the module id matches the repo name: use **`com.github.OneThum:AppStats-Android`** — replace any legacy `com.github.OneThum:appstats-android` lines.

### From Maven Central (post-Phase 9)

After the Sonatype Central Portal namespace `com.onethumsoftware` is verified and CI secrets are configured (see checklist below):

```kotlin
dependencies {
    implementation("com.onethumsoftware:appstats-android:1.0.0")
}
```

#### Maintainer checklist — Maven Central graduation

1. **Namespace**: In [Central Portal](https://central.sonatype.com/), claim `com.onethumsoftware` (DNS TXT verification as documented by Sonatype).
2. **Signing**: Create a dedicated GPG key for artifacts; publish the public key; store private key + passphrase in GitHub Actions secrets for the **AppStats-Android** repo (names depend on `release.yml`; typically along the lines of `SIGNING_KEY`, `SIGNING_PASSWORD`).
3. **Publishing**: The Android repo uses the Vanniktech Maven Publish plugin with `RELEASE_SIGNING_ENABLED=true` only in the release workflow (JitPack builds leave signing off).
4. **Release**: Tag `v1.0.0` on **OneThum/AppStats-Android**, run the release workflow, confirm staging → release on Central.
5. **Consumers**: Update apps from JitPack coordinates to `com.onethumsoftware:appstats-android:1.0.0` (or newer).

Until these steps are complete, stay on **JitPack** coordinates above.

## Quick start

### Manual configuration

```kotlin
import com.onethumsoftware.appstats.AppStats

class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        AppStats.configure(
            context = this,
            apiKey = BuildConfig.APPSTATS_API_KEY,
        )
    }
}
```

### Manifest-driven auto-configuration

```xml
<!-- AndroidManifest.xml -->
<application>
    <meta-data
        android:name="com.onethumsoftware.appstats.API_KEY"
        android:value="as_live_xxxxxxxxxxxx" />
    <meta-data
        android:name="com.onethumsoftware.appstats.AUTO_TRACK_SCREENS"
        android:value="true" />
</application>
```

The SDK initializes via `androidx.startup` on the first content provider
boot, with no explicit `Application.onCreate` call required.

## API surface

```kotlin
AppStats.configure(context, apiKey, autoTrackScreens = true, flushInterval = 30.seconds)
AppStats.track("purchase_completed", mapOf("amount" to 9.99, "currency" to "USD"))
AppStats.trackScreen("HomeView")          // for Compose / non-Activity navigation
AppStats.flush()                          // fire-and-forget
suspend fun shutdown() = AppStats.flushAsync()
AppStats.setUserProperty("plan", "pro")
```

## Architecture

The SDK mirrors the Swift SDK module-for-module so cross-platform analytics
behave identically. See [docs/SDK_PROTOCOL.md](../docs/SDK_PROTOCOL.md) for the
wire-protocol contract both SDKs implement.

| Layer | Responsibility |
|---|---|
| `AppStats` | Public façade. Singleton with internal coroutine scope. |
| `EventCollector` | In-memory queue (cap 500), batches at 20, sends up to 100/request. |
| `NetworkManager` | OkHttp + zlib deflate, exponential-backoff retries, circuit breaker. |
| `StorageManager` | Atomic JSON file at `filesDir/appstats/events.json`, 10 MB budget. |
| `ScreenTracker` | `Application.ActivityLifecycleCallbacks`-based auto-tracking. |
| `CrashReporter` | `Thread.setDefaultUncaughtExceptionHandler` writes a marker, sends on next launch. |
| `Lifecycle observer` | `ProcessLifecycleOwner` to emit `session_start` / `session_end`. |
| `BackgroundFlushWorker` | Expedited `WorkManager` job to finish flushing after backgrounding. |

## Privacy

The SDK collects only what is documented in the protocol spec. No advertising
identifiers, no contacts, no precise location. IP-based geolocation is performed
server-side and uses only country/city granularity.

## Versioning

The Android SDK uses independent semver from the Swift SDK. Both conform to the
same wire-protocol version (`/v1/ingest`).

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

[MIT](LICENSE)
