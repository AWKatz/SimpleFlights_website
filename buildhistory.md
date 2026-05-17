# SimpleFlights — Build History

A complete record of every build increment since development began. Technical patch notes are grouped under their release milestone. Update this file alongside `pubspec.yaml` and `whats_new_dialog.dart` on every version bump.

---

## v1.0.1 — 2026-05-15

*Logbook export/import · Aviation decimal time · Guidance line ETE · Circuit detection · Map pin polish · iOS notification throttle · Night mode polish · MQTT sharing reliability · W&B status display*

| Build | Change |
|-------|--------|
| 119 | Map screen pillbox clearance: `_RouteLegLabel` / `_LegLabel` on all three map screens changed from `Center(child: Container(...))` to `Align(Alignment.bottomCenter) + Padding(bottom: 8) + Container`; anchor marker heights bumped 22→30 and 36→44 px; ensures dist/hdg/ETE pill never overlaps the traceline |
| 120 | Active flight ICAO pin UX: tap-to-toggle dismiss added to `_showAirportTooltip` — second tap on the same pin closes the tooltip; `_AirportPin.build()` wraps the sequence-number Text and ICAO Container in `if (!isSelected)` so the label is hidden while the runway overlay is visible |
| 121 | Guidance line distance/ETE pill: `_GuidanceLabel` widget added to active flight screen; rendered as a `MarkerLayer` at the midpoint of the amber dashed guidance line; shows live distance to next waypoint in NM and optional ETE derived from aircraft cruise TAS; amber colour theme matches the dashed line |
| 122 | Circuit/touch-and-go detection on all map screens: route polyline, leg labels, and guidance pill are suppressed when the resolved waypoint list has exactly two entries with matching coordinates (lat/lon delta < 1e-7); applied to active flight screen, flight detail screen, and planning screen; eliminates 0 NM label on runway circuit overlay |
| 123 | iOS background notification throttle: `_gpsCount % 60` → `_gpsCount % 300`; lock screen "Tap to return to app" notification re-posts at most once per ~5 minutes (300 GPS fixes at 1 Hz) instead of every minute |
| 124 | Aviation decimal time in flight log cards: AIR TIME and FLIGHT TIME badges display `formatAirTime()` output (nearest 0.1 hr block); DURATION badge added showing H:MM; Hobbs used when available, GPS duration as fallback; three-badge row with 6 px gaps |
| 125 | Flight card GPS trace layout fix: `_FlightTrackPainter` moved from `Positioned` in the outer `Stack` to an inline `SizedBox(height: traceAreaH, child: CustomPaint(...))` inside the Column between the route row and the footer; `traceAreaH` is responsive (60/84/112 dp at small/medium/large); eliminates overlap with ICAO labels and time badges |
| 126 | Logbook JSON export: `LogbookExportService.export(db)` serialises all aircraft (full schema) and flights (GPS points embedded as `{ts,lat,lng,alt_ft,spd_kts,hdg,acc_m}`); pretty-printed to a datestamped temp file; handed to native share sheet via `share_plus`; `file_picker ^8.0.0` added as dependency |
| 127 | Logbook JSON import: `LogbookExportService.import(db)` opens the system file picker, reads and validates the JSON (`export_version` check), then upserts aircraft and flights inside a single `db.transaction()` for atomicity; existing GPS points deleted before re-insert to prevent track duplication; `ImportResult` with `success/cancelled/error` factory constructors; `AppDatabase.transaction()` helper with `BEGIN/COMMIT/ROLLBACK` |
| 128 | Settings LOGBOOK DATA section: Export Logbook and Import Logbook `ListTile`s added inside `if (!demo)` block between DATABASE and AIRSPACE; `_runExport` surfaces share sheet errors as snackbar; `_runImport` shows success snackbar with imported counts and reloads `flightProvider` + invalidates `aircraftProvider` |
| 129 | Website Firefox gradient text fix: `-webkit-text-fill-color: transparent` added to `.bg-clip-text` CSS rule in `index.html`, `tos.html`, `privacy-policy.html`, `contact.html`; fixes `background-clip: text` gradient rendering in Gecko (Firefox for Android) |
| 130 | Version bump to 1.0.1; What's New dialog updated with 6 v1.0.1 feature entries |
| 131 | Settings Support item: Support `ListTile` added to ABOUT section; `launchUrl(Uri.parse('mailto:aaron@simpleflights.ca?subject=In-App%20Support%20Request'))` via `url_launcher ^6.3.0`; placed before Acknowledgements |
| 132 | Linux desktop app icon: `AppIcon.png` (1024×1024) bundled via `CMakeLists.txt` install rule into `data/app_icon.png`; `my_application.cc` resolves exe path via `readlink("/proc/self/exe")`, loads the PNG with `gdk_pixbuf_new_from_file`, and applies it with `gtk_window_set_icon` at startup |
| 133 | Night mode map icon tint: `MapIconBtn.activeColor` and `flight_detail_screen._MapButton` accent changed from `AppTheme.accentBlue` (hardcoded cyan) to `Theme.of(context).colorScheme.primary`; `_BlinkingDot` on flight detail map updated to match — active state now follows theme primary in all three themes |
| 134 | Theme-aware runway and route colours: `AppTheme.routeLineFor()` updated to return `accentBlue` for light mode (was always blood orange); `nightWrap` added to `airport_screen.dart` and `viewer_screen.dart` FlutterMap; runway polylines, runway markers, and route polyline on both screens wired to `routeLineFor(tileMode)` |
| 135 | MQTT reliability: `connectTimeoutPeriod` (milliseconds) now applied to `MqttServerClient` — was defined in `MqttConfig` but never used, causing indefinite hangs on unreachable brokers; `isConnecting` bool added to `MqttSharingState` and `SharingOrchestratorState`; `_cancelled` flag in `MqttPublisherNotifier.start()` discards stale connect results when `stop()` is called mid-connection |
| 136 | MQTT error UX: `_describeError()` maps raw exception strings (DNS, connection refused, timeout, TLS/handshake, auth, generic) to short user-readable messages with support detail in parentheses; share screen connecting state shows amber progress indicator and active STOP CONNECTING button; error display split into bold headline + de-emphasised monospace support detail |
| 137 | Active flight MQTT feedback: `ref.listen<SharingOrchestratorState>` shows a SnackBar on MQTT error transition; flight card share button shows red STOP OutlinedButton while connecting; idle SHARE button navigates to share screen via `Navigator.push(MaterialPageRoute)` — fixes go_router "duplicate page key" crash caused by pushing a shell-nested route (`/share`) from a top-level route (`/fly`) |
| 138 | W&B status chip in aircraft form: `_hasPayload` getter checks if any calculator weight field is non-zero; `_WbStatusChip` widget inserted inline in the WEIGHT & BALANCE section header row — shows "not configured" (grey) when POH limits are absent, "not checked" (amber) when no payload is entered, "within limits" (green) or "outside limits" (red) based on live calculator result; visible when section is collapsed |
| 139 | W&B card on aircraft detail screen: `_WbCard` widget added between specs card and notes; shows amber "NOT CONFIGURED" badge with setup prompt when `!a.hasWbData`; shows Max Gross Weight, CG Envelope, and Empty Weight/CG rows when configured; full-width "CHECK W&B CALCULATOR" / "CONFIGURE W&B" button navigates to edit screen |
| 140 | Aircraft detail AppBar edit button: `IconButton(Icons.edit_outlined)` replaced with `TextButton.icon` showing pencil icon + "EDIT" label |

