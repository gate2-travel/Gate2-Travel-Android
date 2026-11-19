# Hotels

[Documentation home](../README.md) · [Getting started](getting-started.md) · [eSIM](esim.md) · [Flights](flights.md) · **Hotels** · [Showcase](showcase.md) · [Troubleshooting](troubleshooting.md)

Hotel search, results, hotel detail, room pre-booking, guest details and confirmation.

> **Artifact:** `travel.gate2:hotels-<company>:1.2.4` (or `travel-all-<company>`), if Hotels is in
> your agreement. Install and initialize the SDK first: [Getting started](getting-started.md).

---

## Launch

```kotlin
import com.gate2.sdk.hotels.Gate2Hotel
import com.gate2.sdk.hotels.HotelCallbacks

Gate2Hotel.start(
    activity         = this,
    sessionId        = sessionId,
    callbacks        = HotelCallbacks(
        onComplete = { confirmationNumber -> /* booked; may be null */ },
        onCancel   = { /* user backed out. Do NOT finish() your own Activity here */ },
        onFail     = { message -> showError(message) }
    ),
    userId           = currentUser.id,
    language         = "en",
    guestNationality = "AZ"      // ISO country code, default "AZ"
)
```

| Parameter | Default | Notes |
|:--|:--|:--|
| `activity`, `sessionId`, `callbacks` | — | As for eSIM; see [Sessions and users](getting-started.md#sessions-and-users) |
| `userId: String?` | `null` | Sent as `X-User-Id` |
| `language: String` | `"en"` | Activity locale + `Accept-Language`; built-in strings are English in this version |
| `accentColor: Int` | brand colour | ARGB override for this launch |
| `guestNationality: String` | `"AZ"` | Nationality sent with searches and bookings |

---

## Payment

There is **no external payment hook for hotels in this version**: `HotelCallbacks` has no
`onProcessPayment`, and the SDK confirms the booking directly from the guest-details screen.

---

## What the user sees

```
Welcome (T&C, first launch only) → Search → Results (sort, refundable filter) → Hotel detail
    → Room pre-book (price re-confirmed) → Guest details → Confirmation
```

---

**Next:** [Showcase](showcase.md) · [Theming](theming.md) · [Troubleshooting](troubleshooting.md)
