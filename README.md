<h1 div align="center"> KinitoPET
<div align="center"> A desktop based on KinitoPET on Steam

</div><img width="228" height="228" alt="Kinito睡觉" src="https://github.com/user-attachments/assets/84e04d7b-4bc3-49a3-962b-0a056f16fb6f" />
<h3 align="center">  <blockquote> Just a harmless Kinito desktop sleeping on your desk</blockquote> </center>

<h2 div align="center"> Kinito Desktop Pet

<h3 div align="center">A fan-made desktop companion inspired by **KinitoPET** — a cheerful little axolotl that walks around your screen, talks to you, and now also lives on your phone as a real floating overlay.

No AI. No screen capture. No cloud. Just a small pink friend.

</div><img width="640" height="360" alt="image" src="https://github.com/user-attachments/assets/adedcaef-1c40-47a4-a21f-09525ece5a36" />
<h3 align="center">  <blockquote> This is a screenshot taken on my own computer, but Kinito appears to be sleeping right now</blockquote> </center>

<img width="310" height="660" alt="22dfb7fed72e413095869cd489bf500f" src="https://github.com/user-attachments/assets/42055f84-fbc9-442b-8c50-8114e37e2926" />
<h3 align="center">  <blockquote> This is a picture taken on my own phone, and he is currently looking at you</blockquote> </center>

*Kinito running as a real floating overlay on a phone. The home-screen wallpaper is by a creator on **Redbook** and is not part of this project.*

> **Fan project.** Not affiliated with troy_en or the KinitoPET team. All artwork and voice lines in this repository are original fan-made assets. No assets from the original game are redistributed.

---

<h2 div align="center">Two builds, one personality<h3 div align="center">

|              |          **Windows desktop**          |               **Android**               |
|--------------|---------------------------------------|-----------------------------------------|
|    Window    | Transparent, frameless, always-on-top |  Native floating overlay above all apps |
|   Movement   |  Free 2D roaming, idle pauses, sleep  | Free 2D roaming across the whole screen |
|    Talking   |      Speech bubbles + voice clips     |  Floating bubble window + voice clips   |
|     Input    |  Double-click, drag, right-click menu |          Tap, drag, long-press          |
| File actions |    Optional, opt-in, confirm-first    |              Not included               |
|    Stack     |       Python · PySide6 · SQLite       | Java · Android SDK · Foreground Service |
| Min version  |               Windows 10              |          Android 6.0 (API 21)           |

There is also a **single-file web build** (`index.html`) for phones and tablets that cannot install APKs. It runs entirely offline, but a web page cannot float above other apps.

---

<h2 div align="center"> Features

<h2 div align="center"> Shared behaviour

- Walks to random points, pauses to look around, falls asleep when left alone (60 s)
- Speech bubbles that stay up for the length of the voice clip
- Voice clips are optional — any line without audio still shows as text
- First launch greets you with a one-time welcome line
- Mute toggle, remembered between sessions

<h2 div align="center"> Windows desktop

- Fully transparent, draggable, always-on-top pet
- Walks with a single walk pose (no frame animation), facing flips automatically
- Right-click menu and system tray menu:
  - Say something
  - Let Kinito sleep / wake up
  - Scan files now
  - Undo last file action
  - Forbid dragging files
  - Forbid deleting files
  - Start with Windows
  - Always on top
  - Mute Kinito
  - Open data folder
  - Quit
- Optional "tidying" behaviour: a small hand + file icon appears in its own floating window, drifting gently
- Single-instance guard: double-clicking the shortcut will not spawn duplicates

<h2 div align="center"> Android

- **Real floating overlay** (`TYPE_APPLICATION_OVERLAY`) that stays above other apps
- Foreground service with a persistent notification, so the system does not kill it
- Tap to talk, drag to move, long-press to sleep / wake
- Bubble is a separate click-through window and flips below the pet near the screen top
- **No file features** — the mobile build cannot move or delete anything

---

<h1 div align="center"> Install

<h2 div align="center"> Windows

1. Download the latest `KinitoPet-win.zip` from Releases
2. Extract the folder anywhere (it is portable — settings and data live next to the exe)
3. Run `KinitoPet.exe`
4. If SmartScreen appears: **More info → Run anyway** (the exe is self-signed)

<h2 div align="center"> Android

1. Install `KinitoPET-overlay.apk`
2. Open **Kinito 悬浮桌宠**
3. Tap **开启悬浮权限** → allow *Display over other apps* in system settings → go back
4. Tap **启动 Kinito** → return to the home screen

On Xiaomi / OPPO / vivo and similar ROMs you may also need to allow *background pop-up* and *autostart* in the app's special permissions, otherwise the overlay can be killed.

To stop it: open the app and tap **停止悬浮**.

---

## Build from source

