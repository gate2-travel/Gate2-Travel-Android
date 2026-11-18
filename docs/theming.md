# Theming and branding

[Documentation home](../README.md) · [Getting started](getting-started.md) · [eSIM](esim.md) · [Flights](flights.md) · [Hotels](hotels.md) · [Showcase](showcase.md) · [Troubleshooting](troubleshooting.md)

Your brand artifact already carries your look: accent colour, a colour palette for light and dark
mode, welcome screens, assets and (where supplied) your typeface are compiled into
`<module>-<company>`. The screens follow the device's light/dark setting. **In the normal case
there is nothing to configure.**

---

## Per-launch overrides

```kotlin
Gate2Esim.start(..., accentColor = ContextCompat.getColor(this, R.color.brand), fontResId = R.font.brand_sans)
Gate2Flight.start(..., accentColor = ContextCompat.getColor(this, R.color.brand))
Gate2Hotel.start(...,  accentColor = ContextCompat.getColor(this, R.color.brand))
```

`fontResId` is accepted by the eSIM entry points (0 = your brand font, or the bundled Inter). A
variable font maps weights 400/500/600/700 automatically; a static font renders at one weight.

---

## App-wide `Gate2Theme`

Pass a theme at initialization or swap it at runtime. Reach for this only if the compiled-in brand
look must change; contact Gate2 first, since brand changes are normally made in the brand build.

```kotlin
import com.gate2.sdk.core.config.*

val theme = Gate2Theme.builder()
    .primaryColor(0xFF1A73E8.toInt())             // shortcut for PrimaryColors(main = ...)
    .secondaryColor(0xFF34A853.toInt())
    .backgroundColors(BackgroundColors.builder().background(0xFFFFFFFF.toInt()).card(0xFFFFFFFF.toInt()).build())
    .textColors(TextColors.builder().primary(0xFF202124.toInt()).link(0xFF1A73E8.toInt()).build())
    .statusColors(StatusColors.builder().success(0xFF34A853.toInt()).error(0xFFEA4335.toInt()).build())
    .shapes(ShapeConfig.builder().cardRadius(16).buttonRadius(24).inputRadius(8).build())   // dp
    .spacing(SpacingConfig.builder().md(16).lg(24).build())                                 // dp
    .typography(TypographyConfig.builder().titleLarge(22).bodyMedium(14).build())           // sp
    .build()

Gate2SdkConfig.builder().context(this).apiKey(token).theme(theme).build()   // at init
Gate2TravelSdk.updateTheme(theme)                                           // or later
```

| Builder | Properties |
|:--|:--|
| `PrimaryColors` / `SecondaryColors` | `main`, `dark`, `light`, `onPrimary` / `onSecondary` |
| `BackgroundColors` | `background`, `surface`, `card`, `divider`, `border` |
| `TextColors` | `primary`, `secondary`, `hint`, `disabled`, `link` |
| `StatusColors` | `success`, `warning`, `error`, `info` |
| `ShapeConfig` | `cardRadius`, `buttonRadius`, `inputRadius`, `chipRadius`, `bottomSheetRadius`, `dialogRadius`, `imageRadius`, `uniform(radius)` |
| `SpacingConfig` | `xxs` … `xxxl` |
| `TypographyConfig` | Material 3 type scale sizes (`displayLarge` … `labelSmall`) |
| `ComponentConfig` | Card elevation, button/input heights, icon and avatar sizes, divider thickness |

Presets: `Gate2Theme.cleanWhite`, `Gate2Theme.darkMode`, `Gate2Theme.rounded`, `Gate2Theme.minimal`,
and `Gate2Theme.simple(primary, secondary)`.

---

**Next:** [Logging, network and security](logging-and-security.md) · [Troubleshooting](troubleshooting.md)
