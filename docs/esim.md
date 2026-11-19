# eSIM

[Documentation home](../README.md) · [Getting started](getting-started.md) · **eSIM** · [Flights](flights.md) · [Hotels](hotels.md) · [Showcase](showcase.md) · [Troubleshooting](troubleshooting.md)

Everything needed to ship the Gate2 eSIM flow inside your Android app: destination search, plan
filters, order, payment hand-off, eSIM installation, "My eSIMs" and top-ups.

> **Artifact:** `travel.gate2:esim-<company>:1.2.4` (staging: `esim-<company>-staging`).
> Install and initialize the SDK first: [Getting started](getting-started.md).

- [Launch the flow](#launch-the-flow)
- [Callbacks](#callbacks)
- [Payment](#payment)
- [Errors](#errors)
- ["My eSIMs"](#my-esims)
- [Hide the feature on unsupported devices](#hide-the-feature-on-unsupported-devices)
- [What the user sees](#what-the-user-sees)
- [Branding](#branding)
- [Pre-release checklist](#pre-release-checklist)

---

## Launch the flow

```kotlin
import com.gate2.sdk.esim.Gate2Esim
import com.gate2.sdk.esim.EsimCallbacks
import com.gate2.sdk.esim.api.EsimError

Gate2Esim.start(
    activity  = this,
    sessionId = sessionId,          // required, non-blank; one per purchase attempt (a UUID is fine)
    callbacks = esimCallbacks,
    userId    = currentUser.id,     // optional, strongly recommended
    language  = "en",               // "en", "az", "ru"
    title     = "eSIM"              // toolbar title
)
```

| Parameter | Default | Notes |
|:--|:--|:--|
| `activity: Activity` | — | Launch context |
| `sessionId: String` | — | Required, non-blank. Sent as `X-Session-Id`; keep the **same** value for `resumeAfterPayment()`. See [Sessions and users](getting-started.md#sessions-and-users). |
| `callbacks: EsimCallbacks` | — | See [Callbacks](#callbacks) |
| `userId: String?` | `null` | Sent as `X-User-Id`; scopes orders and "My eSIMs" |
| `language: String` | `"en"` | UI language: `en`, `az`, `ru` |
| `accentColor: Int` | your brand colour | ARGB override for this launch; already correct for your brand build |
| `fontResId: Int` | `0` | `@FontRes` override (0 = your brand font, or the bundled Inter) |
| `title: String` | `"eSIM"` | Toolbar title |
| `esimLabel: String?` | `null` | Best-effort display name for the installed subscription |

`start()` **does not throw.** If the SDK is uninitialized, the token has no eSIM entitlement, or
`sessionId` is blank, it calls `onFail` and returns. Always supply `onFail`.

---

## Callbacks

```kotlin
val esimCallbacks = EsimCallbacks(
    onComplete = {
        // Order completed and eSIM installed.
    },
    onCancel = {
        // User backed out. Do NOT finish() your own Activity here:
        // back already returns the user to your app.
    },
    onFail = { error: EsimError ->
        handleError(error)                       // see Errors
    },
    onProcessPayment = { request ->
        // Charge the customer, then resumeAfterPayment(); see Payment
    },
    onApiResponse = { statusCode, errorMessage ->
        // Optional: fires on every HTTP response. Useful for your own monitoring.
        analytics.track("gate2_http", statusCode, errorMessage)
    }
)
```

Every callback is optional, but ship `onFail` and `onProcessPayment` at minimum: without the latter
no purchase can complete.

> **Annotate the `onFail` lambda parameter.** `EsimCallbacks` is both a data class (whose `onFail`
> takes `EsimError`) and a same-named top-level function in the same package (whose `onFail` takes
> `String`). An un-annotated single-parameter lambda is ambiguous between the two, so write
> `{ error: EsimError -> … }`, or `{ message: String -> … }` if you want the string form:
>
> ```kotlin
> EsimCallbacks(onFail = { message: String -> toast(message) })
> ```

---

## Payment

The SDK creates the order, hands it to you in `onProcessPayment`, and waits for you to come back.
The model is the same for every product; read [Payments](getting-started.md#payments) once.

```
Gate2Esim.start(…)
    │
    │  user picks a plan, SDK creates the order
    ▼
onProcessPayment(EsimPaymentRequest)      ← your payment UI runs here
    │                                       SDK's Activity finishes
    │  you charge the customer
    ▼
Gate2Esim.resumeAfterPayment(orderId, sessionId, …)
    │
    ▼
onComplete()                              ← SDK confirms the order and installs the eSIM
```

### Store both `orderId` and `sessionId`

This is the single most common integration mistake. The SDK's Activity **finishes** while your
payment screens are in front, so the callback objects you passed to `start()` are gone by the time
payment returns. Persist both identifiers and pass them to a fresh `resumeAfterPayment()` call with
fresh callbacks.

```kotlin
onProcessPayment = { request ->
    // Persist durably: the user may be gone for minutes, and your process may die.
    paymentPrefs.edit()
        .putString("gate2_order_id", request.orderId)
        .putString("gate2_session_id", request.sessionId)
        .apply()

    startYourPaymentFlow(
        amount = request.amount,
        currency = request.currency,
        reference = request.orderId
    )
}
```

Then, when payment succeeds (from your deep link handler, activity result, or wherever your payment
provider returns):

```kotlin
Gate2Esim.resumeAfterPayment(
    activity  = this,
    orderId   = paymentPrefs.getString("gate2_order_id", null) ?: return,
    sessionId = paymentPrefs.getString("gate2_session_id", null) ?: return,   // same value as start()
    callbacks = EsimCallbacks(
        onComplete = { clearPaymentState(); showSuccess() },
        onCancel   = { clearPaymentState() },
        onFail     = { error: EsimError -> clearPaymentState(); handleError(error) }
    ),
    language  = "en"
)
```

If payment **fails or is abandoned**, do not call `resumeAfterPayment()`; just clear your stored
state. The order expires on the Gate2 side.

### `EsimPaymentRequest`

```kotlin
data class EsimPaymentRequest(
    val orderId: String,        // persist this
    val sessionId: String,      // persist this
    val userId: String,
    val productId: String,
    val productName: String,
    val amount: Double,
    val currency: String,       // ISO 4217, e.g. "AZN"
    val dataLimit: String,      // e.g. "5GB", "Unlimited"
    val validityDays: Int,
    val coverage: List<String>, // covered countries/regions
    val isTopUp: Boolean = false,
    val iccid: String? = null   // set for top-up orders
)
```

`isTopUp` is `true` when the customer is topping up an existing eSIM rather than buying a new one.
The charge path is identical; use it for your own receipt wording if you want to distinguish them.

---

## Errors

`EsimError` carries structured detail rather than just a string:

```kotlin
data class EsimError(
    val code: EsimErrorCode,
    val message: String,                    // display-ready
    val cause: Throwable? = null,
    val isRetryable: Boolean = false,
    val httpStatusCode: Int? = null         // set for HTTP errors
)
```

```kotlin
private fun handleError(error: EsimError) {
    when (error.code) {
        EsimErrorCode.NETWORK_ERROR ->
            if (error.isRetryable) showRetryDialog() else showOffline()

        EsimErrorCode.UNAUTHORIZED,
        EsimErrorCode.FORBIDDEN ->
            reportToBackend(error)          // bad or expired API key: not user-fixable

        EsimErrorCode.MODULE_ACCESS_DENIED ->
            reportToBackend(error)          // build/init problem, not an entitlement one;
                                            // see Troubleshooting

        EsimErrorCode.DEVICE_NOT_SUPPORTED,
        EsimErrorCode.ESIM_EUICC_DISABLED ->
            showDeviceUnsupported()

        EsimErrorCode.PAYMENT_FAILED ->
            returnToPaymentRetry()

        else -> showGenericError(error.message)
    }
    analytics.logError(error.code.name, error.httpStatusCode)
}
```

| Group | `EsimErrorCode` values |
|:--|:--|
| SDK state | `SDK_NOT_INITIALIZED`, `MODULE_ACCESS_DENIED`, `DEVICE_NOT_SUPPORTED` |
| Validation | `INVALID_SESSION_ID`, `INVALID_ORDER_ID`, `INVALID_USER_ID`, `VALIDATION_ERROR` |
| Network | `NETWORK_ERROR` |
| HTTP | `UNAUTHORIZED`, `FORBIDDEN`, `NOT_FOUND`, `RATE_LIMITED`, `SERVER_ERROR` |
| Order | `NO_PRODUCT_SELECTED`, `ORDER_CREATION_FAILED`, `PAYMENT_FAILED`, `ORDER_CONFIRMATION_FAILED` |
| Installation | `ESIM_INSTALLATION_FAILED`, `ESIM_EUICC_DISABLED` |
| Other | `PARSE_ERROR`, `UNKNOWN` |

An `EsimException` wrapper is also available if you prefer to surface these through an
exception-based error pipeline.

---

## "My eSIMs"

Open the customer's purchased eSIMs directly, bypassing the catalogue:

```kotlin
Gate2Esim.showMyEsims(
    activity  = this,
    sessionId = sessionId,
    callbacks = esimCallbacks,
    userId    = currentUser.id,   // REQUIRED here: the list is scoped to a user
    language  = "en"
)
```

Unlike `start()`, `userId` is **required and must be non-blank** here. A blank value invokes
`onFail` and returns.

---

## Hide the feature on unsupported devices

Run both checks before showing an eSIM entry point:

```kotlin
// Hardware/OS: is this device eSIM-capable at all?
// Returns false on API < 28 (EuiccManager does not exist) and where eUICC is disabled.
val deviceOk = Gate2Esim.isDeviceEsimCapable(context)

// SDK state: is the SDK initialized, and is the eSIM module on the classpath?
val moduleOk = Gate2Esim.isAvailable()

if (deviceOk && moduleOk) showEsimEntryPoint()
```

`isAvailable()` never throws; it returns `false` if the SDK is uninitialized. It reports SDK
**state**, not entitlement: once `initialize()` has run and `esim-<company>` is on the classpath, it
is `true`. Entitlement is enforced by the backend, so an unentitled token surfaces as an HTTP error
during the flow rather than here.

An unsupported device means API < 28, no eUICC hardware, or an eUICC that the carrier/OEM has
disabled. Users on those devices should not see the feature at all rather than get an error after
tapping.

---

## What the user sees

```
Welcome (T&C, first launch only) → Search destination → Plans → Order → [host payment] → Confirmation → Install
                                                                                         ↘ My eSIMs → Detail → Top-up
```

- **Plan filters.** The plan list has a filters sheet: plan type (total data or daily data),
  region, country, data volume and duration, with a live count of matching plans on the
  "Show plans" button.
- **After purchase.** The confirmation screen shows the activation code, SM-DP+ address and
  installation steps, followed by a short checklist: remove old eSIM profiles, make sure the eSIM
  line is turned on, and enable data roaming for it.
- **FAQ and banner.** The search screen shows your FAQ and, when one is configured, a promotional
  banner. The banner is hidden silently when none is configured.
- **Merchant content.** The plan catalogue, discounts, FAQ and banner are set up per merchant on
  the Gate2 backend and follow the access token, so a staging token can show different content
  from production.

---

## Branding

Your brand build already carries your branding: the accent colour, a colour palette for light and
dark mode, the welcome screen and brand assets are compiled into `esim-<company>`. The screens
follow the device's light/dark setting. In the normal case you pass no theme at all.

```kotlin
// Per-launch overrides, only if you need them:
Gate2Esim.start(
    activity    = this,
    sessionId   = sessionId,
    callbacks   = esimCallbacks,
    accentColor = ContextCompat.getColor(this, R.color.brand_accent),
    fontResId   = R.font.brand_sans          // 0 = your brand font, or the bundled Inter
)
```

App-wide theming and presets: [Theming and branding](theming.md).

---

## Pre-release checklist

- [ ] `travel.gate2:esim-<company>:<version>` is the only Gate2 dependency; `core` is not declared separately
- [ ] `compileSdk` is 35 or newer
- [ ] Access token injected from a secret store, not committed
- [ ] `logLevel` is `NONE` in release builds
- [ ] `initialize()` guarded by `isInitialized()`
- [ ] eSIM entry point gated on `isDeviceEsimCapable()` **and** `isAvailable()`
- [ ] `onFail` implemented and mapped to real UI for every `EsimErrorCode` group
- [ ] Both `orderId` **and** `sessionId` persisted durably in `onProcessPayment`
- [ ] `resumeAfterPayment()` called on payment success; state cleared on failure
- [ ] Nothing calls `finish()` inside `onComplete` / `onCancel`
- [ ] Tested on a real API 28+ eSIM-capable handset (emulators have no eUICC)
- [ ] Tested payment resume after the app is killed mid-payment
- [ ] The merged permissions have cleared security review ([list](getting-started.md#what-the-sdk-adds-to-your-manifest))

---

**Next:** [Theming](theming.md) · [Troubleshooting](troubleshooting.md) · [API reference](api-reference.md)
