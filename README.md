# Tom Player — Professional Lightweight Android Video & Audio Player

[![Platform](https://img.shields.io/badge/Platform-Android-green.svg)](https://developer.android.com)
[![Language](https://img.shields.io/badge/Language-Kotlin-purple.svg)](https://kotlinlang.org)
[![Engine](https://img.shields.io/badge/Media%20Engine-AndroidX%20Media3%20ExoPlayer-blue.svg)](https://developer.android.com/media/media3)
[![UI](https://img.shields.io/badge/Theme-AMOLED%20Pure%20Black-black.svg)](#ui-and-design-system)
[![License](https://img.shields.io/badge/License-Apache%202.0-lightgrey.svg)](LICENSE)

**Tom Player** is a high-performance, lightweight, pure AMOLED offline video and audio player for Android. Built with native **Kotlin** and **AndroidX Media3 (ExoPlayer + MediaSessionService)**, it delivers seamless background audio playback, video-to-audio transitions without restarting playback, media notification controls, touch gestures, Picture-in-Picture, and external subtitle support — completely offline with zero tracking, zero ads, and minimal battery consumption.

---

## Table of Contents

1. [Key Features](#key-features)
2. [Architecture & Design Principles](#architecture--design-principles)
3. [Technology Stack](#technology-stack)
4. [Video-to-Audio Mode](#video-to-audio-mode)
5. [UI & Design System](#ui--design-system)
6. [Supported Media Formats](#supported-media-formats)
7. [Android Storage & Permissions](#android-storage--permissions)
8. [Project Structure](#project-structure)
9. [Build Instructions](#build-instructions)
10. [Installing on a Physical Android Phone](#installing-on-a-physical-android-phone)
11. [Comprehensive QA Testing Checklist](#comprehensive-qa-testing-checklist)
12. [Resource & Memory Optimization](#resource--memory-optimization)

---

## Key Features

- **True Background Playback**: Audio continues smoothly when switching apps, pressing Home, or turning off the screen.
- **Instant Video-to-Audio Switching**: Switch video playback directly into background audio without interrupting or restarting playback.
- **Full Media3 Notification & Lockscreen Controls**: Native Android media notification controls with seek bar, play/pause, skip, and metadata display.
- **Picture-in-Picture (PiP)**: Smooth PiP transition on Android 8.0+ (API 26+) with auto-pip support on user leave.
- **On-Screen Touch Gestures**:
  - Vertical swipe on left side: Screen brightness HUD (0% – 100%).
  - Vertical swipe on right side: Volume HUD with mute indicator.
  - Double-tap left: Configurable rewind HUD (e.g. -10s).
  - Double-tap right: Configurable forward HUD (e.g. +10s).
  - Single tap: Reveal / auto-hide controls with inactivity timer.
  - Screen Lock: Prevent accidental touches with floating unlock button.
- **Subtitles & Audio Tracks**:
  - Embedded subtitle and audio track switching.
  - External **SRT** file loader via Android Storage Access Framework (SAF).
- **Speed Selector**: 0.5x, 0.75x, 1.0x, 1.25x, 1.5x, and 2.0x playback speed with pitch preservation.
- **Aspect Ratio Control**: Fit to Screen, Fill Screen (Crop), and 100% Original.
- **Dedicated Audio Player**: AMOLED album artwork card, track queue, shuffle, repeat (Off, All, One), and Sleep Timer (15, 30, 45, 60 min).
- **Persistent Mini-Player**: Bottom bar synchronized across the library tabs for quick play/pause, progress, and expanding to full player.
- **Folder & Storage Browsing**: MediaStore indexed bucket folders + SAF document picker for opening any media file on device.
- **Resume Playback**: Automatically remembers and restores playback positions per media file.

---

## Architecture & Design Principles

```
┌────────────────────────────────────────────────────────┐
│                      UI Layer                          │
│  MainActivity  •  VideoPlayerActivity  •  AudioPlayer  │
│  MediaListFragment  •  FolderBrowser  •  MiniPlayer   │
└───────────────────────────▲────────────────────────────┘
                            │ (StateFlow / MediaController)
┌───────────────────────────▼────────────────────────────┐
│                  Player Connection                     │
│               PlayerConnection (Singleton)             │
└───────────────────────────▲────────────────────────────┘
                            │ (MediaSession token)
┌───────────────────────────▼────────────────────────────┐
│                  Playback Service                      │
│            PlaybackService (MediaSessionService)        │
│          owns single ExoPlayer + MediaSession          │
└───────────────────────────▲────────────────────────────┘
                            │
┌───────────────────────────▼────────────────────────────┐
│                  Media Repository                      │
│        MediaStore (Scoped Storage) + SAF Resolvers     │
└────────────────────────────────────────────────────────┘
```

1. **Single Source of Truth**: The playback engine is exclusively owned by `PlaybackService` extending `MediaSessionService`. Activities and fragments never instantiate duplicate players; they connect to the service via `MediaController`.
2. **Audio Focus & Becoming Noisy**: Automatically pauses when headphones are unplugged (`handleAudioBecomingNoisy = true`) and handles audio interruptions (e.g. incoming phone calls).
3. **Decoupled UI**: Views observe player state via Kotlin Coroutines `StateFlow<PlaybackState>`.

---

## Technology Stack

- **Language**: Kotlin 2.0
- **Android Target**: Compile SDK 35, Target SDK 35, Min SDK 24 (compatible with Android 7.0 through Android 15)
- **Media Engine**: AndroidX Media3 ExoPlayer 1.5.1
- **Media Session**: AndroidX Media3 Session 1.5.1
- **Media UI**: AndroidX Media3 UI 1.5.1
- **Build System**: Gradle 8.11 with Kotlin DSL (`build.gradle.kts`) and version catalog (`libs.versions.toml`)
- **Architecture Components**: AndroidX Lifecycle, ViewModel, Fragment KTX, Activity KTX, ViewBinding, Preference KTX
- **Concurrency**: Kotlin Coroutines (`kotlinx-coroutines-android`)
- **Zero Third-Party Image Bloat**: In-memory LRU cache with Android native `ContentResolver.loadThumbnail` (API 29+) and `MediaMetadataRetriever` fallback.

---

## Video-to-Audio Mode

### How it Works
1. When watching a video in `VideoPlayerActivity`, the user taps the **Play as Audio** action button (or exits while background audio is enabled).
2. The UI safely detaches the video surface: `playerView.player = null`.
3. The underlying `ExoPlayer` inside `PlaybackService` continues decoding and playing audio uninterrupted. No transcoding, no restarting, and zero lost playback position.
4. The user can switch to `AudioPlayerActivity`, navigate the media library with the mini-player, open other applications, or lock their phone.
5. Returning to `VideoPlayerActivity` attaches the `playerView.player = player` again, seamlessly rendering video frames from the exact current position.

---

## UI & Design System

Designed specifically for **AMOLED** smartphone displays to minimize power draw:

| Token | Color Code | Usage |
|---|---|---|
| Background | `#000000` | Pure AMOLED black across all screens |
| Surface | `#0A0A0A` | App bars and bottom navigation background |
| Card Surface | `#141414` | Media cards, dialogs, and HUD panels |
| Border | `#2A2A2A` | Subtle division lines and card borders |
| Accent Primary | `#00E5FF` | Cyan / vibrant teal for seekbars, active buttons |
| Accent Secondary | `#00B4D8` | Deep teal for secondary badges |
| Playing Indicator | `#00E676` | Vibrant emerald green dot on active item |
| Primary Text | `#FFFFFF` | High-contrast readable typography |
| Secondary Text | `#A0A0A0` | Subtitles, artists, file sizes, and dates |

---

## Supported Media Formats

Utilizes Android's hardware decoders and AndroidX Media3 default extractors:

- **Video Containers**: MP4, MKV, WebM, 3GP, TS, M4V, FLV
- **Video Codecs**: H.264 (AVC), H.265 (HEVC), VP8, VP9, AV1 (device dependent)
- **Audio Containers & Codecs**: MP3, AAC, M4A, FLAC, WAV, OGG (Vorbis/Opus), AMR
- **Subtitles**: SubRip (`.srt`), WebVTT (`.vtt`), embedded MKV/MP4 text tracks

*Note: Codec support depends on the physical hardware and Android OS decoders present on the device.*

---

## Android Storage & Permissions

Tom Player strictly adheres to modern Android Scoped Storage best practices:

- **Android 13+ (API 33+)**:
  - `READ_MEDIA_VIDEO` — to index and display local videos.
  - `READ_MEDIA_AUDIO` — to index and display music.
  - `POST_NOTIFICATIONS` — to display media playback controls in the notification shade.
- **Android 7.0 – Android 12 (API 24–32)**:
  - `READ_EXTERNAL_STORAGE` (with `maxSdkVersion="32"`).
- **Storage Access Framework (SAF)**:
  - When users open custom files via the "Open File" button or load external subtitles, Android's built-in file picker (`OpenDocument`) is used without requiring broad filesystem access.

---

## Project Structure

```
Tom Player/
├── build.gradle.kts
├── settings.gradle.kts
├── gradlew
├── gradlew.bat
├── gradle/
│   ├── libs.versions.toml
│   └── wrapper/
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
└── app/
    ├── build.gradle.kts
    ├── proguard-rules.pro
    └── src/
        ├── main/
        │   ├── AndroidManifest.xml
        │   ├── java/com/tomjit/tomplayer/
        │   │   ├── TomPlayerApplication.kt
        │   │   ├── MainActivity.kt
        │   │   ├── player/
        │   │   │   ├── PlaybackService.kt
        │   │   │   ├── PlayerConnection.kt
        │   │   │   └── PlaybackState.kt
        │   │   ├── media/
        │   │   │   ├── MediaItemModel.kt
        │   │   │   ├── MediaRepository.kt
        │   │   │   └── SortOrder.kt
        │   │   ├── ui/
        │   │   │   ├── home/
        │   │   │   │   └── MediaListFragment.kt
        │   │   │   ├── video/
        │   │   │   │   └── VideoPlayerActivity.kt
        │   │   │   ├── audio/
        │   │   │   │   └── AudioPlayerActivity.kt
        │   │   │   ├── folders/
        │   │   │   │   └── FolderBrowserFragment.kt
        │   │   │   ├── settings/
        │   │   │   │   └── SettingsFragment.kt
        │   │   │   ├── adapters/
        │   │   │   │   ├── MediaAdapter.kt
        │   │   │   │   ├── FolderAdapter.kt
        │   │   │   │   └── QueueAdapter.kt
        │   │   │   └── dialogs/
        │   │   │       ├── TrackSelectionDialog.kt
        │   │   │       ├── SpeedSelectionDialog.kt
        │   │   │       ├── QueueBottomSheetDialog.kt
        │   │   │       └── SleepTimerDialog.kt
        │   │   ├── preferences/
        │   │   │   └── PlaybackPreferences.kt
        │   │   └── utils/
        │   │       ├── FormatUtils.kt
        │   │       ├── PermissionUtils.kt
        │   │       └── ArtworkUtils.kt
        │   └── res/
        │       ├── drawable/ (35+ vector icons and shape assets)
        │       ├── layout/ (13 XML layouts)
        │       ├── menu/ (bottom navigation)
        │       ├── mipmap-anydpi-v26/ (adaptive icons)
        │       ├── values/ (colors, strings, themes, arrays)
        │       ├── values-night/ (AMOLED dark theme)
        │       └── xml/ (preferences schema)
        └── test/
            └── java/com/tomjit/tomplayer/
                ├── FormatUtilsTest.kt
                └── SortOrderTest.kt
```

---

## Build Instructions

### Prerequisites
1. **Android Studio**: Android Studio Hedgehog (2023.1.1) or newer (Ladybug / Koala recommended).
2. **JDK**: Java Development Kit 17 (bundled automatically with Android Studio).
3. **Android SDK**: Android SDK Platform 35 and Build Tools installed.

### Option A: Building with Android Studio
1. Launch **Android Studio**.
2. Select **Open** and choose the `Tom Player` directory (`c:\Users\tomjit\Documents\Tom Player`).
3. Allow Gradle to sync dependencies.
4. Select build variant `debug` or `release` in the **Build Variants** panel.
5. Click **Build > Build Bundle(s) / APK(s) > Build APK(s)**.
6. The generated APK will be located in:
   `app/build/outputs/apk/debug/app-debug.apk` or `app/build/outputs/apk/release/app-release-unsigned.apk`.

### Option B: Building via Command Line (Gradle)
From the root project directory:

**Windows PowerShell:**
```powershell
.\gradlew.bat assembleDebug
```

**macOS / Linux:**
```bash
./gradlew assembleDebug
```

To run unit tests:
```powershell
.\gradlew.bat testDebugUnitTest
```

---

## Installing on a Physical Android Phone

### Method 1: Direct Run from Android Studio
1. Enable **Developer Options** on your Android phone:
   - Go to **Settings > About Phone**.
   - Tap **Build Number** 7 times until you see *"You are now a developer!"*.
2. Enable **USB Debugging**:
   - Go to **Settings > System > Developer Options**.
   - Toggle **USB Debugging** on.
3. Connect your phone to your computer via USB.
4. Accept the *"Allow USB debugging?"* prompt on your phone screen.
5. In Android Studio, select your phone in the device dropdown and press the green **Run** (▶) button.

### Method 2: Manual Installation via ADB
```bash
adb install -r app/build/outputs/apk/debug/app-debug.apk
```

### Method 3: Direct APK Transfer
1. Copy `app-debug.apk` to your phone's storage via USB cable, Google Drive, or local file sharing.
2. Open your phone's **Files** app and tap `app-debug.apk`.
3. If prompted, allow your file manager permission to *"Install unknown apps"*.
4. Tap **Install** and open **Tom Player**.

---

## Comprehensive QA Testing Checklist

Use this checklist to verify application functionality on device:

- [ ] **1. Permission Flow**:
  - Launch app on Android 13+. Verify permission dialog for Photos & Videos and Audio appears.
  - Deny permission: Verify clear empty state displays with "Grant Permission" and "Open File" buttons.
  - Grant permission: Verify media library immediately indexes and displays device media.
- [ ] **2. Video Playback**:
  - Tap a video in Videos tab: Fullscreen player opens and plays video smoothly.
  - Tap screen: Controls appear; wait 3.5s: controls automatically hide.
  - Tap Play/Pause: Video pauses and resumes instantly.
- [ ] **3. Video Gestures**:
  - Swipe vertically on left side of screen: Brightness HUD appears and adjusts screen brightness.
  - Swipe vertically on right side of screen: Volume HUD appears and adjusts music volume.
  - Double-tap left side: Video rewinds by 10s and displays "-10 sec" HUD.
  - Double-tap right side: Video forwards by 10s and displays "+10 sec" HUD.
- [ ] **4. Video-to-Audio Mode**:
  - While video is playing, tap the **Play as Audio** headphone icon in the top bar.
  - Video screen finishes; audio continues playing without interruption or restart.
  - Check persistent mini-player on Home screen: shows active title and progress.
- [ ] **5. Background Playback & Screen Off**:
  - While audio is playing (either music or video in audio mode), press the Home button.
  - Verify audio continues playing while using other apps.
  - Press phone power button to turn off screen. Verify audio continues playing uninterrupted.
- [ ] **6. Media Notification & Lock Screen**:
  - Pull down notification shade while playing.
  - Verify media player card shows: Title, Artist, Play/Pause toggle, Previous/Next, and scrubber.
  - Tap Play/Pause in notification: playback toggles correctly.
- [ ] **7. Headphone Disconnect**:
  - While playing audio or video with headphones plugged in or connected via Bluetooth, disconnect them.
  - Verify playback automatically pauses immediately (`handleAudioBecomingNoisy`).
- [ ] **8. Playback Speed Selector**:
  - Tap Speed button (1.0x) in video or audio player.
  - Select 1.5x or 2.0x. Verify audio/video speeds up with correct pitch.
- [ ] **9. External Subtitles**:
  - Tap Subtitles icon on video player > "Load external SRT file...".
  - Select an `.srt` file via SAF picker. Verify subtitles render on the video.
- [ ] **10. Picture-in-Picture (PiP)**:
  - While playing video, tap PiP button or swipe to Home.
  - Verify floating PiP window appears on home screen with synchronized playback.
- [ ] **11. Resume Playback**:
  - Play video or audio halfway, then close the app.
  - Re-open and select the same file: playback resumes from the saved position.
- [ ] **12. Sleep Timer**:
  - In Audio Player, tap Timer icon > select 15 minutes.
  - Verify toast confirms sleep timer.
- [ ] **13. Queue Management**:
  - In Audio Player, open Queue bottom sheet.
  - Verify active item is marked with green dot; removing or tapping queue items updates playback.
- [ ] **14. Pure AMOLED Theme**:
  - Inspect UI on an OLED device: Verify background is `#000000` with high-contrast text and zero light bleed.

---

## Resource & Memory Optimization

- **Zero Network Traffic**: No analytics, telemetry, crash reporting, ad SDKs, or account login libraries are included.
- **Minimal RAM Footprint**: Video decoders and surfaces are detached when inactive. The in-memory bitmap thumbnail cache is bounded to 8MB.
- **Battery Friendly**: Native AMOLED black pixels turn off OLED pixels completely, reducing screen battery drain by up to 60%.
- **No Background Battery Drain**: The `PlaybackService` stops itself cleanly as soon as playback stops or the task is cleared.