---

## v1.0 — 2026-04-14

*Home screen widgets · Heading-up map · GPS trace z-order · Rotation-aware labels · Production deploy*

| Build | Change |
|-------|--------|
| 098 | `WidgetService` rewrite: replaced `updateIdle()` with `updateLogbook()` accepting full logbook totals + last flight summary; `updateFlightStopped()` extended with logbook params; `_triggerUpdate()` now fires all three Android receivers + iOS widget; fixed `_fmtElapsed` typo (`'$m:$m:$s'` → `'$m:$s'`) |
| 099 | iOS widget redesign: `SimpleFlightsWidget.swift` rewritten — `FlightEntry` reads logbook keys (`totalFlights`, `totalAirTime`, `totalHobbs`, `highestAlt`, `totalDist`, `lastRoute`, `lastDate`, `lastDuration`, `lastDistNm`, `lastMaxAlt`); `MediumView` (4×2) shows logbook totals + last flight; `LargeView` (4×4) adds LazyVGrid stat layout; `QuickStartView` (small) deep-links to flight screen; bundle rewritten with `@main` `SimpleFlightsWidgetBundle` |
| 100 | iOS `SimpleFlightsWidgetControl.swift` deleted: Xcode-generated iOS 18 Control widget caused compile errors and was not needed |
| 101 | Android `FlightStatusWidget.kt` rewritten: reads new logbook keys; shows flight count header, logbook totals row, last flight section; exports `BgColor`, `AccentColor`, `StatCell`, `GlanceDivider` as `internal` for reuse |
| 102 | Android `FlightLogWidget.kt` added: large (4×4) Glance widget showing 5-stat logbook grid (totalFlights, totalAirTime, totalDist, totalHobbs, highestAlt) + last flight card with route + 3 stats; `FlightLogWidgetReceiver` registered |
| 103 | Android `QuickStartWidget.kt` added: small widget with airplane icon + START FLIGHT CTA; `actionStartActivity<MainActivity>()` deep link; `QuickStartWidgetReceiver` registered |
| 104 | Android widget XML configs added: `flight_status_widget_info.xml` (4×2, 250×110dp), `flight_log_widget_info.xml` (4×4, 250×250dp), `quick_start_widget_info.xml` (2×2, 110×110dp, no resize) |
| 105 | `AndroidManifest.xml` updated: deep link intent filter (`simpleflights://` scheme) on `MainActivity`; `FlightStatusWidgetReceiver`, `FlightLogWidgetReceiver`, `QuickStartWidgetReceiver` registered with `APPWIDGET_UPDATE` intent filters and provider meta-data |
| 106 | `build.gradle.kts` updated: `buildFeatures { compose = true }`, `composeOptions { kotlinCompilerExtensionVersion = "1.5.14" }`; added `glance-appwidget:1.1.1`, `glance-material3:1.1.1`, Compose BOM `2024.09.00`, `compose.ui`, `material3` dependencies |
| 107 | `main.dart` widget push: `_pushLogbookToWidget()` called from `addPostFrameCallback` on app launch; computes all logbook totals from `flightProvider` + `aircraftProvider`; calls `WidgetService.updateLogbook()` |
| 108 | GPS trace z-order fix: GPS trace polylines and blinking dot removed from `FlutterMap.children`; replaced with `_GpsOverlay` widget rendered in the outer `Stack` after `FlutterMap` — guarantees GPS elements always paint above all map overlay layers regardless of `ConsumerStatefulWidget` compositing boundaries |
| 109 | `_GpsTracePainter` added: `CustomPainter` replicating altitude-colour segmentation logic (`AppTheme.altColor`) in screen space; `_GpsOverlay` subscribes to `mapController.mapEventStream` to repaint on every camera move; blinking dot positioned via `camera.latLngToScreenPoint()` |
| 110 | `_buildAltPolylines` removed: dead code after GPS trace moved out of `FlutterMap`; no functional change |
| 111 | Heading-up speed gate lowered: `>= 10.0 kts` guard reduced to `>= 2.0 kts` so heading-up activates earlier; path-bearing seed added — if `_hdgSeeded` is false and path has 2+ points, initial heading computed from last two fixes rather than defaulting to 0° (north) |
| 112 | Real GPS heading fallback: `pos.heading < 0` (invalid) or `spdKt < 2.0` now falls back to bearing computed from last two path points; prevents invalid `-1` heading from corrupting `_smoothedHdg` on stationary or slow-moving device |
| 113 | Rotation-aware route arrows: `_buildRouteArrows()` on active flight screen now subtracts `_mapRotation` from bearing angle so arrows stay geo-aligned when map is rotated in heading-up or manual rotate mode |
| 114 | Rotation-aware route arrows (planning): `_RouteArrow` gains `mapRotation` parameter; call site passes `_mapRotation`; angle is `(bearing - mapRotation) * pi/180` |
| 115 | Rotation-aware natural feature labels: `_buildFeatureLabels()` and `_buildNaturalLayer()` gain `mapRotationRad` parameter; waterway/railway label `Transform.rotate` angle becomes `labelAngle - mapRotationRad`; `build()` passes `camera.rotationRad` |
| 116 | Production deploy script (`deploy_production.sh`): `per_listener_settings false` added to Mosquitto config; `require_certificate true` removed (was enabling mutual TLS instead of password auth); `tls_version tlsv1.2` moved inside each TLS listener block; stock `mosquitto.conf` replaced with include-only stub; `conf.d/*.conf` cleanup loop deletes non-`simpleflights.conf` files; UFW port 80 pre-opened before Certbot (Step 1b) with `|| true` guard |
| 117 | Xcode build cycle fix: `USE_RECURSIVE_SCRIPT_INPUTS_IN_SCRIPT_PHASES = NO` set in Runner Release build config — root cause of "Cycle inside Runner" error involving `SimpleFlightsWidgetExtension` and Thin Binary + Embed Pods Frameworks phases |
| 118 | Version bump to 1.0.0; What's New dialog updated with 7 v1.0 feature entries; README updated with v1.0 header and What's New table |

