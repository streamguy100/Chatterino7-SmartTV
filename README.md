# Chatterino7 Smart TV

Smart TV edition of **Chatterino7** (SevenTV/chatterino7, commit
`910b01dd428c4ac042028708704278b93227491b`, "fix: plugin test after merge").
Dark TV UI, big readable text, tabs like TV channels — and **every feature
still works**: Twitch + Kick chat, 7TV / BTTV / FFZ emotes, highlights,
notifications, plugins, tabs and splits. Style + the two features below
only; no protocol code was touched.

## What it does

### 0. TV boot loading screen (every launch)
Every time the app is opened, a black TV screen shows first, like a real
TV being turned on: **SMART TV / Chatterino7 TV** logo, a red loading bar
filling up, "Loading channels..." — then it fades into the chats. About
2.5 s on screen + a short fade. It shows on **every** launch, not just
first run.

It never blocks anything: the chats load underneath while the splash
shows (the splash is only a visual cover over the window), and a fixed
timer hides it no matter what — so it can never stick on loading even if
chats are slow to connect. Main window only; popups don't get it. It
follows the same Smart TV switch as the remote + static: untick
"Smart TV mode" in Settings → General → Messages and it won't show.

### 1. On-screen TV remote
A remote bar sits under the chat in the main window: power (minimises the
window), a `CH 1` / `CH 2` … channel display, `CH-` and `CH+` buttons.
`CH+` / `CH-` switch tabs exactly like the notebook does.

**Every time you change tabs — click, Ctrl+Tab, Ctrl+1–9, mouse wheel,
remote buttons, any tab switch — the remote's CH+ button lights up** in
the current theme accent colour for ~1.5 s, like changing channel on a TV.
The glow colour follows the holiday theme (gold in January, pink in
February, orange in October, …).

### 2. Two seconds of TV static on every tab switch
On every real tab switch, animated TV static covers the chat area for
exactly ~2 s, then the new tab's chat is revealed. The noise is generated
live (a small random image repainted at 20 fps and scaled up), never a
still image. The switch itself happens **first** — the static can never
block or delay it — and the overlay ignores mouse clicks, so you can keep
switching tabs while static is showing (each switch restarts the 2 s).
No static on startup, and none when a tab is closed.

### 3. Auto holiday theme (no setup needed)
The Smart TV theme changes automatically with the PC date, whole month =
that holiday's look. One primary holiday per month (USA):

| Month | Holiday | Look |
|---|---|---|
| Jan | New Year's | gold |
| Feb | Valentine's | pink / red |
| Mar | St Patrick's | green |
| Apr | Easter | pastel |
| May | Memorial Day | red / white / blue |
| Jun | Juneteenth | red / gold / green |
| Jul | Independence Day | red / white / blue fireworks |
| Aug | — none — | default Smart TV theme |
| Sep | Labor Day | blue / work |
| Oct | Halloween | orange / black |
| Nov | Thanksgiving | brown / orange |
| Dec | Christmas | red / green |

Each is a **full theme swap** — accent, window, messages, tabs (including
the selected-tab line), split header/input, highlights, scrollbars — and
the remote CH+ glow follows the accent. The default Smart TV theme
(TV red on black) is the fallback for no-holiday months.

**Easter is calculated properly** with the Gregorian computus (Anonymous
algorithm): 2024 → Mar 31, 2025 → Apr 20, 2026 → Apr 5, 2027 → Mar 28,
2028 → Apr 16. So in Easter-in-March years (2024, 2027, …) March becomes
Easter and April falls back to the default Smart TV theme.

Checked on startup and again exactly at the next month boundary (local PC
time, single-shot timer — no polling). It only auto-swaps while a Smart TV
theme is active; if you pick Dark, Light, or any other theme, your choice
is never overridden.

