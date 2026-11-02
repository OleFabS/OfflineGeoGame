# OfflineGeoGame

A cooperative offline Android game for two players, each using their own phone. One player draws a clue for a country. The other places a pin where they think its capital is. Closer guesses earn more points for the team.

**Status:** Android starter only. The libraries and features below are planned, not implemented yet.

## First version

- Two rounds per match: each player draws once and guesses once.
- One shared team score.
- The drawer receives a country selected randomly from a predetermined list.
- Any drawing clue is allowed; no clue restrictions for now.
- The guesser sees the finished sketch, then places a pin on the map.
- Distance is measured from the guessed pin to the country's capital.
- Play without internet, a router, or a game account.

## Game flow

1. One player hosts; the other finds the game through Nearby Connections.
2. Both confirm the matching connection code.
3. Only the drawer sees the country prompt and creates a sketch.
4. The drawer submits the finished sketch. The other player sees it while using the map.
5. The guesser places a pin and confirms it.
6. Both see the country, capital, correct location, distance, and team points earned.
7. Swap roles for round two, then show the final team score and offer a rematch.

The game host stays host when drawing roles change.

## Libraries and data

| Part | Choice | Purpose |
| --- | --- | --- |
| Language and UI | Kotlin, Jetpack Compose, Material 3 | Screens, controls, and drawing canvas; already in the starter. |
| Connection | Google Nearby Connections | Find, pair, and transfer messages and sketches over Bluetooth and Wi-Fi. Requires Google Play services and relevant permissions. |
| Map | MapLibre Native for Android | Pan, zoom, map coordinates, and guess/target markers. Use a small Compose wrapper around its Android view. |
| Map data | Bundled Natural Earth-derived GeoJSON and local style | Country borders and coastlines without map downloads. Recommended starting display: no country or city labels. |
| Game data | Bundled JSON | Predetermined country list, capital names, and target coordinates. Record sources and one target for countries with multiple capitals. |
| Messages | Kotlin serialization | Convert typed game messages to and from JSON. |
| State and background work | AndroidX ViewModel, StateFlow, Kotlin coroutines | Keep UI state and asynchronous work out of screen code. |
| Sketch transfer | Compose Canvas, Android Bitmap APIs, Nearby file payloads | Export and send a finished PNG without a separate drawing library. |
| Tests | Existing JUnit and Compose test tools | Check game rules, scoring, messages, and important UI flows. |

Bundle every map resource needed for a fresh offline launch, including its style and marker assets. Capital coordinates determine scoring; country borders are visual context. Calculate geographic distance in kilometers, not screen pixels.

Add libraries when their task starts and pin verified versions in `gradle/libs.versions.toml`. Check compatibility with the existing toolchain before upgrading anything. No database, cloud backend, extra networking framework, or dependency-injection framework is planned for version one.

## Code structure

Keep one app module under the existing `com.example.offlinegeogame` package. The package and assets folders are prepared with `.gitkeep` files. Add source files as each feature is implemented.

```text
MainActivity.kt    App entry and dependency wiring
ui/               Compose screens, theme, and GameViewModel
connection/       Nearby calls, permissions support, messages, and sketch transfers
game/             Round state, role changes, distance, and scoring
geography/        Country/capital models and bundled catalog loading
map/              MapLibre wrapper, map lifecycle, and markers
drawing/          Canvas input and PNG export
```

Put map data, style, and country/capital JSON in `app/src/main/assets/`. Keep provenance and conversion instructions alongside the data.

Screens send actions to the ViewModel. The host validates game actions, updates the round, and sends results to the other phone. Keep rules and scoring separate from Nearby and MapLibre so they can be tested without Android devices. Keep map views out of the ViewModel and release their resources when leaving.

## Implementation tasks

All tasks are pending. Work through them one at a time.

| ID | Task | Completion check |
| --- | --- | --- |
| 1 | Verify build and test phones | Debug app installs on two physical phones. Record Android versions, Play services availability, and build setup. |
| 2 | Add Nearby pairing | Host/find, permissions, matching-code confirmation, rejection, and cancellation work. Use Point-to-Point for two phones. |
| 3 | Exchange messages and retry | Both phones exchange versioned messages without internet or a router. Disconnect and start a new session; repeat with host roles reversed. |
| 4 | Add the offline MapLibre map | Fresh offline launch displays bundled borders/coastlines. Pan, zoom, and tap-to-pin work. Leaving the map releases its resources. |
| 5 | Draw and transfer a sketch | Either player can draw and clear, submit a bounded PNG, and see it render correctly on the other phone. Failed transfers can be retried. |
| 6 | Add countries and scoring | Load the predetermined list and capital coordinates. Check known distances, zero distance, date-line crossings, and the agreed scoring curve. |
| 7 | Complete a match | Drawing, guessing, reveal, role swap, two-round team result, and rematch work. Both phones show identical results. |
| 8 | Test and trim | Complete games on both phones with either hosting. Check denied permissions, failed transfers, disconnects, Back, rotation, screen lock, and fresh offline map loading. Measure release size. |

For messages and sketches, include session/round identifiers and reject stale or duplicate actions. Associate sketch files with their metadata and wait for successful transfer before enabling guessing. Award each round's points once and clean up temporary images on exit.

Suggested first-version interruption policy: a disconnect ends the match; reconnect starts a new one. Keep sessions in the foreground. Background play, match recovery, live drawing, larger groups, and saved history can follow playtesting.

## Decisions still open

- Which countries are on the list? Selection is random; suggest a small familiar set and no repeats within a match.
- What scoring curve and full-score tolerance should we use? Suggest up to 1,000 points per round, with some tolerance for finger placement.
- Confirm the map hints: borders/coastlines only, or country labels too? City labels could reveal the capital.
- Which two phones and Android versions can we test?

Two rounds are enough to prove the game. With unrestricted clues, players can write the answer; that is allowed for now. Playtest enjoyment before adding restrictions or more features.

## Development and contribution

Open the project in Android Studio and sync Gradle. Current settings: minimum Android 7.0 (API 24), compile/target SDK 37. Build and library compatibility still need verification. Follow Nearby's current permission guidance for each supported Android version, including API 37 where applicable.

Contributors and AI coding agents should take one task at a time, explain the code introduced, and report checks run and anything unverified. Use unit tests for rules and distance, physical phones for Nearby, and fresh offline launches for bundled map assets. Do not mark a planned feature as implemented.

## References

- [Nearby overview](https://developers.google.com/nearby/connections/overview), [Android setup](https://developers.google.com/nearby/connections/android/get-started), and [data transfer](https://developers.google.com/nearby/connections/android/exchange-data)
- [MapLibre Android](https://maplibre.org/maplibre-native/android/examples/) and [local GeoJSON](https://maplibre.org/maplibre-native/android/examples/geojson-guide/)
- [Natural Earth data terms](https://www.naturalearthdata.com/about/terms-of-use/)
- [Kotlin serialization](https://kotlinlang.org/docs/serialization.html) and [Android Views in Compose](https://developer.android.com/develop/ui/compose/migrate/interoperability-apis/views-in-compose)