---

## v0.98 RC — 2026-03-31

*Theme-aware map colours · Airspace overlay reliability · iOS waypoint UX · Viewport-based map loading*

| Build | Change |
|-------|--------|
| 087 | Planning screen UX: `legCount` now counts DEP + all VIAs + ARR correctly; compass and controls panel relocated to left side above HUD matching active flight layout |
| 088 | iOS map refresh fix: each DEP/VIA/ARR field gains a dedicated in-field commit button (`Icons.add_circle_outline_rounded`); `TextInputAction.search` + `autocorrect: false` triggers map update without requiring Enter tap |
| 089 | Listener accumulation fix: `_attachListeners()` rewritten with stored named closures in `_listeners` list; `removeListener(() {})` with new lambda was silently no-oping causing duplicate rebuilds on add/remove stop |
| 090 | Runway cache staleness fix: clearing a waypoint now removes its entry from `_runways` map preventing stale runway overlay data |
| 091 | Layer ordering hardened on all map screens: AirspaceOverlayLayer → runway overlays → MapOverlayLayer → route polyline → leg labels/arrows → ICAO pins; highway/railway/waterway labels always render below pins and route trace |
| 092 | Leg label pillboxes added to active flight screen and flight detail screen: dist + bearing pill at midpoint of each planned-route leg matching planning screen style |
| 093 | Runway zoom-gate added to flight detail screen: runway overlays (planned-route and tooltip) now zoom-gated at ≥ 12 matching all other map screens |
| 094 | Airspace zone labels: name + type badge + up to 2 contact frequencies rendered as `MarkerLayer` at zoom ≥ 8 overlaid on the `PolygonLayer` |
| 095 | Map data load performance: `MapOverlayLayer` split into two independent fetch paths — airports/navaids use full route bounds (local CSV, always fast); natural features use camera viewport snapped to 0.5° grid (Overpass, always small bbox regardless of route length) |
| 096 | Theme-aware route colours: `AppTheme.routeLineFor(SfThemeMode)` added; light theme → cyan (`0xFF00BCD4`), dark/night → blood orange (`0xFFCC3300`); applied to route polylines, direction arrows, leg-label pills, and runway overlays on all three map screens |
| 097 | Airspace overlay architecture rewrite: zone loading moved from widget `initState` into `airspaceZonesProvider` (`StateNotifierProvider`); `AirspaceOverlayLayer` simplified to a `ConsumerWidget` returning `PolygonLayer` + `MarkerLayer` directly from `build()` — eliminates flutter_map v7 `MobileLayerTransformer` projection race; `showAirspace` defaulted to `true`; overlay added to flight detail screen |

