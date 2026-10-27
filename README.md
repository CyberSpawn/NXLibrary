# NXLibrary

A cross-platform desktop viewer for your Nintendo Switch ROM collection. NXLibrary reads the manifest written by **NXsort** and presents your library as a searchable, sortable, paged table with icons, versions, firmware requirements and verification status. It can also launch NXsort for you to rename, sort and verify files, then rescan automatically when it finishes.

Built with [Avalonia UI](https://avaloniaui.net/) 11 on .NET 10, using the MVVM pattern (CommunityToolkit.Mvvm).

---

## Table of contents

- [Features](#features)
- [Requirements](#requirements)
- [Getting started](#getting-started)
- [Using NXLibrary](#using-nxlibrary)
  - [Library](#library)
  - [Detail screen](#detail-screen)
  - [Organize (NXsort)](#organize-nxsort)
  - [Log](#log)
  - [Settings](#settings)
  - [Placeholder pages](#placeholder-pages)
- [How the library is built from the manifest](#how-the-library-is-built-from-the-manifest)
- [Manifest format](#manifest-format)
- [Settings file](#settings-file)
- [Project structure](#project-structure)
- [Architecture notes](#architecture-notes)
- [Building from source](#building-from-source)
- [Platform notes](#platform-notes)
- [Troubleshooting](#troubleshooting)
- [Known limitations and roadmap](#known-limitations-and-roadmap)

---

## Features

- **Library table** with icon, title, title ID, languages, latest update, current version, minimum firmware, validation status and file name.
- **Instant search** across title, title ID and file name.
- **Sorting across the whole library** (not just the visible page) with a three-state header click: ascending, descending, back to default.
- **Version-aware sorting** so `9.0.0 < 10.0.0 < 10.1.0` instead of plain alphabetical order.
- **Pagination** with configurable rows per page (1 to 500) and a sliding window of numbered page buttons.
- **Detail screen** per game showing the base game, every update and every DLC, each with its own title ID, version, display version, min firmware, size and validation result.
- **Click to copy** titles and title IDs on the detail screen.
- **Click to reveal** any file in your OS file manager.
- **NXsort integration** with Fast, Normal and Slow modes plus icon generation, run in its own console window.
- **Automatic rescan** after NXsort closes.
- **In-app log** of scans, settings saves and NXsort runs.
- **Remembers** window size, position, maximized state and column widths.
- **Dark theme** (Fluent).

---

## Requirements

| Requirement | Notes |
|---|---|
| .NET 10 SDK | Needed to build. The runtime is needed to run unless you publish self-contained. |
| NXsort | `NXsort.exe` (Windows) or `NXsort` (Linux/macOS) placed **next to** `NXLibrary.exe`. Required for the Organize page and for creating the manifest and icons. |
| A ROM folder processed by NXsort | NXLibrary reads `nxsort.manifest.json`; it does not parse NSP/XCI files itself. |

---

## Getting started

1. **Build or publish** NXLibrary (see [Building from source](#building-from-source)).
2. Copy **NXsort** into the same folder as the NXLibrary executable.
3. Launch NXLibrary and open **Settings**.
4. Set the **Roms folder** (use *Browse*). Optionally set the icons folder and manifest file if they are not in the default locations.
5. Click **Save and rescan**. The library loads and the app switches to the Library page.

If you have not yet generated a manifest, open **Organize** and run NXsort first. Choose *Normal* for a first pass, then *Generate Game Icons* for thumbnails.

---

## Using NXLibrary

The top menu switches between pages: **Library**, **Missing Updates**, **Missing DLC**, **Organize**, **Log**, **Settings**, plus a **Rescan** action. A status bar at the bottom shows the current state (for example `120 of 250 games  |  sorted by Version ↓`).

### Library

- **Search box**: filters by title, title ID or file name (case-insensitive, substring match). Changing the search returns you to page 1.
- **Columns**

  | Column | Meaning |
  |---|---|
  | Icon | 56 px thumbnail (decoded at 64 px wide to keep memory low) |
  | Title | Taken from the file name with `[tags]` and extension removed |
  | Title ID | The **base** title ID of the game |
  | Languages | Abbreviated codes (EN, FR, ES, ...). Hover for the full names |
  | Update | Highest update version found, shown as `v<number>`, or `None` |
  | Version | Display version of the newest update, or of the base game if there are no updates |
  | Min Firmware | Minimum firmware of the base game |
  | Validation | `Passed` only if every file belonging to the game verified OK |
  | File Name | File name of the main (base) file |

- **Sorting**: click a header to sort ascending, again for descending, a third time to return to the default title order. Sorting applies to the *entire* filtered list before paging. Update, Version and Min Firmware use version-aware comparison.
- **Resizing**: columns can be dragged. Widths are saved when the window closes.
- **Paging**: First / Prev / numbered buttons / Next / Last. Up to nine numbered buttons are shown at a time, with the current page highlighted.
- **Opening a game**: click anywhere on a row.

### Detail screen

Opening a row covers the whole content area with a detail view:

- Large icon (loaded at full size), title, base title ID, publisher, product code, full language list, current version, latest update, min firmware, total size on disk, overall validation and containing folder.
- Three tables: **Base game**, **Updates** (newest first) and **DLC** (ordered by title ID). Empty sections show "None".
- **Click the title or any title ID** to copy it to the clipboard. The status bar confirms what was copied.
- **Click a file name** to open its folder in your file manager. On Windows and macOS the file is selected. If the file no longer exists, the nearest existing parent folder is opened instead.
- Press **Esc** to return to the list. Switching pages from the menu also closes the detail view.

### Organize (NXsort)

Launches NXsort in its own console window so you can answer any prompts. The window stays open at the end ("Press any key") so you can read the output. When you close it, NXLibrary rescans automatically. Buttons are disabled while a run is in progress.

| Mode | NXsort arguments | What it does |
|---|---|---|
| Fast | `--noverify --rename --recursive --folderize` | Rename and sort without verification |
| Normal | `--rename --recursive --folderize` | Rename, sort into game folders and verify each file |
| Slow | `--rescan --rename --recursive --folderize` | Ignore saved scan info, re-read and re-verify everything |
| Generate Game Icons | `--recursive --noverify --icon` | Extract one JPEG icon per game into an `icons` folder |

Every run is invoked with `--roms "<your roms folder>"`. Icons are named `Name [BASETITLEID].jpg`.

### Log

A timestamped list of events: scans (with game count, manifest path and icon folder), settings saves, NXsort launches and errors such as a missing manifest.

### Settings

| Setting | Description |
|---|---|
| Roms folder | The folder containing your ROMs. Required. |
| Icons folder | Optional. If blank, an `icons` folder inside the roms folder is used, otherwise one next to it. |
| Manifest file | Optional. If blank, `nxsort.manifest.json` inside the roms folder is used, otherwise one next to it. |
| Rows per page | 1, 5, 10, 25, 50, 100, 200 or 500 |
| Remember window size and position | Restores size, position and maximized state on next launch |

Click **Save and rescan** to persist and reload.

### Placeholder pages

**Missing Updates** and **Missing DLC** are not implemented yet. They will need an external source of latest versions and a title database respectively.

---

## How the library is built from the manifest

`RomScanner.Scan` turns the flat list of files and titles in the manifest into one row per game:

1. **Flatten**: every title in every file becomes a "part" carrying its file path, size and verification result.
2. **Classify**: `Application` is the base game, `Patch` is an update, and anything else (such as `AddOnContent`) is DLC.
3. **Group** parts by base title ID (`baseTitleId`, falling back to `titleId`).
4. **Pick the main file**: the highest-version base part, or the first part in the group if no base exists.
5. **Title**: the main file name with `[tags]` and extension stripped. Fallbacks, in order: the manifest name, the first folder below the roms folder, then the title ID.
6. **Update**: highest update version as `v<n>`, or `None`.
7. **Version**: display version of the newest update, else of the base game, else `v<version>`.
8. **Languages**: from the base game, else any title in the group that has them. Full names are mapped to short codes. Regional variants collapse (American/British English to `EN`, Canadian French to `FR`, Latin American Spanish to `ES`, Brazilian Portuguese to `PT`), while Traditional and Simplified Chinese stay separate (`ZH-TW`, `ZH-CN`). Unknown languages fall back to their first two letters, upper-cased.
9. **Validation**: `Passed` only if every file in the group has a verify result of `pass`; otherwise `Failed`.
10. **Total size**: sum of unique files (a file containing several titles is counted once).
11. **Icon**: first matched by `[BASETITLEID]` in the icon file name, then by exact (tag-stripped) name against the folder name and title. Substring matching is deliberately avoided so "Aggelos" and "Aggelos 2" do not get mixed up.
12. **Sort**: the default order is by title, case-insensitive.

---

## Manifest format

NXLibrary expects `nxsort.manifest.json` with a top-level `files` array. File paths are relative to the roms folder (forward slashes are converted automatically).

```json
{
  "files": [
    {
      "path": "BOMCAT/BOMCAT [0100XXXXXXXXXXXX][v0].nsp",
      "size": 123456789,
      "verify": { "result": "pass" },
      "titles": [
        {
          "type": "Application",
          "titleId": "0100XXXXXXXXXXXX",
          "baseTitleId": "0100XXXXXXXXXXXX",
          "version": 0,
          "name": "Game Name",
          "publisher": "Publisher",
          "productCode": "HAC-P-XXXXX",
          "displayVersion": "1.0.0",
          "minFirmware": "10.0.0",
          "languages": ["American English", "Japanese"]
        }
      ]
    }
  ]
}
```

`languages` may be an array of strings or a single string. Unknown fields are ignored.

---

## Settings file

Settings are stored in `settings.json` **next to the executable**. It holds the folder paths, rows per page, window placement (if enabled) and column widths (keyed by column header text). Because it lives in the app folder, that folder must be writable. Delete the file to reset everything.

---

## Project structure

```
NXLibrary/
├── NXLibrary.csproj          Project file (net10.0, Avalonia 11.2.3)
├── Program.cs                Entry point, Avalonia app builder
├── App.axaml / App.axaml.cs  Application, Fluent dark theme, DataGrid styles
├── MainWindow.axaml          All pages and the detail screen
├── MainWindow.axaml.cs       View code-behind (pickers, copy, reveal, sorting hook, window state)
├── MainViewModel.cs          State, paging, filtering, sorting, NXsort commands
├── GameItem.cs               GameItem and GamePart models
├── RomScanner.cs             Manifest parsing and game grouping
├── VersionComparer.cs        Dotted-version comparison for sorting
├── SettingsStore.cs          AppSettings and JSON persistence
├── NXsortRunner.cs           Launches NXsort in its own console window
└── ShellHelper.cs            Reveal a file in the OS file manager
```

---

## Architecture notes

- **MVVM** with `CommunityToolkit.Mvvm` source generators (`[ObservableProperty]`, `[RelayCommand]`). `MainViewModel` owns all state, and the view only handles things that need UI APIs (folder pickers, clipboard, window state).
- **Whole-list sorting**: the DataGrid's built-in sort only affects the visible page, so `OnSorting` cancels it and forwards the header to `MainViewModel.SetSort`, which sorts the full filtered list before paging.
- **Pipeline**: `_all` (every game) is filtered into `_filtered`, optionally sorted, then paged into the `Items` collection the grid binds to.
- **Scanning off the UI thread**: `RomScanner.Scan` runs inside `Task.Run`; the UI stays responsive while loading.
- **Compiled bindings** are enabled by default; the DataGrid and page-button `ItemsControl` opt out with `x:CompileBindings="False"`.
- **Event handling**: copy and reveal behavior is attached with tunnel-free bubble handlers on the window that look for `TextBlock`s marked with the style classes `copy` and `file`.

---

## Building from source

```bash
# restore and run
dotnet run

# release build
dotnet build -c Release

# self-contained publish (Windows example)
dotnet publish -c Release -r win-x64 --self-contained
```

Then copy `NXsort` (`NXsort.exe` on Windows) into the output folder next to `NXLibrary.exe`.

Packages: `Avalonia`, `Avalonia.Desktop`, `Avalonia.Themes.Fluent`, `Avalonia.Controls.DataGrid` (all 11.2.3) and `CommunityToolkit.Mvvm` 8.4.0.

---

## Platform notes

- **Windows** is the primary target. NXsort is run through `cmd.exe /c "..." & pause` with `UseShellExecute` so it gets its own console. The working directory is the app folder because `cmd.exe` cannot use a UNC path as its current directory, so ROM folders on network shares work fine.
- **Linux** is best effort. NXsort is launched through `x-terminal-emulator -e bash -c ...`, which requires a terminal emulator that supports `-e`. File reveal uses `xdg-open` on the containing folder.
- **macOS**: file reveal uses `open -R`. The NXsort launch path is the Linux one and may need adjusting.

---

## Troubleshooting

| Symptom | Likely cause and fix |
|---|---|
| "Choose your roms folder in Settings." | No roms folder is set or it no longer exists. Open Settings. |
| "Manifest not found." | Run NXsort from the Organize page, or set the manifest path manually in Settings. The default lookup is inside the roms folder, then its parent. |
| "Manifest error." | The JSON could not be parsed. See the Log page for the exception message. |
| "NXsort not found." | Place the NXsort executable next to NXLibrary. The expected path is written to the Log. |
| Games show no icon | Run *Generate Game Icons*, or set the icons folder. Icon file names must contain `[BASETITLEID]` or exactly match the title or folder name. |
| Validation shows `Failed` | At least one file for that game (base, update or DLC) did not pass. Check the detail screen for the specific file, or re-verify with Normal or Slow mode. |
| Min Firmware or Version empty | The manifest has no value for that title. |
| Window or column layout not restored | Enable *Remember window size and position* and save. Check that `settings.json` can be written. |

---

## Known limitations and roadmap

- **Missing Updates** and **Missing DLC** pages are placeholders.
- A game with no `Application` entry is still listed, using its first available part as the main file.
- A file that fails verification marks the whole game as `Failed`, even if the failing file is only a DLC.
- Sorting by Languages sorts the abbreviated string, not the full names.
- Settings are stored next to the executable, which is not suitable for read-only install locations.
