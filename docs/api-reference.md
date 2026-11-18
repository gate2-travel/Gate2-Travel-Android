# API reference

[Documentation home](../README.md) · [Getting started](getting-started.md) · [eSIM](esim.md) · [Flights](flights.md) · [Hotels](hotels.md) · [Showcase](showcase.md) · [Troubleshooting](troubleshooting.md)

Every public symbol a host app uses, by package. The Maven group is `travel.gate2`; the Kotlin
packages are `com.gate2.sdk.*`. That is expected.

| Symbol | Package | Guide |
|:--|:--|:--|
| `Gate2TravelSdk.initialize` / `.isInitialized` / `.updateTheme` / `.getInstance().getVersion()` | `com.gate2.sdk.core` | [Getting started](getting-started.md#initialize-the-sdk) |
| `Gate2SdkConfig`, `SdkLogLevel`, `SdkLogEntry` | `com.gate2.sdk.core.config` | [Getting started](getting-started.md#gate2sdkconfigbuilder), [Logging](logging-and-security.md#logging) |
| `Gate2Theme` and its colour / shape / spacing / typography / component configs | `com.gate2.sdk.core.config` | [Theming](theming.md) |
| `Gate2Esim.start` / `.resumeAfterPayment` / `.showMyEsims` / `.isAvailable` / `.isDeviceEsimCapable`, `EsimCallbacks` | `com.gate2.sdk.esim` | [eSIM](esim.md) |
| `EsimPaymentRequest`, `EsimError`, `EsimErrorCode`, `EsimException` | `com.gate2.sdk.esim.api` | [eSIM](esim.md#errors) |
| `Gate2Flight.start` / `.startMyFlights` / `.startTravelers` / `.startFaq` / `.resumeAfterPayment`, `FlightCallbacks` | `com.gate2.sdk.flights` | [Flights](flights.md) |
| `FlightPaymentRequest` | `com.gate2.sdk.flights.api` | [Flights](flights.md#payment) |
| `Gate2Hotel.start`, `HotelCallbacks` | `com.gate2.sdk.hotels` | [Hotels](hotels.md) |
| `Gate2TravelShowcase`, `Gate2ShowcaseConfig`, `Gate2ShowcaseProduct`, `Gate2CabinetConfig`, `Gate2Product` | `com.gate2.sdk.travel` (travel-all only) | [Showcase](showcase.md) |

Deprecated: `Gate2TravelSdk.notifyPaymentResult()` (use `resumeAfterPayment()`) and
`Gate2SdkConfig.Builder.clientCertificate(...)` (mTLS is embedded).
