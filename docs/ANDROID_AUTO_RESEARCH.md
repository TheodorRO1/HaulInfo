# Android Auto Reference Research

**Project:** HaulInfo  
**Research date:** November 25, 2026  
**Phase:** Phase 1 — Android Auto reference research  
**Status:** Under review  
**Reference:** Latest actually released Android Auto version

---

## 1. Research scope

The goal of this research is to understand how Android Auto looks, behaves, and responds to user interactions, so that HaulInfo can recreate the Android Auto experience as faithfully as possible on Windows for ETS2.

This is a documentation-only research phase. No application code was written and no framework decisions were made.

All findings are based on publicly available web sources and official Google documentation. Every claim is sourced alongside the finding.

## 2. Current Android Auto version and availability

### Released versions (as of November 25, 2026)

| Version | Release channel | Release date | Notes | Source |
|---------|----------------|--------------|-------|--------|
| 18.2 | Stable (wave rollout) | Nov 20, 2026 | App-internal side navigation observed in some media apps and the dialer; larger music artwork; mini-player for playback control | [autoevolution — Nov 20, 2026](https://www.autoevolution.com/news/everyone-can-now-update-to-android-auto-182-here-s-how-to-do-it-278698.html), [autoevolution — Nov 25, 2026](https://www.autoevolution.com/news/android-auto-182-starts-rolling-out-with-a-big-navigation-change-for-wide-screens-278950.html), [Android Authority — Nov 24, 2026](https://www.androidauthority.com/android-auto-app-layout-3739081/) |
| 18.1 | Stable | Nov 3, 2026 | Light mode assets and Cielo redesign preparation observed in teardown; no confirmed user-facing changes | [9to5Google — Nov 3, 2026](https://9to5google.com/2026/11/03/android-auto-18-1-update-rollout-teardown/) |
| 18.0 | Stable | Mid-October 2026 | Initial release of the v18 series | [9to5Google — Nov 3, 2026](https://9to5google.com/2026/11/03/android-auto-18-1-update-rollout-teardown/) |

### Version 18.2 — reported changes

Android Auto 18.2 is the latest version in the stable channel as of November 25, 2026. It is rolling out through the Google Play Store using Google's wave-based system, so not all users receive it at the same time. Manual APK installation is also possible.

Reported observations about v18.2 come from third-party reporting, not from an official Google changelog (Google does not publish changelogs for Android Auto builds):

- **autoevolution (Nov 20, 2026)** reports that the production rollout of v18.2 began and that "there are no new front-facing changes," describing the update as focused on bug fixes and performance refinement.
- **Android Authority (Nov 24, 2026)** reports a hands-on observation of a new app navigation layout in v18.2.664604-release, where navigation menus inside certain applications (Spotify, YouTube Music, Amazon Music, and the system dialer) are stacked along the side of the screen instead of at the top. The report also notes a larger now-playing bar with album art and playback controls.
- **autoevolution (Nov 25, 2026)** reports on the same wide-screen navigation change, describing it as making the interface easier to use on wide displays.

Because these are third-party observations from a staged rollout, it cannot be confirmed that every v18.2 installation has the same interface. Availability depends on the user's device, region, and rollout wave.

### Important distinction: system navigation bar vs app-internal navigation

Two different interface elements must not be conflated:

1. **System navigation bar** — Android Auto already supports a vertical system navigation bar on the side of the screen for widescreen or touchpad-supported displays. This is existing system UI functionality, documented in Google's design guidelines. It is not a new feature in v18.2.

2. **App-internal navigation menus** — The v18.2 change reported by Android Authority concerns navigation menus *inside certain applications* (Spotify, YouTube Music, Amazon Music, and the system dialer), where app options are stacked along the side instead of at the top of the screen. This is an application-level layout change, not a change to the Android Auto system navigation bar.

### Unreleased / announced features (not generally available)

These features were announced at Google I/O 2026 (May 12, 2026) or observed in beta/teardown analysis but are **not yet released** to production users:

- **Cielo redesign** — a major widget-filled redesign. Widgets displayed side-by-side with dashboard content.
- **Video app support** (YouTube) — video playback when parked; audio-only mode when driving.
- **Browser support** (Chrome, Vivaldi likely) — web browsing when parked only.
- **Light mode** — UI assets observed in 18.1 teardown; not yet user-facing.
- **Quick Settings tiles** — observed in 18.1 teardown; not yet user-facing.
- **Gemini integration** — Magic Cue context awareness, Doordash ordering, car-specific Q&A.
- **Immersive Navigation / live lane guidance** — clarified in section 4.3.

Google has not shared an ETA for these features.

**Important:** HaulInfo's first version should be built around the *released* Android Auto experience, not around announced or beta-only features.

**Sources:**
- [Google blog — Android in cars updates, May 12, 2026](https://blog.google/products-and-platforms/platforms/android/android-in-cars-updates/)
- [autoevolution — Nov 20, 2026](https://www.autoevolution.com/news/everyone-can-now-update-to-android-auto-182-here-s-how-to-do-it-278698.html)
- [autoevolution — Nov 25, 2026](https://www.autoevolution.com/news/android-auto-182-starts-rolling-out-with-a-big-navigation-change-for-wide-screens-278950.html)
- [Android Authority — Nov 24, 2026](https://www.androidauthority.com/android-auto-app-layout-3739081/)
- [9to5Google — Nov 3, 2026](https://9to5google.com/2026/11/03/android-auto-18-1-update-rollout-teardown/)


## 3. Interface breakdown

### 3.1 System UI structure

Android Auto's interface consists of three main regions:

| Region | Position | Contents | Source |
|--------|----------|----------|--------|
| Navigation bar | Bottom or side of screen | Apps launcher, Assistant button, App dock, Notification Center badge, Status elements | [Google Design — Navigation bar](https://developers.google.com/cars/design/android-auto/product-experience/system-ui/nav-bar) |
| Content area | Above or beside navigation bar | Dashboard (minimized view) or full-screen app (expanded view) | [Google Design — Content area](https://developers.google.com/cars/design/android-auto/product-experience/system-ui/content-area) |
| Status bar | Top of screen | Clock, weather, connectivity, battery; optional ETC icon in Japan | [Google Design — Status bar](https://developers.google.com/cars/design/android-auto/product-experience/system-ui/status-bar) |

**System UI specification.** These regions are defined by Google's Android Auto design guidelines and apply to the system UI itself.

### 3.2 Navigation bar (system navigation bar)

The navigation bar is always visible when Android Auto is running. Its position and orientation vary by screen dimensions, vehicle controls, and left/right-hand drive:

- **Horizontal** — bottom of screen; standard-width screens; left-hand drive.
- **Flipped horizontal** — bottom of screen; right-hand drive; items reversed so the most important elements stay closest to the driver.
- **Vertical** — side of screen; widescreen or touchpad-supported vehicles.

**Elements:**
1. **Apps launcher** — shows all available apps; replaced by a dashboard icon when the app launcher is open in the content area.
2. **Assistant button** — opens Google Assistant for voice interaction.
3. **App dock** — shortcuts to recently used apps.
4. **Notification Center badge** — shows count of waiting notifications.
5. **Status elements** — clock, signal, battery.

**System UI specification.** Documented in Google's Android Auto design guidelines. The vertical navigation bar is existing functionality for widescreen displays — it is not a v18.2 introduction. The v18.2 change reported by third parties concerns app-internal navigation menus (see section 2), not the system navigation bar.

**Source:** [Google Design — Navigation bar](https://developers.google.com/cars/design/android-auto/product-experience/system-ui/nav-bar)

### 3.3 Status bar

Located at the top of the screen. Elements appear in the same order regardless of left/right-hand drive:

- Clock (digital time)
- Weather information (temperature + icon; toggleable in Settings)
- Connectivity (Wi-Fi and cellular)
- Battery level
- ETC icon (Japan only — Electronic Toll Collection)

**Background options:** opaque, transparent, or semi-transparent gradient. Default is opaque.

**System UI specification.** Documented in Google's Android Auto design guidelines.

**Source:** [Google Design — Status bar](https://developers.google.com/cars/design/android-auto/product-experience/system-ui/status-bar)

### 3.4 Content area — dashboard (minimized view)

The dashboard gives users quick access to multiple active apps. It contains:

- A map or main app
- Cards with concurrent tasks and activities
- The navigation bar (including an app launcher icon)

Cards provide access to the most-used functions of an app. Full functionality requires entering the expanded view. The dashboard adjusts its layout as events occur (e.g., music area shrinks when a message arrives during navigation).

Users return to the dashboard by tapping the dashboard icon (which replaces the app launcher icon in the navigation bar when the launcher is open).

**System UI specification.** Documented in Google's Android Auto design guidelines.

**Source:** [Google Design — Content area](https://developers.google.com/cars/design/android-auto/product-experience/system-ui/content-area)

### 3.5 Content area — expanded view

The app takes over the entire content area for a richer experience. Users enter expanded view by tapping a card, toast, or notification. They exit by tapping the dashboard icon.

The navigation bar remains visible, showing a dashboard icon instead of the app launcher icon.

**System UI specification.** Documented in Google's Android Auto design guidelines.

**Source:** [Google Design — Content area](https://developers.google.com/cars/design/android-auto/product-experience/system-ui/content-area)


## 4. Navigation research

### 4.1 Navigation widget (existing system widget)

When navigation is active but the map is not on-screen, a **Navigation widget** appears in the navigation bar, showing turn-by-turn directions. The widget provides information about an upcoming turn while the user browses media or uses other apps.

**System UI specification.** Documented in Google's Android Auto design guidelines as an existing widget type.

**Source:** [Google Design — Widgets](https://developers.google.com/cars/design/android-auto/product-experience/system-ui/widgets)

### 4.2 Navigation templates

Android Auto provides app templates for navigation experiences, including:

- **Navigation template** — for turn-by-turn guidance
- **Map + Content template** — map alongside supporting content
- **Route Preview template** — route overview before starting navigation
- **Pane template** — supplementary information panes
- **Place List (map) template** — search results on a map

**Third-party app design guideline.** These templates are provided for apps built with the Android for Cars App Library. They define how navigation apps present content within Android Auto, not the system UI itself.

**Source:** [Google Design — Templates Overview](https://developers.google.com/cars/design/create-apps/media-apps/create-settings)

### 4.3 Immersive Navigation — platform clarification

Google's May 12, 2026 announcement covers "Android in cars" broadly, including both Android Auto (phone projection) and cars with Google built-in (Android Automotive OS). The following capabilities must be separated by platform:

| Capability | Platform | Availability | Hardware requirement | Status |
|------------|----------|--------------|---------------------|--------|
| Immersive Navigation in Google Maps | Cars with Google built-in (Android Automotive OS) | Announced | Front-facing camera | Not confirmed for Android Auto |
| Live lane guidance using front-facing camera | Cars with Google built-in | Announced | Vehicle front-facing camera | Not confirmed for Android Auto |
| 3D buildings and terrain | Google Maps (general) | Released in Google Maps mobile | Not specified | Not confirmed for Android Auto |

**Important:** The Google blog announcement specifically attributes Immersive Navigation with live lane guidance to "eligible cars with Google built-in" using the "vehicle's front-facing camera." It does not confirm these features for Android Auto (phone projection). HaulInfo should not assume camera-based lane guidance, 3D buildings, or immersive terrain rendering are available on Android Auto.

**Unconfirmed:** Whether any Immersive Navigation capabilities will come to Android Auto, and on what timeline.

**Source:** [Google blog — Android in cars updates, May 12, 2026](https://blog.google/products-and-platforms/platforms/android/android-in-cars-updates/)



## 5. Media and dashboard research

### 5.1 Media playback view

The playback view displays media information and playback controls:

- **Back button**
- **Media metadata** — album art, title, optional subtitle, explicit-content indicator
- **Elapsed time**
- **Queue button**
- **Playback controls with visual progress indicator**

**Primary actions (up to 9)**, appearing left to right:
1. Playlist/Queue
2. Previous
3. Play/Pause
4. Next
5. Optional function (far right)

**Secondary actions (up to 4)** — such as thumbs up/down — slide up from the primary actions when the user selects the overflow button.

**Third-party app design guideline.** These are requirements for media apps built with the Android for Cars App Library, defining how the media playback view should look within Android Auto.

**Source:** [Google Design — Playback view](https://developers.google.com/cars/design/android-auto/apps/playback-view)

### 5.2 Dashboard media cards

On the dashboard, media appears as cards alongside the map and other active apps. The music app card can shrink to make room for other information (e.g., an incoming message preview).

The mini-player for playback control remains accessible without opening the full Now Playing interface. Reported in v18.2 hands-on observation (Android Authority, Nov 24, 2026) as a larger now-playing bar with album art and playback controls. This is a third-party observation from a staged rollout, not an officially confirmed Google change.

**Sources:**
- [Google Design — Content area](https://developers.google.com/cars/design/android-auto/product-experience/system-ui/content-area)
- [Android Authority — Nov 24, 2026](https://www.androidauthority.com/android-auto-app-layout-3739081/)

### 5.3 Media app browsing

Media apps use templates including Grid template, List template, and Sectioned Item template for browsing views. Tabs are used to switch views within media apps.

**Third-party app design guideline.**

**Source:** [Google Design — Templates Overview](https://developers.google.com/cars/design/create-apps/media-apps/create-settings)

## 6. App launcher research

### 6.1 Layout

The app launcher displays available apps in a **vertically scrolling grid**. Each app is represented by its icon and title.

**Contents:**
- Apps (navigation, media, communications, and other Android Auto-compatible apps)
- **Exit option** — quits Android Auto and returns to the car's built-in infotainment system; can include the car maker's logo and brand name
- **Digital assistant actions** — app icons that invoke the Assistant for specific voice tasks

**Default layout:** Exit option appears first, before the alphabetical grid of apps. Users can customize the order.

**System UI specification.** Documented in Google's Android Auto design guidelines.

**Source:** [Google Design — App launcher](https://developers.google.com/cars/design/android-auto/product-experience/system-ui/app-launcher)

### 6.2 Screen-size variation

- **Wide screens:** The app launcher opens next to the navigation bar on the side of the screen.
- **Portrait screens:** The app launcher opens above the navigation bar at the bottom of the screen.

**System UI specification.**

**Source:** [Google Design — App launcher](https://developers.google.com/cars/design/android-auto/product-experience/system-ui/app-launcher)

### 6.3 Customisation

Users can:
- Hide certain apps and options
- Change the order in which apps appear

Customisation is done in Android Auto settings on the user's phone (Settings > Connected Devices > Connection Preferences > Android Auto > Customize launcher). Changes take effect after exiting and restarting Android Auto.

**System UI specification.**

**Source:** [Google Design — App launcher](https://developers.google.com/cars/design/android-auto/product-experience/system-ui/app-launcher)


## 7. Notifications research

### 7.1 Notification types

Notifications are designed for glanceability while driving. Each contains:
- App icon
- Avatar (if applicable)
- Primary and secondary text
- Up to 3 action buttons

**Categories:**

| Category | Appears as HUN | Saved to Notification Center |
|----------|----------------|------------------------------|
| Navigation instructions | Yes | No |
| Phone calls | Yes | No |
| Messages | Yes | Yes |
| Reminders | No | Yes |

Navigation and phone call notifications are HUN-only because they are time-sensitive. Messages appear as HUNs first, then move to the Notification Center. Reminders go directly to the Notification Center.

**System UI specification.** Documented in Google's Android Auto design guidelines.

**Source:** [Google Design — Notifications](https://developers.google.com/cars/design/android-auto/product-experience/system-ui/notifications)

### 7.2 Heads-up notifications (HUNs)

A HUN temporarily covers part of the current screen when it arrives. Users can respond using action buttons or dismiss it with the close button. If not acted on, it times out after a short interval and moves to the Notification Center.

**Source:** [Google Design — Notifications](https://developers.google.com/cars/design/android-auto/product-experience/system-ui/notifications)

### 7.3 Notification Center

The Notification Center stores notifications in a vertically scrolling list, ordered by recency (most recent at the top). Users access it from the bell-shaped Notifications button on the navigation bar. Older notifications may be removed to keep the list manageable, but all notifications remain available on the phone.

**Source:** [Google Design — Notifications](https://developers.google.com/cars/design/android-auto/product-experience/system-ui/notifications)



## 8. Widgets research

### 8.1 Existing Android Auto system widgets

Android Auto supports three system widget types for multitasking:

| Widget type | When it appears | What it shows |
|-------------|-----------------|---------------|
| Dialer widget | During a phone call while another app is onscreen | Mute or end the call without switching screens |
| Navigation widget | When navigation is active but the map is not on screen | Turn-by-turn directions for an upcoming turn |
| Media widget | When media is playing but the media app is not on screen | Up to 3 customizable playback controls |

Widgets show the app icon plus information or up to 3 controls. The decision about which widget to show in the nav bar is based on which apps are open and their relative priority for the driver.

**System UI specification.** Documented in Google's Android Auto design guidelines as existing functionality.

**Source:** [Google Design — Widgets](https://developers.google.com/cars/design/android-auto/product-experience/system-ui/widgets)

### 8.2 Upcoming Cielo dashboard widgets

Widgets displayed side-by-side with other dashboard content (on the Coolwalk screen) are part of the unreleased Cielo redesign announced at Google I/O 2026. These are **upcoming**, not released functionality.

**Status:** Announced, not generally available.

**Source:** [Google blog — May 12, 2026](https://blog.google/products-and-platforms/platforms/android/android-in-cars-updates/), [autoevolution — Nov 2026](https://www.autoevolution.com/news/everyone-can-now-update-to-android-auto-182-here-s-how-to-do-it-278698.html)

## 9. Settings research

Android Auto settings are accessed on the user's phone (Settings > Connected Devices > Connection Preferences > Android Auto). App-specific settings are optional on the car screen; the phone already has its own settings experience.

When apps do provide car-screen settings, they should:
- Include only essential settings (account info, app preferences, sign-in/sign-out, media-relevant settings like explicit content)
- Be organized for easy navigation, ideally on a single screen using the List template
- Use templates like Message template for dialogs and error flows

Weather display in the status bar can be toggled on/off in Settings.

**Third-party app design guideline.** These are recommendations for app developers creating settings within their Android Auto apps, not a specification of the Android Auto system settings UI.

**Source:** [Google Design — Create settings](https://developers.google.com/cars/design/create-apps/media-apps/design-settings)

## 10. Animations and interactions

### 10.1 Dashboard transitions

- Users enter expanded view by tapping a card, toast, or notification.
- Users return to the dashboard by tapping the dashboard icon.
- The dashboard layout adjusts dynamically as events occur (e.g., music area shrinks when a message arrives).

### 10.2 Cielo redesign animations (unreleased, observed in v18.1 teardown)

Animation strings observed in Android Auto 18.1 teardown analysis:

- `cielo_map_scale_out.xml` — map scaling animation
- `cielo_dashboard_scale_fade_enter_widescreen.xml` — dashboard scale/fade enter on widescreen
- `cielo_dashboard_slide_out_to_left.xml` — dashboard slide out to left
- `cielo_dashboard_slide_in_from_right.xml` — dashboard slide in from right
- `cielo_launcher_no_op_exit.xml` — launcher no-op exit
- `cielo_launcher_app_scale_enter.xml` — launcher app scale enter
- `cielo_notification_center_app_scale_fade_in.xml` — notification center app scale/fade in

These animation strings were observed in APK teardown analysis (9to5Google, Nov 3, 2026) and were seen in early hands-on demos in May 2026. They are **not yet user-facing** as of November 25, 2026.

**Observed implementation detail (unreleased).** These are implementation strings found in a beta/teardown analysis, not confirmed released functionality.

**Source:** [9to5Google — Nov 3, 2026](https://9to5google.com/2026/11/03/android-auto-18-1-update-rollout-teardown/)

### 10.3 Secondary media actions

Secondary actions (up to 4) slide up from the primary playback controls when the user selects the overflow button.

**Third-party app design guideline.**

**Source:** [Google Design — Playback view](https://developers.google.com/cars/design/android-auto/apps/playback-view)

## 11. Responsive layout research

Android Auto uses an **adaptive-responsive (AR) framework** that adapts app layouts to any car screen size.

### 11.1 Breakpoints

Breakpoints are calculated from the app window size (not the entire screen):

| Class | Width range (dp) |
|-------|------------------|
| Narrow | 0–600dp |
| Standard | 600–930dp |
| Wide | 930–1280dp |
| Extra wide | 1280dp+ |

**System UI specification.** Documented in Google's Android Auto design guidelines.

**Source:** [Google Design — Layout](https://developers.google.com/cars/design/android-auto/design-system/layout)

### 11.2 Responsive margins

Left and right margins adjust proportionally to 12% of the screen width. The remaining space is the app canvas. On smaller screens, the right-side margin can be removed to provide more space.

**System UI specification.** Documented in Google's Android Auto design guidelines.

**Source:** [Google Design — Layout](https://developers.google.com/cars/design/android-auto/design-system/layout)

### 11.3 Layout grid

UI elements align to an **8dp grid**; smaller components may use a **4dp grid**.

**System UI specification.** Documented in Google's Android Auto design guidelines.

**Source:** [Google Design — Layout](https://developers.google.com/cars/design/android-auto/design-system/layout)

### 11.4 Padding

Padding is applied in multiples of 8dp (P0–P8):

| Label | Value |
|-------|-------|
| P0 | 4dp |
| P1 | 8dp |
| P2 | 12dp |
| P3 | 16dp |
| P4 | 24dp |
| P5 | 32dp |
| P6 | 48dp |
| P7 | 64dp |
| P8 | 96dp |

Smaller components may use 12dp padding sparingly.

**System UI specification.** Documented in Google's Android Auto design guidelines.

**Source:** [Google Design — Layout](https://developers.google.com/cars/design/android-auto/design-system/layout)

### 11.5 Keylines

Keylines organize horizontal element placement and change based on screen width:

| Keyline | Narrow (0–600dp) | Standard (600–930dp) | Wide (930–1280dp) | Extra wide (1280dp+) |
|---------|------------------|----------------------|-------------------|----------------------|
| KL0 | 16dp | 24dp | 24dp | 32dp |
| KL1 | 24dp | 32dp | 32dp | 48dp |
| KL2 | 96dp | 112dp | 112dp | N/A |
| KL3 | 112dp | 128dp | 128dp | 152dp |
| KL4 | 148dp | 168dp | 168dp | N/A |

**System UI specification.** Documented in Google's Android Auto design guidelines.

**Source:** [Google Design — Layout](https://developers.google.com/cars/design/android-auto/design-system/layout)

### 11.6 Screen orientation

Car infotainment screens are in a fixed orientation — either landscape or portrait. Apps must support both.

**Source:** [Android Developers — Build adaptive apps for cars](https://developer.android.com/develop/adaptive-apps/guides/cars/build-adaptive-apps-for-cars)

### 11.7 Other considerations

- Gesture navigation is not supported; back affordances may vary.
- Some vehicles have non-rectangular or irregularly shaped screens with cutouts.
- Car manufacturers can control whether apps can show/hide system bars or enter immersive mode.
- Cars often have intermittent or no internet connectivity; apps should remain functional offline.
- Apps are primarily accessible when the car is parked; UI should be optimized for quick, clear interactions.

**Source:** [Android Developers — Build adaptive apps for cars](https://developer.android.com/develop/adaptive-apps/guides/cars/build-adaptive-apps-for-cars)

## 12. Color and visual design

### 12.1 Color palette

- **Background:** Black, used consistently across day and night themes. Works well with dark car interiors.
- **Primary palette:** Grayscale, with minimal and purposeful use of muted additional colors.
- **Accent color:** One official accent color — a saturated shade of blue referred to as "car accent."
- **Contrast:** Minimum 4.5:1 ratio between foreground and background elements for legibility while driving.

**System UI specification.** Documented in Google's Android Auto design guidelines.

**Source:** [Google Design — Color](https://developers.google.com/cars/design/android-auto/design-system/color)

### 12.2 Elevation and depth

Elevation is expressed using different shades of gray on the black background (shadows are not visible on true black). Darker shades indicate less elevation; lighter shades indicate more elevation.

**Source:** [Google Design — Color](https://developers.google.com/cars/design/android-auto/design-system/color)

### 12.3 Text opacity

- Night mode text: 96% opacity white
- Visual hierarchy: 88%, 72%, and 56% white opacity for different levels of information importance

**Source:** [Google Design — Color](https://developers.google.com/cars/design/android-auto/design-system/color)

### 12.4 Color continuity

The same color is used for an item across multiple views (e.g., green for turn-by-turn navigation). App-dominant colors (e.g., Spotify green) can be used on related elements.

**Source:** [Google Design — Color](https://developers.google.com/cars/design/android-auto/design-system/color)

## 13. Released vs upcoming features comparison

### Released (generally available as of Nov 25, 2026) — HaulInfo should recreate these first

| Feature | Status | Source |
|---------|--------|--------|
| Dashboard with map + cards | Released | Google Design — Content area |
| Navigation bar (bottom/side, horizontal/vertical) | Released | Google Design — Navigation bar |
| Status bar (clock, weather, connectivity, battery) | Released | Google Design — Status bar |
| App launcher (vertical grid, customizable) | Released | Google Design — App launcher |
| Notifications (HUN + Notification Center) | Released | Google Design — Notifications |
| Media playback view (controls, metadata, progress) | Released | Google Design — Playback view |
| Media widgets (up to 3 controls) | Released | Google Design — Widgets |
| Navigation widget (turn-by-turn) | Released | Google Design — Widgets |
| Dialer widget (mute/end call) | Released | Google Design — Widgets |
| Side navigation in media apps and dialer (v18.2 reported) | Observed (third-party) — not officially confirmed | Android Authority — Nov 24, 2026; autoevolution — Nov 25, 2026 |
| Mini-player / larger now-playing bar (v18.2 reported) | Observed (third-party) — not officially confirmed | Android Authority — Nov 24, 2026 |
| 8dp layout grid, breakpoints, responsive margins | Released | Google Design — Layout |

### Announced / rolling out / beta (NOT for initial HaulInfo version)

| Feature | Status | Expected timeline | Source |
|---------|--------|-------------------|--------|
| Cielo redesign (widgets side-by-side with dashboard) | Announced, in beta builds | Beta end of 2026; production Q1 2027 | Google blog May 2026; autoevolution Nov 2026 |
| Video apps (YouTube, parked only) | Announced | Not yet released | Google blog May 2026; autoevolution Nov 2026 |
| Browser support (Chrome, Vivaldi, parked only) | Announced | Not yet released | autoevolution Nov 2026 |
| Light mode | In development (18.1 teardown) | Not yet released | 9to5Google Nov 3, 2026 |
| Quick Settings tiles | In development (18.1 teardown) | Not yet released | 9to5Google Nov 3, 2026 |
| Gemini integration (Magic Cue, Doordash) | Announced | Throughout 2026 | Google blog May 2026 |
| Immersive Navigation / live lane guidance (cars with Google built-in) | Announced | Not confirmed for Android Auto | Google blog May 2026 |

## 14. Important conclusions for HaulInfo

1. **Reference version:** Android Auto 18.2 is the latest released version as of November 25, 2026. HaulInfo should use this as its primary reference.

2. **Dashboard-first design:** The dashboard (minimized view) is the central experience — map + cards + navigation bar always visible. HaulInfo should recreate this before building full-screen expanded views.

3. **Navigation bar is always visible:** Whether bottom or side, horizontal or vertical, the navigation bar never disappears. HaulInfo must account for this in its layout system.

4. **Side navigation on wide screens:** Android Auto already supports a vertical system navigation bar on widescreen displays. Since HaulInfo targets Windows desktop (which will have wide displays), this is directly relevant. Reported v18.2 changes to app-internal side navigation are third-party observations and not officially confirmed.

5. **Responsive layout system:** Android Auto's breakpoints (narrow/standard/wide/extra wide) and 8dp grid provide a concrete framework HaulInfo can adapt for Windows display sizes.

6. **No custom styling:** Android Auto uses a black background, grayscale palette, and a single blue accent color. HaulInfo should follow this exactly rather than inventing its own design.

7. **Widgets:** Android Auto already has three released system widgets (Dialer, Navigation, Media). The Cielo dashboard widget redesign (widgets side-by-side with dashboard content) is announced but not generally available. HaulInfo should use the existing system widgets, not the unreleased Cielo dashboard widgets, in its initial version.

8. **Notifications are HUN-first:** Notifications appear as heads-up banners first, then move to a Notification Center. Navigation and call notifications are HUN-only.

9. **Media and navigation coexist:** The dashboard shows navigation and media simultaneously. The mini-player allows music control without leaving navigation.

10. **Offline operation matters:** Android Auto documentation notes that cars often have intermittent connectivity. HaulInfo's offline-first map approach aligns well with this principle.

## Sources

- [Google Design for Driving — Android Auto System UI overview](https://developers.google.com/cars/design/android-auto/product-experience/system-ui/overview)
- [Google Design for Driving — Navigation bar](https://developers.google.com/cars/design/android-auto/product-experience/system-ui/nav-bar)
- [Google Design for Driving — Content area](https://developers.google.com/cars/design/android-auto/product-experience/system-ui/content-area)
- [Google Design for Driving — Status bar](https://developers.google.com/cars/design/android-auto/product-experience/system-ui/status-bar)
- [Google Design for Driving — App launcher](https://developers.google.com/cars/design/android-auto/product-experience/system-ui/app-launcher)
- [Google Design for Driving — Notifications](https://developers.google.com/cars/design/android-auto/product-experience/system-ui/notifications)
- [Google Design for Driving — Widgets](https://developers.google.com/cars/design/android-auto/product-experience/system-ui/widgets)
- [Google Design for Driving — Playback view](https://developers.google.com/cars/design/android-auto/apps/playback-view)
- [Google Design for Driving — Color](https://developers.google.com/cars/design/android-auto/design-system/color)
- [Google Design for Driving — Layout](https://developers.google.com/cars/design/android-auto/design-system/layout)
- [Google Design for Driving — Create settings](https://developers.google.com/cars/design/create-apps/media-apps/design-settings)
- [Google Design for Driving — Templates Overview](https://developers.google.com/cars/design/create-apps/media-apps/create-settings)
- [Android Developers — Build adaptive apps for cars](https://developer.android.com/develop/adaptive-apps/guides/cars/build-adaptive-apps-for-cars)
- [Google Blog — Android in cars updates, May 12, 2026](https://blog.google/products-and-platforms/platforms/android/android-in-cars-updates/)
- [9to5Google — Android Auto 18.1 update, Nov 3, 2026](https://9to5google.com/2026/11/03/android-auto-18-1-update-rollout-teardown/)
- [autoevolution — Android Auto 18.2, Nov 20, 2026](https://www.autoevolution.com/news/everyone-can-now-update-to-android-auto-182-here-s-how-to-do-it-278698.html)
- [autoevolution — Android Auto 18.2 wide screen navigation, Nov 25, 2026](https://www.autoevolution.com/news/android-auto-182-starts-rolling-out-with-a-big-navigation-change-for-wide-screens-278950.html)
- [Android Authority — Android Auto app nav layout for wide displays, Nov 24, 2026](https://www.androidauthority.com/android-auto-app-layout-3739081/)






