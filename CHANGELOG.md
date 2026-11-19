# Changelog

All notable changes to the Gate2 Travel SDK for Android. Versions are published per company
(`travel.gate2:<module>-<company>:<version>`), so not every version exists for every company; see
[Installation](docs/getting-started.md#installation).

| Version | Date | Highlights |
|:--|:--|:--|
| **1.2.4** | Nov 2026 | eSIM: the confirmation screen shows a "Check this after receiving your eSIM" box below the installation steps (remove old eSIM profiles, make sure the eSIM line is active, turn on data roaming), in English, Azerbaijani and Russian. No API changes. |
| **1.2.3** | Nov 2026 | eSIM: promotional banner supports the new top slot and per-language images and keeps the image's own proportions; FAQ answers without a question title are shown directly instead of collapsed; numbered lists in FAQ answers. Brand builds can carry their own light and dark colour palette. No API changes. |
| **1.2.2** | Nov 2026 | eSIM: redesigned plan filters (plan type, region, country, data volume, duration) with a live result count. Built with Android Gradle Plugin 9; Kotlin stays 2.2.21. The AARs now require `compileSdk` 35 or newer. No API changes. |
| **1.2.1** | Oct 2026 | Flights: seats overview on the review screen, "Select seats" entry from the passenger form, scroll edge fades on flight screens. |
| **1.2.0** | Oct 2026 | Flights: seat map after pricing (display-only selection), saved-traveler edit, recent-search removal (flights v2 endpoints). |
| **1.1.6** | Sep 2026 | eSIM: merchant banner on the search screen. |
| **1.1.5** | Sep 2026 | eSIM: plan discounts. |
| **1.1.4** | Jun – Jul 2026 | Brand and environment moved from the version into the artifact ID. Brandable typography. Flight order validation. |
| **1.1.0 – 1.1.1** | Jun 2026 | Shared `core` across products; `travel-all` umbrella with `Gate2TravelShowcase`; per-brand builds; Flights and Hotels products. |
| **1.0.x** | Jan – Jun 2026 | eSIM-only line: automatic installation, T&C flow, structured `EsimError`, My eSIMs, top-ups, device-compatibility banner. Retired coordinates; see [Migrating from 1.0.x](docs/migration-from-1.0.md). |