---

## v0.95 RC — 2026-03-13

*Flight planning tab · Airspace overlays · En-route weather · W&B calculator · Smooth heading-up*

| Build | Change |
|-------|--------|
| 080 | Airspace data layer: `AirspaceZone` model + `AirspaceType`/`AirspaceClass` enums; `parseAirspaceGeoJson` top-level function for `compute()` isolate; `AirspaceRepository` with 28-day disk cache per country; downloads from OpenAIP GCS bulk export |
| 081 | Airspace overlay UI: `AirspaceOverlayLayer` PolygonLayer zoom-gated at 6.5, bbox-filtered per frame; `region_picker_dialog` multi-select with live download/cached status; `airspace_regions_provider` (SharedPreferences-backed country list); `showAirspace` added to `MapOverlaySettings` |
| 082 | Flight plans: `flight_plans` SQLite table; `FlightPlan` + `FlightPlanWaypoint` domain models with JSON serialisation; `FlightPlanNotifier` StateNotifier (save / delete / byId) |
| 083 | Planning tab: `PlanningScreen` full-screen map + collapsible HUD; waypoint entry with airport auto-resolve; route polyline with waypoint pin markers; leg distance + bearing labels at midpoints; route summary (legs, NM, ETE from aircraft TAS); save/load plans; PLAN tab added to shell nav (index 2); `/planning` route registered |
| 084 | En-route weather briefing: `WeatherBriefingPanel` horizontally-scrollable METAR cards for all route stops; `WeatherStopCard` shows flight category dot, wind, visibility, cloud, altimeter; collapsible toggle bar; wired into Planning screen |
| 085 | Active flight load plan: LOAD PLAN button in pre-flight HUD opens a picker of saved plans; selecting a plan populates DEP/VIA/ARR waypoint controllers; airspace toggle added to active flight map controls panel |
| 086 | Settings airspace section: AIRSPACE section in Settings with Regions tile navigating to `/settings/regions`; `RegionsSettingsScreen` wraps the region picker dialog |

