# Migrating from 1.0.x

[Documentation home](../README.md) · [Getting started](getting-started.md) · [eSIM](esim.md) · [Troubleshooting](troubleshooting.md)

The 1.0.x line (`travel.gate2:esim:1.0.x`, `travel.gate2:core:1.0.0`, and the pre-release
`com.gate2.sdk:*` coordinates in the January 2026 docs) is retired and receives no updates.

| Then | Now |
|:--|:--|
| `travel.gate2:esim:1.0.x` | `travel.gate2:esim-<company>:1.2.3` (or `travel-all-<company>`) |
| A separate `core` dependency | Transitive. Remove it. |
| Version suffixes `1.0.20-test`, `1.1.4-<company>` | `-staging` suffix on the **artifact ID**; plain semver version |
| `Gate2EsimSdk` / `Gate2HotelSdk` | One `Gate2TravelSdk.initialize()` for every product |
| `Gate2TravelScreen` / `Gate2EsimScreen` / `Gate2HotelScreen` composables wrapped in `Gate2TravelTheme` | `Gate2Flight.start()` / `Gate2Esim.start()` / `Gate2Hotel.start()`. The screens are internal; each product runs in its own Activity. |
| `PaymentRequest` + `PaymentResult` + `notifyPaymentResult()` | `EsimPaymentRequest` / `FlightPaymentRequest` + `resumeAfterPayment(orderId, sessionId)` |
| `onFail: (String) -> Unit` for eSIM | `onFail: (EsimError) -> Unit` (the `String` form still compiles via the `EsimCallbacks(...)` function) |
| `.clientCertificate(pkcs12, password)` | Remove; mTLS is embedded |

Then follow [Getting started](getting-started.md) and the [eSIM guide](esim.md).