**Preview / test any month (no settings UI):** close the app, then launch
it from a command prompt with the month set, e.g. for October:
```cmd
set CHATTERINO_SMARTTV_HOLIDAY_MONTH=10
Chatterino7SmartTV.exe
```
`1`–`12`; the current year is used for the Easter computus. Unset the
variable to go back to the real PC date.

## Turn-on steps (sir only downloads the .exe)

1. Download `Chatterino7SmartTV.zip` from the GitHub Actions run, unzip it.
2. Run `Chatterino7SmartTV.exe`. No install, no building.
3. It boots straight into the Smart TV theme, with this month's holiday
   look already applied (November → Thanksgiving brown/orange). You'll see
   the black TV boot screen first (~2.5 s, then it fades into the chats).
4. Big text (recommended, your call): Settings → Appearance → set the chat
   font size to 18–20 and UI Scale to 1.25–1.5. The theme itself can't set
   font size; that's a setting.
5. To turn the remote + static off (theme and holiday auto-swap stay on):
   Settings → General → Messages → untick **"Smart TV mode (remote +
   channel static)"**.

## Files in this folder

- `smarttv.patch` — the complete patch against Chatterino7
  `910b01d` (22 files: 8 edited, 2 new widgets, 12 theme JSONs).
  Apply: `git apply smarttv.patch` in a fresh Chatterino7 checkout.
- `build-smarttv.yml` — GitHub Actions workflow (Qt 6.9.3 / MSVC2022 /
  Conan 2.11.0 / Ninja, same pattern as the Mergerino iPhone build).
  Goes in `.github/workflows/` of the `Chatterino7-SmartTV` repo.
- `generate_holiday_themes.py` — generator for the 12 theme JSONs
  (palettes are the source of truth; don't hand-edit the JSONs, edit the
  palettes here and re-run).
- `README.md` — this file.

## Files changed (in the patch)

| File | Change |
|---|---|
| `src/widgets/SmartTv.hpp` / `.cpp` | **New.** `SmartTvRemote` (CH−/CH+, power, channel label, accent glow), `SmartTvStaticOverlay` (live animated noise, 2 s, mouse-transparent), and `SmartTvBootScreen` (black TV boot splash, logo + loading bar, ~2.5 s + fade, fixed fallback timeout, chats load underneath). |
| `src/widgets/Notebook.cpp` | `SplitNotebook::select()` — the single funnel for every tab switch — captures the previous page and, only on a real switch between two open tabs, calls `Window::onSmartTvTabChanged()` after the switch is done. Startup and tab closes never fire it. |
| `src/widgets/Window.{hpp,cpp}` | Creates the remote bar + static overlay + boot screen on main windows only; `onSmartTvTabChanged()` flashes CH+ and shows the static; remote visibility follows the `smartTvMode` setting; the boot screen is shown once per launch (same switch). |
| `src/singletons/Settings.hpp` | New `smartTvMode` bool (`/appearance/smartTvMode`, default `true`). |
| `src/singletons/Theme.{hpp,cpp}` | Registers all 12 Smart TV themes (after Dark, so Dark stays the fallback at index 2); default theme on a fresh install is `SmartTV`; `checkSmartTvHolidayTheme()` + `scheduleSmartTvHolidayCheck()`; Gregorian computus + month map in the anonymous namespace. |
| `resources/themes/SmartTV*.json` | **New, 12 files.** Full themes generated from Dark.json by `generate_holiday_themes.py`. |
| `src/widgets/settingspages/GeneralPage.cpp` | "Smart TV mode (remote + channel static)" checkbox under General → Messages. |
| `src/CMakeLists.txt` | Adds `widgets/SmartTv.cpp` / `.hpp` to the explicit source list. |

## How the wiring works

