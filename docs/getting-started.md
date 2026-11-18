# Getting started

[Documentation home](../README.md) · **Getting started** · [eSIM](esim.md) · [Flights](flights.md) · [Hotels](hotels.md) · [Showcase](showcase.md) · [Troubleshooting](troubleshooting.md)

Everything every product needs: requirements, installation, initialization, and the few concepts
all products share. Read this first, then the guide for your product.

- [Requirements](#requirements)
- [Installation](#installation)
- [Initialize the SDK](#initialize-the-sdk)
- [Concepts shared by every product](#concepts-shared-by-every-product)

---

## Requirements

| | Requirement | Notes |
|:--|:--|:--|
| **minSdk** | 24 | The library's declared floor. Gate2 supports Android 8.1 (API 27) and newer. |
| **eSIM provisioning** | API 28+ | `EuiccManager` does not exist below API 28. Gate the eSIM entry point with `Gate2Esim.isDeviceEsimCapable()`. |
| **compileSdk** | 35 or newer | Enforced since 1.2.2: with a lower `compileSdk` the build fails in `check…AarMetadata`. |
| **Kotlin** | 2.2.21 | |
| **JVM target** | 11 | |
| **Jetpack Compose** | Not required | Each product renders inside its own `Activity`. You need Compose in your app only if you embed `Gate2TravelShowcase`. |
| **Google Play Services** | Not required | |
| **Access token** | Issued by Gate2 | A JWT sent as `X-API-Key`. Product entitlement is enforced server-side. |

### What the SDK adds to your manifest

These are merged automatically. They are listed here because they usually need to clear a
security review.

| Permission | Declared by | Why |
|:--|:--|:--|
| `INTERNET` | all modules | Backend calls |
| `READ_PHONE_STATE` / `READ_BASIC_PHONE_STATE` | eSIM | Detect eUICC capability and existing subscriptions. The eSIM Activity requests this at runtime. |
| `WRITE_SETTINGS` | eSIM | Required by the system eSIM activation flow |
| `USE_CREDENTIALS` | eSIM | eUICC subscription provisioning |

Each product declares one non-exported `Activity` (`EsimActivity`, `FlightActivity`,
`HotelActivity`). No launcher entries, services, receivers or content providers.

---

## Installation

Artifacts are published to **Maven Central** under the group `travel.gate2`. Brand and backend
environment are part of the **artifact ID**, so the version is always plain semver:

```
travel.gate2:<module>-<company>[-staging]:<version>
```

| Part | Values | Meaning |
|:--|:--|:--|
| `<module>` | `esim`, `flights`, `hotels`, `travel-all` | What you integrate (`core` is transitive) |
| `<company>` | assigned by Gate2 (one short lowercase word) | Your brand build: colours, welcome screens, fonts and the backend endpoint are compiled in |
| `-staging` | optional suffix | Points at the staging backend (`api.gate2.travel`). Omit it for production (`gw.gate2.travel`). |
| `<version>` | e.g. `1.2.3` | Plain semver. **Never** pin `1.2.3-<company>` or `1.0.20-test`; that scheme is retired. |

```kotlin
// app/build.gradle.kts
dependencies {
    // One product. Each pulls its matching core automatically:
    implementation("travel.gate2:esim-<company>:1.2.3")
    // implementation("travel.gate2:flights-<company>:1.2.3")
    // implementation("travel.gate2:hotels-<company>:1.2.3")

    // ...or, if your agreement covers all three, everything (eSIM + Flights + Hotels + showcase)
    // over one shared core, instead of the lines above:
    // implementation("travel.gate2:travel-all-<company>:1.2.3")
}
```

```kotlin
// settings.gradle.kts
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
    }
}
```

All products share one `core` artifact (`core-<company>`) that owns initialization, networking,
theming and logging. It arrives transitively: **never declare `core` yourself**; pinning it
separately can desynchronise the two.

> **Which products and versions you get depends on your agreement with Gate2.** Gate2 publishes,
> for your `<company>`, only the products you use, and releases each company's build on its own
> schedule. Before pinning a version, check that
> `https://repo1.maven.org/maven2/travel/gate2/<module>-<company>/<version>/` exists (and the
> `-staging` one for QA), and ask Gate2 if it is missing. Staging builds carry the same version
> number as production.
>
> **Exactly one `<company>` per app.** Two brand artifacts on the classpath produce
> `Duplicate class com.gate2.sdk...` at build time.

### ProGuard / R8

Nothing to add. The AARs ship unminified with their own `consumer-rules.pro`; your app's R8 pass
obfuscates the SDK together with your own code. Do not add manual `-keep class com.gate2.sdk.**`
rules, they only reduce your own obfuscation coverage.

---

## Initialize the SDK

Call this once, before the first product call. `Application.onCreate()` is the natural place.

```kotlin
import com.gate2.sdk.core.Gate2TravelSdk
import com.gate2.sdk.core.config.Gate2SdkConfig
import com.gate2.sdk.core.config.SdkLogLevel

class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()

        if (Gate2TravelSdk.isInitialized()) return   // initialize() throws if called twice

        Gate2TravelSdk.initialize(
            Gate2SdkConfig.builder()
                .context(this)
                .apiKey(BuildConfig.GATE2_ACCESS_TOKEN)   // inject from a secret store, never commit it
                .logLevel(if (BuildConfig.DEBUG) SdkLogLevel.DEBUG else SdkLogLevel.NONE)
                .build()
        )
    }
}
```

Keep the access token out of source control: inject it via `BuildConfig` from a Gradle property or
your CI secret store.

### `Gate2SdkConfig.Builder`

| Method | Required | Notes |
|:--|:--:|:--|
| `.context(Context)` | yes | Stored as `applicationContext` |
| `.apiKey(String)` | yes | Your Gate2 token. `build()` throws `IllegalArgumentException` if blank. |
| `.logLevel(SdkLogLevel)` | no | `ALL`, `DEBUG`, `INFO`, `WARN`, `ERROR`, `FATAL`, `NONE` (default `NONE`) |
| `.enableLogging(Boolean)` | no | Shorthand: `true` is `DEBUG`, `false` is `NONE` |
| `.onLog { entry -> }` | no | Receive every `SdkLogEntry`; see [Logging](logging-and-security.md#logging) |
| `.theme(Gate2Theme)` | no | App-wide theme override; see [Theming](theming.md) |
| `.clientCertificate(...)` | deprecated | mTLS is embedded in the SDK. Remove any call. |

### What can and cannot fail here

- `initialize()` throws `IllegalStateException` **only** when the SDK is already initialized. Guard
  with `isInitialized()`.
- `Gate2SdkConfig.builder().build()` throws `IllegalArgumentException` only for *missing* values:
  a blank `apiKey`, or a `context` / `apiKey` never set.
- The token is **not** validated on the client. A wrong or expired token initializes cleanly and
  surfaces later as an `UNAUTHORIZED` (HTTP 401) failure in the product callbacks. Handle it there,
  not with a `try/catch` around `initialize()`.
- `Gate2TravelSdk.getInstance().getVersion()` returns the running SDK version (staging builds report
  `1.2.3-test`). Include it in support requests.

---

## Concepts shared by every product

```
┌──────────────────────────────────────────────────────────────────────────┐
│                              YOUR APPLICATION                            │
│                                                                          │
│  Application.onCreate()            Anywhere in your UI                   │
│  ┌──────────────────────┐          ┌────────────────────────────────┐    │
│  │ Gate2TravelSdk       │          │ Gate2Esim.start(...)           │    │
│  │   .initialize(config)│  once    │ Gate2Flight.start(...)         │    │
│  └──────────────────────┘          │ Gate2Hotel.start(...)          │    │
│                                    │ Gate2TravelShowcase { ... }    │    │
│                                    └──────────────┬─────────────────┘    │
│                                                   │ launches             │
│                                    ┌──────────────▼─────────────────┐    │
│  ┌──────────────────────┐          │   SDK Activity (own UI)        │    │
│  │ Your callbacks       │◀─────────│   • full product flow          │    │
│  │  onComplete          │          │   • creates the order          │    │
│  │  onCancel            │          └──────────────┬─────────────────┘    │
│  │  onFail              │                         │ onProcessPayment     │
│  │  onProcessPayment ───┼──▶ your payment UI ──▶ resumeAfterPayment()   │
│  └──────────────────────┘                                                │
└──────────────────────────────────────────────────────────────────────────┘
```

### Who does what

| SDK | Host app |
|:--|:--|
| All product screens, state and validation | Call `initialize()` once with the Gate2 access token |
| Backend calls, mTLS, certificate pinning, retries | Provide a `sessionId` per flow and (ideally) a `userId` |
| Order creation and confirmation | Charge the customer with its own payment provider |
| eSIM provisioning on the device | Persist `orderId` + `sessionId` and call `resumeAfterPayment()` |
| Brand look (compiled into the brand artifact) | Decide where the product entry points live |

### Sessions and users

- **`sessionId`** is required and must not be blank. Generate one per purchase attempt (a UUID is
  fine) and keep it for as long as that attempt is alive: you pass the **same** value again after
  payment. It is sent as `X-Session-Id` so Gate2 can correlate a purchase across the payment hop.
  Persist it across process death if the payment journey leaves your app.
- **`userId`** is optional but strongly recommended. It is sent as `X-User-Id` and scopes orders,
  "My eSIMs", "My flights" and saved travelers. Some entry points require it; each guide says which.

### Payments

Gate2 never takes payment. For eSIM and Flights the SDK creates the order, hands it to you and
waits:

```
Gate2Esim.start(...)  /  Gate2Flight.start(...)
    │
    │   user completes the flow, SDK creates the order
    ▼
onProcessPayment(request)        ← your payment UI runs here; the SDK Activity has FINISHED
    │
    │   you charge the customer with your provider
    ▼
Gate2Esim.resumeAfterPayment(orderId, sessionId, freshCallbacks)
Gate2Flight.resumeAfterPayment(orderId, sessionId, freshCallbacks)
    │
    ▼
onComplete()                     ← SDK confirms the order (and installs the eSIM / issues tickets)
```

**Persist both `orderId` and `sessionId` durably inside `onProcessPayment`**, and pass them to a
fresh `resumeAfterPayment()` call with fresh callbacks. The SDK's Activity finishes while your
payment screens are in front, so the callback objects you passed to `start()` are gone by the
time payment returns, and your process may have been killed in between. This rule prevents most
support tickets.

If payment **fails or is abandoned**, do not call `resumeAfterPayment()`; just clear your stored
state. The order expires on the Gate2 side.

> `Gate2TravelSdk.notifyPaymentResult()` is **deprecated** precisely because of the stale-callback
> problem above. Use `resumeAfterPayment()`.

Product-specific code: [eSIM payment](esim.md#payment), [Flights payment](flights.md#payment).
Hotels has no external payment hook in this version.

### Back navigation

Every product launches as its own `Activity` on top of yours, in the same task, with no special
launch flags. Inside the flow, back steps through the SDK's screens; at the root it calls
`onCancel` (or `onComplete` on a confirmation screen) and finishes the SDK Activity, which
returns the user to your screen automatically.

**Do not call `finish()` on your own Activity inside `onComplete` / `onCancel`.** That is what
makes a host app appear to exit after a product flow. If you present the showcase as its own
destination, give it `onBack` so the system back button lands where you launched it from.

### Language

Every entry point takes a `language` (`"en"` by default). The eSIM module ships `en`, `az` and
`ru` strings; Flights and Hotels apply the locale and send it as `Accept-Language`, but their
built-in strings are English in this version.

---

**Next:** the guide for your product: [eSIM](esim.md) · [Flights](flights.md) · [Hotels](hotels.md) · [Showcase](showcase.md)
