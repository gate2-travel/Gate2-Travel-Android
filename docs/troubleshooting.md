# Troubleshooting

[Documentation home](../README.md) · [Getting started](getting-started.md) · [eSIM](esim.md) · [Flights](flights.md) · [Hotels](hotels.md) · [Showcase](showcase.md) · **Troubleshooting**

## Build time

| Symptom | Cause | Fix |
|:--|:--|:--|
| `Could not find travel.gate2:<module>-<company>:<version>` | That version (or that product) is not published for your company | Check Maven Central and ask Gate2 ([Installation](getting-started.md#installation)) |
| `check…AarMetadata` fails: "requires … to compile against version 35 or later" | `compileSdk` below 35 | Set `compileSdk = 35` or newer |
| `Duplicate class com.gate2.sdk...` | Two brand artifacts on the classpath | Depend on exactly one `<module>-<company>` |
| `EsimCallbacks` overload ambiguity | Un-annotated `onFail` lambda | Write `{ error: EsimError -> ... }` ([Callbacks](esim.md#callbacks)) |

## Runtime

| Symptom | Cause | Fix |
|:--|:--|:--|
| `IllegalStateException: SDK already initialized` | `initialize()` called twice | Guard with `isInitialized()` |
| `onFail` with `SDK_NOT_INITIALIZED` / "SDK is not initialized." | Product called before `initialize()` | Initialize in `Application.onCreate()` |
| `onFail` with `MODULE_ACCESS_DENIED` / "not available with your current subscription" | The SDK is uninitialized, or the product module was not found at runtime (modules register **by reflection** and a missing class fails silently). **Not** a token problem: reissuing the key will not help. | Confirm `initialize()` ran and the product artifact resolved. If it fails only in release builds, check that the AAR's consumer R8 rules are applied and not overridden. |
| `onFail` with `UNAUTHORIZED` (401) | Wrong or expired token, or a production token against staging (or vice versa) | Check the token and the artifact's `-staging` suffix |
| `onFail` with `INVALID_SESSION_ID` | Blank `sessionId` | Generate one per session; never pass `""` |
| Purchase never completes | `resumeAfterPayment()` not called, or a different `sessionId` | Persist **both** `orderId` and `sessionId` ([Payments](getting-started.md#payments)) |
| App exits after a product flow | Host called `finish()` in `onComplete` / `onCancel` | Remove it ([Back navigation](getting-started.md#back-navigation)) |
| eSIM entry shown but always errors | Device is not eSIM-capable | Gate on `isDeviceEsimCapable()` ([eSIM](esim.md#hide-the-feature-on-unsupported-devices)) |
| Staging shows no FAQ, banner or discounts | Content is configured per merchant and follows the token | Ask Gate2 to configure the merchant behind your staging token |

## Capturing SDK internals

During integration set `.logLevel(SdkLogLevel.ALL)` and supply
`.onLog { entry -> Log.d("Gate2", "${entry.level}: ${entry.message}") }`. Turn it back to `NONE`
for release builds. See [Logging](logging-and-security.md#logging).

## Reporting a problem

Email [support@gate2.travel](mailto:support@gate2.travel) with the SDK version
(`Gate2TravelSdk.getInstance().getVersion()`), the error code or message, and the `orderId` /
`sessionId` of any failed purchase.
