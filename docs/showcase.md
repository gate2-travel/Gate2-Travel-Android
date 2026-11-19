# Showcase screen (travel-all)

[Documentation home](../README.md) · [Getting started](getting-started.md) · [eSIM](esim.md) · [Flights](flights.md) · [Hotels](hotels.md) · **Showcase** · [Troubleshooting](troubleshooting.md)

`travel-all-<company>` bundles eSIM, Flights and Hotels and adds `Gate2TravelShowcase`, a Compose
screen that presents the products and lets the host launch each one with its own callbacks. It
wraps itself in the SDK theme; your app only needs Compose to place it.

> **Artifact:** `travel.gate2:travel-all-<company>:1.2.4`, if your agreement covers all three
> products. Install and initialize the SDK first: [Getting started](getting-started.md).

---

## Usage

```kotlin
import com.gate2.sdk.travel.Gate2TravelShowcase
import com.gate2.sdk.travel.Gate2ShowcaseConfig
import com.gate2.sdk.travel.Gate2CabinetConfig
import com.gate2.sdk.travel.Gate2Product

setContent {
    Gate2TravelShowcase(
        onProductSelected = { product ->
            when (product) {
                Gate2Product.ESIM    -> launchEsim()
                Gate2Product.FLIGHTS -> launchFlights()
                Gate2Product.HOTELS  -> launchHotels()
            }
        },
        onBack = { finish() },   // null = embeddable (no back arrow, no back interception)
        config = Gate2ShowcaseConfig(
            headerTitle    = "Travel with us",
            headerSubtitle = "Everything for your trip, in one place",
            promoImageUrl  = null,                           // optional banner above the cards
            products       = Gate2ShowcaseConfig.defaultProducts(),
            cabinet        = Gate2CabinetConfig(
                onOpenOrders = { product ->
                    when (product) {
                        Gate2Product.FLIGHTS -> Gate2Flight.startMyFlights(this, sessionId, flightCallbacks, userId)
                        Gate2Product.ESIM    -> Gate2Esim.showMyEsims(this, sessionId, esimCallbacks, userId)
                        Gate2Product.HOTELS  -> launchHotels()
                    }
                },
                onOpenTravelers = { Gate2Flight.startTravelers(this, sessionId, flightCallbacks, userId) },
                onOpenFaq       = { Gate2Flight.startFaq(this, sessionId, flightCallbacks, userId) },
                supportWhatsApp = "+12025550123",
                supportEmail    = "support@gate2.travel",
            ),
        ),
    )
}
```

## Configuration

| Type | Fields |
|:--|:--|
| `Gate2ShowcaseConfig` | `headerTitle`, `headerSubtitle`, `promoImageUrl`, `products: List<Gate2ShowcaseProduct>`, `cabinet: Gate2CabinetConfig?` |
| `Gate2ShowcaseProduct` | `product: Gate2Product`, `title`, `subtitle`, `imageUrl`, `badge` (e.g. `"NEW"`), `enabled` |
| `Gate2CabinetConfig` | `onOpenOrders`, `onOpenTravelers`, `onOpenFaq`, `supportWhatsApp`, `supportEmail`. Rows appear only for the callbacks you provide. |
| `Gate2Product` | `ESIM`, `FLIGHTS`, `HOTELS` |

The rendered design is brand-specific: some brand builds ship a redesigned home with a "My cabinet"
section, others use the default card picker. The API is identical.

---

**Next:** [Theming](theming.md) · [Troubleshooting](troubleshooting.md) · [API reference](api-reference.md)