---

## v0.90 — 2026-03-09

*Map overlays · City labels · Smooth playback · W&B schema · Demo diversity*

| Build | Change |
|-------|--------|
| 064 | Map overlay layer: `MapOverlayLayer` added; fetches nearby airports and navaids from OurAirports CSV per bounding box; memoized futures prevent flicker on GPS rebuilds; toggle buttons in collapsible map controls panel |
| 065 | Natural features overlay: `NaturalFeaturesRepository` queries Overpass API for waterways (rivers/canals — blue polylines), railways (dotted grey polylines), and major highways (motorway+trunk — amber polylines); worldwide coverage; bbox cache |
| 066 | Map controls 2×2 grid: overlay toggles arranged as `[✈ airports][📡 navaids]` / `[💧 waterways][🛣 roads+rails]`; roads button toggles both highways (motorway+trunk) and railways simultaneously |
| 067 | Feature name labels: named waterways, highways, and railways show pill badge labels on map at zoom ≥ 9.5; de-duplicated per (type, name) — longest OSM segment wins; colour-coded amber/blue/grey matching polyline colour |
| 068 | City & town labels: Overpass `place` nodes rendered as pill badges — cities at zoom ≥ 6 (bold, larger), towns at zoom ≥ 8 (regular); theme-aware text (white on dark/night-red tiles, dark on light tiles) |
| 069 | Loading map data badge: animated blue spinner with "Loading map data" text at top-right of all map screens while any Overpass request is in-flight; fades out once all four data sources resolve |
| 070 | Overpass cache improvements: `NaturalFeaturesRepository` caps in-memory bbox cache at 20 entries (FIFO eviction) preventing unbounded growth on long cross-country flights; applies to both features and place-label caches |
| 071 | Overlay pre-warm consistency: `initLocation()` now pre-warms all four overlay sources (natural features, place labels, navaids, airports) as soon as an initial GPS fix is available |
| 072 | Shared `MapIconBtn` widget: circular 40×40 map button centralised in `map_controls_panel.dart`; replaces three separate private button classes across active-flight, flight-detail, and viewer screens |
| 073 | Map controls order standardised across all screens: zoom+, zoom−, centre/follow *(contextual)*, overlay toggles |
| 074 | Demo flight diversity: five sample flights replaced with geographically diverse global routes — KMIA→KTPA (Florida, US), EGKB→EGNM (London→Leeds, UK), MUHA→MUVR (Havana→Varadero, Cuba), CYOW→CYYZ (Ottawa→Toronto, Canada), TJSJ→TJBQ (San Juan→Aguadilla, Puerto Rico) |
| 075 | Demo flight playback density: `_denseTrack` interpolator generates GPS points at 30-second intervals — produces 61–197 points per flight (vs 6–9 previously); scrubber, altitude chart, and animated dot now behave like a real logged flight |
| 076 | Smooth heading-up mode: 60fps camera animation via `AnimationController` interpolates position + rotation between GPS fixes; speed gate (≥10 kts) prevents erratic rotation while taxiing; heading-up ON by default |
| 077 | Settings About sheet: Version tile is tappable — opens a bottom sheet displaying version number and the full What's New feature list |
| 078 | W&B + fuel planning schema: 13 new nullable aircraft columns (`cruise_tas_kts`, `fuel_flow_gph`, `empty_weight_lb`, `empty_cg_arm_in`, `max_gross_weight_lb`, 5 station arms, `fuel_capacity_gal`, 2 CG limits); idempotent `ALTER TABLE` migrations |
| 079 | Aircraft form W&B section: collapsible WEIGHT & BALANCE panel with POH fields + inline payload calculator; live CG and gross weight computation; colour-coded `_WbResultCard` (pass/fail vs limits) |

