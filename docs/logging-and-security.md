# Logging, network and security

[Documentation home](../README.md) · [Getting started](getting-started.md) · [eSIM](esim.md) · [Flights](flights.md) · [Hotels](hotels.md) · [Showcase](showcase.md) · [Troubleshooting](troubleshooting.md)

---

## Logging

```kotlin
Gate2SdkConfig.builder()
    .context(this)
    .apiKey(token)
    .logLevel(SdkLogLevel.WARN)
    .onLog { entry: SdkLogEntry ->
        when (entry.level) {
            SdkLogLevel.ERROR, SdkLogLevel.FATAL -> crashReporter.captureMessage("${entry.tag}: ${entry.message}")
            SdkLogLevel.WARN                     -> crashReporter.addBreadcrumb(entry.message)
            else                                 -> Unit
        }
    }
    .build()
```

`SdkLogEntry` carries `level`, `tag` (e.g. `Gate2-esim`), `message` and `timestampMs`. Exceptions
thrown inside `onLog` are caught and never crash the SDK. Use `SdkLogLevel.ALL` while integrating
and `NONE` in release builds.

---

## Network and security

| | |
|:--|:--|
| **Production host** | `https://gw.gate2.travel/` (unsuffixed artifacts) |
| **Staging host** | `https://api.gate2.travel/` (`-staging` artifacts) |
| **Transport** | HTTPS with certificate pinning and an embedded mTLS client certificate. Nothing to configure, no certificate to ship. |
| **Timeouts** | 30 s connect / read / write |
| **Card data** | Never touches the SDK |

Headers on every request:

```
X-API-Key: <your token>
X-Session-Id: <sessionId you passed>
X-User-Id: <userId you passed, if any>
X-SDK-Version: 1.2.4
X-Platform: Android
Accept-Language: <language you passed>
```

The manifest permissions the SDK adds are listed in
[Getting started](getting-started.md#what-the-sdk-adds-to-your-manifest).

---

**Next:** [Troubleshooting](troubleshooting.md) · [API reference](api-reference.md)
