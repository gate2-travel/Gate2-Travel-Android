<div align="center">

# Gate2 Travel SDK for Android

### eSIM data plans, flight booking and hotel booking inside your own app

[![SDK Version](https://img.shields.io/badge/SDK-1.2.3-blue.svg)](https://central.sonatype.com/search?q=travel.gate2)
[![Platform](https://img.shields.io/badge/Platform-Android-green.svg)](https://developer.android.com)
[![Min SDK](https://img.shields.io/badge/minSdk-24-orange.svg)](https://developer.android.com)
[![Kotlin](https://img.shields.io/badge/Kotlin-2.2.21-purple.svg)](https://kotlinlang.org)
[![Maven Central](https://img.shields.io/badge/Maven%20Central-travel.gate2-teal.svg)](https://central.sonatype.com/search?q=travel.gate2)

**One dependency. One `initialize()`. One call per product.**

</div>

---

## Products

Each product renders its own Jetpack Compose UI in its own `Activity`, talks to the Gate2 backend
over a pinned mTLS connection, and hands control back to your app through callbacks. Your app never
handles card data: payment goes through your own provider and the SDK resumes afterwards.

| Product | Artifact | Entry point | What your users get | Guide |
|:--|:--|:--|:--|:--|
| **eSIM** | `esim-<company>` | `Gate2Esim` | Destination search, plan filters, order, eSIM installation, "My eSIMs", top-ups | [eSIM](docs/esim.md) |
| **Flights** | `flights-<company>` | `Gate2Flight` | Search, results, fare details, travelers, seat map, review, confirmation, "My flights", saved travelers, FAQ | [Flights](docs/flights.md) |
| **Hotels** | `hotels-<company>` | `Gate2Hotel` | Search, results, hotel detail, room pre-booking, guest details, confirmation | [Hotels](docs/hotels.md) |
| **All three** | `travel-all-<company>` | the three above + `Gate2TravelShowcase` | Everything, plus a customizable product picker with a "My cabinet" area | [Showcase](docs/showcase.md) |

`<company>` is the identifier Gate2 assigned to your build; your colours, welcome screens and fonts
are compiled into it. Gate2 publishes for your company only the products in your agreement: for an
eSIM-only integration that is `esim-<company>`.

---

## Quick start

**1. Add the dependency** (Maven Central, `compileSdk` 35+):

```kotlin
// app/build.gradle.kts
dependencies {
    implementation("travel.gate2:esim-<company>:1.2.3")   // -staging suffix for the staging backend
}
```

**2. Initialize once**, in `Application.onCreate()`:

```kotlin
if (!Gate2TravelSdk.isInitialized()) {
    Gate2TravelSdk.initialize(
        Gate2SdkConfig.builder()
            .context(this)
            .apiKey(BuildConfig.GATE2_ACCESS_TOKEN)   // issued by Gate2; never commit it
            .build()
    )
}
```

**3. Launch a product**, and hand payment back to the SDK when your provider succeeds:

```kotlin
Gate2Esim.start(
    activity  = this,
    sessionId = sessionId,          // one per purchase attempt, e.g. a UUID
    callbacks = EsimCallbacks(
        onComplete       = { /* eSIM installed */ },
        onFail           = { error: EsimError -> showError(error.message) },
        onProcessPayment = { request -> /* save orderId + sessionId, charge, then resumeAfterPayment() */ }
    ),
    userId    = currentUser.id
)
```

The payment hand-off is the one part to get right: read
[Payments](docs/getting-started.md#payments) before you ship.

---

## Documentation

| Guide | What's inside |
|:--|:--|
| **[Getting started](docs/getting-started.md)** | Requirements, installation, initialization, sessions, payments, back navigation |
| **[eSIM](docs/esim.md)** | Launch, callbacks, payment, errors, "My eSIMs", device checks, pre-release checklist |
| **[Flights](docs/flights.md)** | Launch, callbacks, payment, "My flights", saved travelers, FAQ |
| **[Hotels](docs/hotels.md)** | Launch, callbacks, booking flow |
| **[Showcase](docs/showcase.md)** | The `travel-all` product picker and "My cabinet" |
| **[Theming and branding](docs/theming.md)** | Brand builds, per-launch overrides, `Gate2Theme` |
| **[Logging, network and security](docs/logging-and-security.md)** | Log levels, hosts, headers, mTLS |
| **[Troubleshooting](docs/troubleshooting.md)** | Build and runtime errors, what to send support |
| **[API reference](docs/api-reference.md)** | Every public symbol, by package |
| **[Migrating from 1.0.x](docs/migration-from-1.0.md)** | From the retired `travel.gate2:esim:1.0.x` line |
| **[Changelog](CHANGELOG.md)** | What changed in each version |

---

## Support

Email [support@gate2.travel](mailto:support@gate2.travel). Include the SDK version
(`Gate2TravelSdk.getInstance().getVersion()`), the error code or message, and the `orderId` /
`sessionId` of any failed purchase.

<div align="center">

**Gate2 Travel SDK** · Android · © 2026 Gate2 Travel. All rights reserved.

</div>