### Desktop (Windows)

Requirements: Python 3.10+, Windows 10/11.

```bash
pip install PySide6 Pillow send2trash
python main.py
```

Package as a portable folder:

```bash
pyinstaller --noconfirm --windowed --name KinitoPet ^
  --icon assets/icon.ico ^
  --add-data "assets;assets" ^
  --add-data "config;config" ^
  main.py
```

### Android

Sources live in `android/`. The APK is built with the plain SDK toolchain — no Gradle required:

```bash
cd android

# 0. put the shared assets where the Android build expects them
mkdir -p assets/pet assets/voice
cp ../config/messages.json assets/messages.json
cp ../assets/idle.png ../assets/walk.png ../assets/Sleep*.png assets/pet/
cp ../voice/*.wav assets/voice/        # optional, only if you ship voice

# 1. resources
aapt2 compile --dir res -o compiled.zip
aapt2 link -o base.apk -I $ANDROID_JAR \
  --manifest AndroidManifest.xml -A assets --java gen compiled.zip

# 2. java -> dex
javac -source 8 -target 8 -classpath $ANDROID_JAR -d classes \
  src/com/kinito/pet/*.java gen/com/kinito/pet/R.java
jar cf classes.jar -C classes .
java -cp r8.jar com.android.tools.r8.D8 --lib $ANDROID_JAR \
  --min-api 21 --output dex classes.jar

# 3. merge classes.dex into base.apk, then sign
jarsigner -keystore kinito.keystore -signedjar KinitoPET.apk base.apk kinito
```

`assets/` must contain `messages.json`, `pet/*.png` and `voice/*.wav`.

### Web

```bash
python tools/build_mobile.py
```

Output: `dist/web/index.html` (roughly 7–8 MB, fully self-contained).
If your voice clips live outside the repo, point the builder at them:

```bash
KINITO_VOICE_DIR=/path/to/voice python tools/build_mobile.py
```

This inlines every image and voice clip into a single self-contained `index.html` (roughly 7–8 MB).

---

## Configuration

Both builds read the same files.

### `config/settings.json`

|                    Key                    |                           Meaning                           |
|-------------------------------------------|-------------------------------------------------------------|
|             `speed`, `tick_ms`            |        Movement speed and update interval (desktop)         |
|              `always_on_top`              |          Desktop window stays above other windows           |
|      `voice_enabled`, `voice_volume`      |               Mute switch and volume (0.0–1.0)              |
|          `behavior.walk_weight` …         | Relative weights for walk / idle / watch / sleep / mischief |
|        `message.cooldown_seconds`         |             Minimum gap between unprompted lines            |
|              `files.enabled`              |               Master switch for file behaviour              |
|               `files.roots`               |  Folders to index (default: Desktop, Downloads, Documents)  |
|       `files.scan_interval_seconds`       |             How often to refresh the file index             |
|        `files.delete_min_age_days`        |     How old a file must be before Kinito asks about it      |
|       `files.delete_quota_per_day`        |        Maximum delete questions per 24 h (default 3)        |
|       `files.confirm_before_delete`       |             Always ask before removing anything             |
| `files.allow_move` / `files.allow_delete` |       Per-feature switches (also exposed in the menu)       |
|         `files.move_probability`          |                  Chance of a tidy-up move                   |
|              `files.dry_run`              |              Log actions without touching files             |

### `config/messages.json`

Lines grouped by category:

```json
{
  "startup": ["..."],
  "shutdown": ["..."],
  "greet": ["..."],
  "idle": ["..."],
  "watch": ["..."],
  "mischief": ["..."],
  "delete": ["..."],
  "night": ["..."]
}
```

Categories are independent — add, remove or rewrite lines freely. Missing categories simply fall back to `greet`.

### Voice clips

Audio is matched by filename, so no mapping file is required:

```
voice/
├── greet_1.wav      → first line of "greet"
├── greet_2.wav      → second line
├── startup_1.wav    → first line of "startup"
├── first_hello.wav  → the one-time welcome line
└── ...
```

Supported formats: `.wav`, `.mp3`, `.ogg`. If a clip is missing, that line is shown silently. An optional `config/voice.json` can override any mapping explicitly:

```json
{ "You're back? Then let me accompany you": "startup_1.wav" }
```

The bubble stays up for `audio duration + 0.8 s`, so long lines are not cut off.

### Artwork contract

Place files in `assets/`:

|             File              |                    Notes                     |
|-------------------------------|----------------------------------------------|
|          `idle.png`           |     512×512, transparent, bottom-centred     |
|          `walk.png`           | Single walk pose (no frame animation needed) |
| `Sleep1.png`, `Sleep2.png`, … |   Sleep loop, played forward then backward   |
|          `drag.png`           |       Optional "being picked up" pose        |
|  `hand.png` / `hand_1.png`…   |    Optional hand for the tidying animation   |
|        `file_icon.png`        |      Optional file icon held by the hand     |
|          `icon.ico`           |          Multi-size app / tray icon          |