```
app launch (main window, smartTvMode on)
  -> Window::addLayout() creates SmartTvBootScreen, showBootScreen()
  -> black splash + loading bar for ~2.5 s (chats load underneath)
  -> fixed timer -> fade out over 0.4 s -> chats visible (never sticks)

any tab switch (click / Ctrl+Tab / Ctrl+1-9 / wheel / CH+ button)
  -> SplitNotebook::select()            (Notebook.cpp, the one funnel)
  -> Notebook::select() completes the switch first
  -> Window::onSmartTvTabChanged()      (main window + smartTvMode on only)
       -> SmartTvRemote::flashChannelUp()   (CH+ glows in theme accent, 1.5 s)
       -> SmartTvStaticOverlay::showStatic() (live noise over chat, 2 s, restartable)
```

Holiday theme:
```
Theme ctor -> update() -> checkSmartTvHolidayTheme()   (startup, PC date)
           -> scheduleSmartTvHolidayCheck()            (single-shot to 1st of next month)
check: current theme starts with "SmartTV"? -> resolve month (+Easter computus)
  -> themeName.setValue(wanted) -> theme listener -> update() (full theme swap)
```

## Build it (GitHub Actions — the only supported route)

Repo `streamguy100/Chatterino7-SmartTV` (create it in the GitHub web UI,
public), put `smarttv.patch` at the repo root and `build-smarttv.yml` in
`.github/workflows/`, push to `main` (or Run workflow). The workflow
clones SevenTV/chatterino7 @ `910b01d`, applies the patch, builds Release
with Qt 6.9.3 / Conan 2.11.0, runs `windeployqt`, and uploads two
artifacts: `Chatterino7SmartTV.zip` (full bundle) and
`Chatterino7SmartTV.exe`.

Local Windows build (same steps, from upstream `docs/building-windows.md`,
x64 Native Tools prompt, from the patched source root):
```cmd
set PATH=C:\Qt\6.9.3\msvc2022_64\bin;%PATH%
python -m pip install "conan==2.11.0"
conan profile detect --force
mkdir build-conan & cd build-conan
conan install .. -s build_type=Release -c tools.cmake.cmaketoolchain:generator=Ninja --build=missing --output-folder=. -o with_openssl3=True
cmake -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_TOOLCHAIN_FILE=conan_toolchain.cmake -DUSE_PRECOMPILED_HEADERS=ON -DCHATTERINO_LTO=OFF -DCHATTERINO_SPELLCHECK=ON ..
cmake --build . --parallel
windeployqt bin/chatterino.exe --release --no-compiler-runtime --no-translations --no-opengl-sw --dir Chatterino7SmartTV --force-openssl
```

## Verification done here (Linux VM, no Qt toolchain)

- Patch applies cleanly to a fresh Chatterino7 clone at `910b01d`
  (`git apply --check` + apply, 23 files).
- Brace/paren/bracket balance on every edited/new C++ file (the one
  GeneralPage.cpp flag is a checker artifact — the pristine upstream file
  trips the same check with the same leftover count; the diff is 8 added
  lines only).
- All 12 theme JSONs parse and have exactly the same key structure as
  upstream Dark.json; theme keys registered in Theme.cpp match the
  generated files 1:1.
- Gregorian computus verified in a Python mirror of the C++ code:
  2024 Mar 31, 2025 Apr 20, 2026 Apr 5, 2027 Mar 28, 2028 Apr 16 — and
  the full month map checked for 2024–2028 (Easter-in-March years give
  April = default; August = default every year).
- Every identifier used was checked against the upstream source
  (`Notebook::selectNextTab/selectPreviousTab/getSelectedIndex`,
  `SplitNotebook::select` override, `Window::parentWidget`,
  `getTheme()->accent`, `BoolSetting::connect` with a bool lambda +
  SignalHolder as in `windowTopMost`, `SettingWidget::checkbox`).

**Not done here, and not claimed:** no Qt toolchain on this VM, so the
code was not compiled here — the GitHub Actions run is the first real
compile check. No Windows `.exe` was produced here. If the build reports
an error it will name the exact file/line, almost certainly in one of
the files listed above.
