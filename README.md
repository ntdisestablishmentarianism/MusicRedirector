# 🎵 MusicRedirector

An Android app that silently intercepts music links from **any** platform and redirects them to your preferred music app — no manual copying, no switching apps.

---

## How It Works

```
User clicks a YouTube Music link
        ↓
Android routes it to RedirectActivity (transparent, invisible)
        ↓
Calls Odesli API → finds the same song on Spotify (or any target)
        ↓
Fires an Intent directly into Spotify
        ↓
Spotify opens at the right song. Done.
```

The Odesli (song.link) API is free, requires no key, and supports all major platforms.

---

## Supported Platforms

| Platform       | Intercept Links | Redirect Target |
|----------------|:--------------:|:---------------:|
| Spotify        | ✅              | ✅               |
| YouTube Music  | ✅              | ✅               |
| Apple Music    | ✅              | ✅               |
| SoundCloud     | ✅              | ✅               |
| Tidal          | ✅              | ✅               |
| Deezer         | ✅              | ✅               |
| Amazon Music   | ✅              | ✅               |
| Pandora        | ✅              | ✅               |
| song.link      | ✅              | —               |

---

## Project Structure

```
MusicRedirector/
├── app/src/main/
│   ├── AndroidManifest.xml          # Intent filters for all music URLs
│   └── java/com/musicredirector/
│       ├── MainActivity.kt          # Settings UI
│       ├── RedirectActivity.kt      # Core redirect logic (transparent)
│       ├── OdesliService.kt         # Odesli API client
│       ├── MusicApp.kt              # Platform definitions & URL detection
│       └── Prefs.kt                 # SharedPreferences wrapper
└── app/src/main/res/
    ├── layout/activity_main.xml
    ├── values/themes.xml
    └── drawable/                    # UI assets
```

---

## Setup Instructions

### 1. Build & Install
```bash
./gradlew assembleDebug
adb install app/build/outputs/apk/debug/app-debug.apk
```

### 2. Configure Default App Links (Critical Step)

Android will only route links to MusicRedirector if you grant it permission per domain:

1. Open **Settings → Apps → MusicRedirector**
2. Tap **Open by default** (or "Set as default")
3. Tap **Add link** and enable all music domains:
   - `open.spotify.com`
   - `music.youtube.com`
   - `music.apple.com`
   - `soundcloud.com`
   - `tidal.com`
   - `deezer.com` / `www.deezer.com`
   - `music.amazon.com`
   - `pandora.com`

> **Tip:** The app's Settings screen has a button that takes you directly to Default Apps.

### 3. Choose Your Target App
Open MusicRedirector and select your preferred app from the dropdown.

---

## Key Design Decisions

### Transparent RedirectActivity
`RedirectActivity` uses `Theme.Transparent` — it has no UI and is completely invisible. The user never sees it; they just see their chosen music app open instantly.

### Same-App Bypass
If the link already points to the user's chosen app, it's opened directly without an API call. This avoids unnecessary network round trips.

### Fallback Chain
1. Odesli API → native app URI (deeplink)
2. Odesli API → web URL for target app
3. Original URL if Odesli fails
4. Toast notification explaining the fallback

### Anti-Loop Protection
`openUrlDirectly()` sets the package to `null` and uses `createChooser()` so the system doesn't re-route back to MusicRedirector.

---

## Extending the App

### Add a New Platform
1. Add an entry to `MusicApp.kt` with the Odesli API key
2. Add an `<intent-filter>` in `AndroidManifest.xml` for its domain/scheme
3. Add a detection case in `MusicApp.detectFromUrl()`

### Add a Notification
In `RedirectActivity`, replace `showToast()` with a heads-up notification for richer feedback.

### Cache Results
Wrap `OdesliService.resolve()` with a simple LRU cache keyed on the input URL to avoid duplicate API calls for the same song.

---

## Permissions

| Permission | Why |
|---|---|
| `INTERNET` | Odesli API calls |
| `QUERY_ALL_PACKAGES` | Check if target app is installed |

---

## API Rate Limits

Odesli free tier: **10 requests/minute**. This is plenty for personal use. For higher volume, register for a free API key at https://odesli.co and add it as a query parameter: `&key=YOUR_KEY`.