---

## v0.80 — 206-03-05

*Social sharing · Unit profiles · Enroute editing · Compass · GPS reliability*

| Build | Change |
|-------|--------|
| 041 | Social sharing: FlightShareSheet (share-as-image / share-as-text / quick SMS), notification contacts |
| 042 | Watch Mode + TV Mode: simplified display modes accessible from Settings → DEBUG |
| 043 | MQTT viewer: `waypoint_coords` in retained session info — airport pins displayed without local DB lookup; route polyline + amber guidance line now rendered |
| 044 | MQTT viewer: auto-reconnect via `WidgetsBindingObserver` + 5-second retry timer on disconnect |
| 045 | MQTT viewer: auto-broker detection from share URL host; iOS TLS `onBadCertificate` fix in viewer |
| 046 | MQTT viewer: PILOT OFFLINE state — presence `offline` shows amber status without dropping to connect dialog; restores LIVE on `online` |
| 047 | iOS flight timer fix: elapsed time and saved `startTime` derived from wall-clock `_flightStart` instead of throttled `Timer.periodic` |
| 048 | iOS notification: `presentBanner: false` + `presentList: true` — notification now appears in notification centre and on lock screen |
| 049 | iOS notification persistence: GPS-callback re-post every 60 events (~1 min) so notification reappears if swiped away |
| 050 | Unit profiles: Settings → UNITS picker (AVIATION / METRIC / IMPERIAL) — converts speed, altitude, and distance display throughout the app and viewer |
| 051 | Airport lookup: confirmed worldwide coverage (EGLL, MUHA tested); altimeter auto-displays inHg or hPa based on returned value |
| 052 | Viewer pairing simplified: 6-character code entry (auto-uppercase, length-limited); full URL prefix handled automatically |
| 053 | Viewer CANCEL button: styled red CANCEL button; LAN broker preset removed (TLS cloud broker is now the default) |
| 054 | Enroute stop: pilot can add/remove VIA waypoints while a flight is actively logging; DEP locked once airborne |
| 055 | Map auto-follow toggle: manually panning disables auto-follow; tapping GPS button re-centres and re-enables follow mode |
| 056 | Compass overlay on all map screens: `MapCompass` widget (88×88 circle, rotating needle, HDG readout) |
| 057 | Flight notes prompt: after HOBBS OUT, pilot prompted to add free-text notes; stored in `flights.notes` |
| 058 | Flight card trace rendering fix: removed `ShaderMask` wrapper; gradient fade moved inside `_FlightTrackPainter` |
| 059 | Pinch-to-zoom and two-finger rotate enabled on all map screens via `InteractiveFlag.rotate` |
| 060 | Collapsible map controls: `MapControlsPanel` widget replaces fixed button columns; `gearAtEnd` param for bottom-anchored panels |
| 061 | Compass improvements: counter-rotates for north-up when map is rotated; S/E/W labels added; needle changed to blue navigation arrow |
| 062 | MQTT share screen: displays clean 6-character session code; COPY and SHARE use the code only |
| 063 | Nearby airport pins fix: OurAirports CSV fallback in `lookup()` so airports not in AWC or Overpass still resolve |

