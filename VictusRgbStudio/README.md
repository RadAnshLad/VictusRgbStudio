# VictusRgbStudio

A lightweight keyboard lighting controller for HP Victus laptops that talks directly to HP's `hpqBIntM` WMI interface.

This is my first public project. I built it after getting tired of OMEN Gaming Hub and OMEN Light Studio locking out lighting effects on Victus models and sticking us with basic static options while eating hundreds of MBs of RAM in the background.

## How It Works

VictusRgbStudio splits the background lighting engine from the UI so it stays out of the way:

- **Tray-first footprint:** Runs headless in the system tray at roughly 12–18 MB RAM when the window is closed (`EmptyWorkingSet` + working-set trimming). The tray icon shows a dark smoked-glass orb when off/idle and a glowing green orb while active.
- **On-demand WPF UI:** Opening the studio spawns a frameless Windows 11 blur window (`SetWindowCompositionAttribute` + `148x148` Snell's-law orb refraction). Closing it tears down the WPF tree completely instead of hiding it in memory.
- **Direct WMI control:** Reads and writes the 128-byte firmware color table (`0x20009 / 0x02` and `0x03`) via `root\wmi` `hpqBIntM` at ~11.1 Hz (`90 ms` ticks), skipping redundant writes when RGB values haven't changed.

## Features

- **18 Lighting Modes:**
  - *Dynamic:* Random Color Pulse, Smooth Random Morph, Heartbeat Pulse, Rainbow Spectrum, Custom Palette Cycle (2–3 colors), Emerald Aurora, Aurora Drift, Solar Flare, Bioluminescent Tide, Ember Drift, Twilight Prism, and Cyberpunk Strobe / Plasma Arc.
  - *Reactive:* Music Visualizer (live Windows `IAudioMeterInformation` peak tracking), Screen Ambilight (GDI `StretchBlt` center-screen sampling), Music Video Ambience (screen color combined with audio beat brightness), Keypress Flash (low-level keyboard hook), and Thermal Gauge (ACPI / chassis sensor).
- **Drag-and-Drop Playlist:** Reorder modes by dragging rows in the Preset Queue, pin any mode for startup, or lock Pulse / Heartbeat / Music Visualizer to a single color.
- **Power & Idle Tweaks:**
  - **Battery Saver:** Caps brightness at 40% when unplugged and battery drops to 40% or below.
  - **Idle Dim:** Dims the keyboard to 20% and throttles WMI polling (`180 ms`) after 1–15 minutes of inactivity.
  - **Idle Sleep:** Turns the backlight off (`0x20009 / 0x05`) after 1–30 minutes of inactivity.
  - **UI Tray Timeout:** Automatically closes the WPF window back to the tray after 1–15 minutes of UI inactivity.
- **Accessibility:** Respects Windows `HighContrast` and `ClientAreaAnimation` system settings.

## Compatibility

| Build | Target Hardware | Notes |
| :--- | :--- | :--- |
| `VictusRgbStudio_Universal.zip` | HP Victus 15 / 16 with 1-zone WMI RGB | Auto-detects laptop model name; also mirrors all 4 slots if run on a 4-zone WMI board. |
| `VictusRgbStudio_15fb.zip` | HP Victus 15-fb3185AX | Original build tuned for the 15-fb3185AX. |

- **Not supported:** White-only `Fn+F4` backlights (fixed hardware steps, no WMI color table) and per-key USB HID keyboards.
- **Windows 11 Dynamic Lighting:** If enabled in Windows Settings (`Personalization > Dynamic Lighting`), Windows will overwrite WMI color writes. Turn it off so VictusRgbStudio can control the keyboard cleanly.

## Building & Running

No external SDK or Visual Studio installation is required. Every `.zip` includes the single `.cs` source file, `build_exe.bat`, and `.ico` icon:

1. Extract the `.zip` into its own folder.
2. Run `build_exe.bat` (compiles with Windows' built-in .NET Framework 4.x `csc.exe /target:winexe /optimize+`).
3. Launch the resulting `.exe` (requires Administrator privileges for `root\wmi` access).

If Windows Task Manager or Explorer caches an older icon after rebuilding, press `I` in `build_exe.bat` to reset the Windows 11 icon cache. Any unhandled runtime errors are logged to `victus_crash.log` next to the executable.

## Afterword

The Liquid Glass UI look was mostly vibecoded, while the core WMI controller and preset system came mostly from my own research into `hpqBIntM` and Windows APIs, with a bit of AI help implementing the preset patterns.
