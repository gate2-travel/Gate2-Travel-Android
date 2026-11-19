# Flights

[Documentation home](../README.md) · [Getting started](getting-started.md) · [eSIM](esim.md) · **Flights** · [Hotels](hotels.md) · [Showcase](showcase.md) · [Troubleshooting](troubleshooting.md)

Flight search, results, fare details, travelers, seat map, review, payment hand-off and
confirmation, plus "My flights", saved travelers and FAQ.

> **Artifact:** `travel.gate2:flights-<company>:1.2.4` (or `travel-all-<company>`), if Flights is in
> your agreement. Install and initialize the SDK first: [Getting started](getting-started.md).

- [Launch](#launch)
- [Callbacks](#callbacks)
- [Payment](#payment)
- [Cabinet entry points](#cabinet-entry-points)
- [What the user sees](#what-the-user-sees)

---

## Launch

```kotlin
import com.gate2.sdk.flights.Gate2Flight
import com.gate2.sdk.flights.FlightCallbacks

Gate2Flight.start(
    activity  = this,
    sessionId = sessionId,
    callbacks = flightCallbacks,
    userId    = currentUser.id,   // optional; required for saved travelers
    language  = "en",
    currency  = "AZN"             // ISO code used for pricing
)
```

| Parameter | Default | Notes |
|:--|:--|:--|
| `activity`, `sessionId`, `callbacks` | — | As for eSIM; see [Sessions and users](getting-started.md#sessions-and-users) |
| `userId: String?` | `null` | Sent as `X-User-Id` |
| `language: String` | `"en"` | Applied to the Activity locale and sent as `Accept-Language`. Built-in strings are English in this version. |
| `accentColor: Int` | brand colour | ARGB override for this launch |
| `currency: String` | `"AZN"` | Pricing currency |

---

## Callbacks

```kotlin
val flightCallbacks = FlightCallbacks(
    onComplete = { orderId -> /* booked and ticketed; orderId may be null */ },
    onCancel   = { /* user backed out. Do NOT finish() your own Activity here */ },
    onFail     = { message -> showError(message) },        // String
    onProcessPayment = { request -> /* optional, see Payment */ }
)
```

---

## Payment

`onProcessPayment` is optional. **When it is null the SDK confirms the booking immediately after
order creation.** When it is set, the SDK creates the order, hands you a `FlightPaymentRequest`
and finishes its Activity; charge the customer, then resume. Persist `orderId` and `sessionId`
first; see [Payments](getting-started.md#payments).

```kotlin
data class FlightPaymentRequest(
    val orderId: String,
    val amount: Double,
    val currency: String,
    val sessionId: String
)

Gate2Flight.resumeAfterPayment(
    activity  = this,
    orderId   = request.orderId,
    sessionId = request.sessionId,
    callbacks = FlightCallbacks(
        onComplete = { orderId -> /* ticketed */ },
        onFail     = { message -> /* confirmation failed */ }
    ),
    currency  = "AZN"
)
```

---

## Cabinet entry points

These open a single screen directly, with the same `FlightCallbacks`:

| Call | Opens | `userId` |
|:--|:--|:--|
| `Gate2Flight.startMyFlights(activity, sessionId, callbacks, userId, ...)` | Orders booked on this account (active and past); empty state can start a new booking | optional |
| `Gate2Flight.startTravelers(activity, sessionId, callbacks, userId, ...)` | Saved travelers and documents: list, view, add, edit | **required** |
| `Gate2Flight.startFaq(activity, sessionId, callbacks, userId, ...)` | Merchant FAQ topics and questions | optional |

---

## What the user sees

```
Welcome (T&C, first launch only) → Search → Results → Fare detail → Travelers
    → Pricing → Seat map (when the carrier provides one) → Review → [host payment] → Confirmation
```

Seat selection is shown on the review screen; in this version the seat choice is display-only and is
not yet carried on the order.

---

**Next:** [Hotels](hotels.md) · [Showcase](showcase.md) · [Troubleshooting](troubleshooting.md)