---

## v0.70 — 2026-02-28

*Real GPS · Multi-leg routes · MQTT sharing · Airport info*

| Build | Change |
|-------|--------|
| 021 | Hobbs meter tracking |
| 022 | Demo Mode + SQLite live storage |
| 023 | Raw sqlite3 (no codegen) |
| 024 | Share toggle + URL banner |
| 025 | Background logging + flight banner |
| 026 | Browser viewer theme toggle |
| 027 | MQTT sharing mode + demo banner |
| 028 | Airport info tab + DEP/ARR markers on flight map |
| 029 | Aircraft selection on flight screen + airport favourites (★ save/pick) |
| 030 | Real GPS tracking (live mode) + lock screen notification + MQTT broker selector |
| 031 | Multi-leg routes (VIA waypoints): up to 4 intermediate stops, map pins in cyan, route polyline |
| 032 | Next-waypoint amber guidance line with 3 NM auto-advance (pilot map + WiFi viewer + in-app viewer) |
| 033 | Via waypoints persisted to database (`via_icaos` column); displayed inline on flight cards |
| 034 | MQTT telemetry fix: GPS now pushed to both WiFi and MQTT backends simultaneously |
| 035 | Browser viewer updated: airport pins, planned-route line, amber guidance line, `/airport` endpoint |
| 036 | Notification: OS-driven chronometer, `Importance.defaultImportance` for lock screen, silent channel |
| 037 | MQTT TLS: `onBadCertificate` callback typed as `(Object) => bool` — fixes Android type-cast crash |
| 038 | Settings DATABASE section: live database file-size indicator with colour-coded progress bar |
| 039 | GPS track compression: Ramer–Douglas–Peucker algorithm with epsilon slider and confirmation dialog |
| 040 | What's New dialog: version-gated, shown once per app version upgrade; debug trigger in Settings |

---

## v0.60 — 2026-02-07

*Material 3 foundation · Map · Playback · Themes · MVP*

| Build | Change |
|-------|--------|
| 001–020 | Material 3, map, playback, themes, icons, MVP 4–6 |