Art faces right by default; the renderer mirrors it automatically when walking left.

---

## Safety & privacy

This project is deliberately boring under the hood:

- **No screen capture, no OCR, no screenshots.** The desktop build only reads the foreground window *title* and system idle time to decide whether to say something.
- **No AI / LLM.** Every line is plain local text. Nothing is generated or sent anywhere.
- **No network calls.** Desktop, Android and web builds run fully offline.
- **File indexing is metadata only.** Path, size and timestamps go into a local SQLite database; file contents are never read.
- **Deletion is confirm-first.** Kinito asks *"Hmm... I helped you organize the files on your desktop, and now I think those icons look much better^ ^"* before touching anything. "No" leaves the file alone.
- **Deleted files are moved, not erased.** They go to `data/trash/<date>/` and can be restored from the menu for 30 days.
- **Hard limits.** At most 3 delete questions per 24 hours, files must be at least 180 days old, and executables, DLLs, shortcuts and system folders are excluded.
- **A kill switch for everything.** `files.allow_move` and `files.allow_delete` — also exposed as menu items — disable the features entirely. `files.dry_run` logs without acting.

The Android build contains none of the file features at all.

---

## Project structure

```
.
├── main.py                 # desktop entry point
├── pet/                    # desktop modules
│   ├── window.py           # pet window, painting, menus
│   ├── bubble.py           # floating speech bubble
│   ├── carry.py            # hand + file overlay
│   ├── behavior.py         # state machine
│   ├── files.py            # scan / move / trash / undo
│   ├── voice.py            # audio playback + duration
│   ├── memory.py           # SQLite index
│   ├── messages.py         # line pools
│   ├── autostart.py        # HKCU Run key
│   └── perception.py       # foreground title, idle time
├── config/
│   ├── settings.json
│   ├── messages.json
│   └── voice.json          # optional explicit mapping
├── assets/                 # artwork + icon
├── voice/                  # recorded voice clips (optional)
├── android/                # Android overlay sources
│   ├── AndroidManifest.xml
│   ├── src/com/kinito/pet/
│   └── res/
├── tools/
│   └── build_mobile.py     # builds the single-file web version
├── docs/                   # screenshots used by this README
├── LICENSE
└── README.md
```

Runtime data is written to `data/` (`pet.db`, `trash/`) and is intentionally separate from the source tree.

---

## Tech

- **Desktop:** Python 3.10+, PySide6 (Qt 6), Windows API through `ctypes`, SQLite, `send2trash`
- **Android:** Java, `WindowManager` overlay, foreground `Service`, `MediaPlayer`, no third-party libraries, no Gradle
- **Web:** plain HTML/CSS/JS, everything inlined as data URIs, works from `file://`

---

## Known limitations

- The desktop build is Windows-only (it uses Win32 APIs for foreground window and idle detection).
- The Android overlay needs *Display over other apps*; some ROMs kill background services aggressively.
- The iOS version does not exist and cannot float above apps — only the web page is available there.
- The UI strings are currently Chinese; dialogue and voice can be any language.
- The Android APK is signed with a self-signed key, so Android shows the usual "unknown developer" warning.

---

## Roadmap

- [ ] English UI strings
- [ ] Per-category voice volume / random pitch
- [ ] Optional TTS fallback for lines without recordings
- [ ] Android: notification action to stop the overlay
- [ ] Packaging scripts for one-command builds

---

## Credits & legal

- **KinitoPET** and the character Kinito are created by **troy_en**. This is an unofficial fan project made for personal enjoyment.
- All fan art, code and voice recordings in this repository are original works by the project author unless noted otherwise.
- No assets extracted from the original game are included or redistributed.
- The phone screenshot in `docs/android-home.png` shows Kinito running as a floating overlay. The **home-screen wallpaper in that screenshot is by a creator on Redbook**; it is used only to demonstrate the overlay, is not bundled with the app, and remains the property of its author.
- The desktop build can optionally reference dialogue lines for personal, non-commercial use. If you are a rights holder and would like something changed or removed, please open an issue.

**License**

- Code: MIT — see [LICENSE](LICENSE)
- Artwork and voice clips: © the project author, all rights reserved. Please ask before reusing them in another project.

---

## GitHub repository settings

Suggested description:

> A fan-made KinitoPET desktop pet for Windows and a floating overlay for Android. No AI, no screen capture, fully offline.

Suggested topics:

```
desktop-pet  kinitopet  pyside6  python  qt6  windows  android  overlay
floating-window  fan-project  offline  no-ai
```
