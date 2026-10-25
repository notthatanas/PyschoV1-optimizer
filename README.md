# Pyscho V1 Optimizer

A Windows configuration utility for performance and privacy. 60 reversible tweaks across eight
modules plus a live dashboard, and a common-apps installer. Every switch is applied through one
journalled engine, detected back from the machine, and reversible.

Built from scratch in C# on .NET 10 + WPF. No code, resources, icons, strings or control libraries
from any other optimizer are used; every registry target, service name and command line here is
re-implemented independently from Microsoft's own documentation.

A community optimizer (GPL-3) was used only as a checklist of features while building this. No
code, strings, icons or layout were taken from it: registry targets and service names are
functional facts, and every entry here carries its own description, risk badge and undo flow.

Built by **notThatanas**. Support and community: [Discord](https://discord.gg/jVMShPrxa) ·
[Instagram](https://www.instagram.com/yesthatanas?xtok=MWViaDdpemRtcXlnMA==).

---

## Install

Run the setup:

```
release/1.0.0/PyschoV1-Setup-1.0.0.exe
```

SHA-256 of the uploaded setup: `3652A123E193E2E7F3ADF1CF24F498C8115F55004EA776EC83C389FF7F062873`.

The installer is per-machine and needs one administrator prompt. It installs a self-contained x64
build to `C:\Program Files\Pyscho V1`, adds Start menu entries and an optional desktop icon,
registers **Pyscho V1 Optimizer** under Settings → Apps, and opens the app unelevated afterwards.
This build is unsigned, so SmartScreen shows "unknown publisher" the first time; choose
*More info → Run anyway*. Once a trusted certificate is configured, the build signs both the app and
the setup (see [Signing](#signing)).

Uninstalling removes the program files and shortcuts but deliberately keeps the settings and change
journal, so history survives a reinstall. The installed `README.txt` lists the exact paths.

Windows 10 1809 or newer / Windows 11, x64. No .NET runtime installation is required.

---

## Run the portable build

```
dist/PyschoV1.exe
```

* Self-contained: no .NET runtime install required.
* Portable mode: create an empty file named `PyschoV1.portable` next to the exe and settings plus
  the change journal stay beside the executable.
* `Reduce motion` in Settings turns off the window entrance, page transitions and card lift;
  Windows' own animation setting is respected too.
* A launch writes a short breadcrumb trail to `%AppData%\PyschoV1\startup.log`. If the app cannot
  open, `%AppData%\PyschoV1\errors.log` plus that file explain what failed instead of failing silently.
  See [If the app does not open](#if-the-app-does-not-open).

The app starts unelevated on purpose. Browsing, detection and preview mode work for a standard user;
the title bar shows the elevation state and relaunches elevated when you ask for it.

---

## What is in it

Eight modules plus a live dashboard, and the Apps installer.

| Module | What it does |
| --- | --- |
| **Dashboard** | Live CPU, memory, disk and network sampling, machine specs, volume usage, suggested next actions, one-click presets. |
| **Performance** | Power plan, Game Mode, Game DVR, MMCSS scheduling, hardware GPU scheduling, visual effects, transparency, Widgets, menu and startup delays, background apps, scheduler quantum, SysMain / Windows Search, power throttling, Delivery Optimization, network throttling, update restart policy, Storage Sense, tooltip delay, mouse acceleration, Edge background work, weekly registry backup,
verbose startup messages, Windows Update driver policy, print spooler, media sharing, Program
Compatibility Assistant, Xbox network and sensor services, and the Windows 10 news feed. |
| **Privacy & security** | Telemetry policy, DiagTrack and WAP push services, advertising ID, tailored experiences, Cortana, web results in Start, Copilot, Recall, feedback prompts, Windows Error Reporting, location, activity history, consumer features, suggestions, app launch tracking, Edge and Office telemetry, speech and inking collection, settings sync, AutoPlay, Remote Assistance, cloud clipboard, search highlights, Windows Spotlight, Remote
Registry, Quick Access recent files, the cross-device clipboard policy and the SMB1 protocol. |
| **Debloat** | Installed AppX packages with publisher and version, protected-system filtering, per-user removal, Store reinstall path. |
| **Startup** | Registry Run keys, Startup folders, the same `StartupApproved` flag Task Manager uses, plus JSON backup and restore. |
| **Cleaner** | Per-category size scan, then removal of temp files, update cache, prefetch, thumbnails, browser caches, crash dumps and the Recycle Bin. |
| **Network** | Physical adapter detection, six DNS presets (IPv4 + IPv6), DHCP reset, DNS flush, latency test. |
| **Apps** | 56 common applications in nine categories. winget when available (latest verified package), otherwise the publisher's own installer downloaded to `Downloads\PyschoV1`, otherwise the official page. Installed-state detection from the uninstall registry, per-card progress and cancel. |
| **Profiles** | Balanced, Competitive, Maximum privacy and Quiet machine presets, plus profile export/import. |

Cross-cutting:

* **Command palette** — `Ctrl+K` searches every tweak and page.
* **Plan preview** — expand any card to see the exact registry paths, values and services it touches
  before anything is written.
* **Change history** — every applied tweak with its previous value, single or bulk undo.
* **Risk badges** — Safe / Caution / Advanced, with a plain-English consequence line.
* **Motion** — window entrance, page and header transitions, hover lift on cards, click-driven toggle
  slides, toast and command-palette transitions, progress bars that fill instead of jumping, and a
  travelling shimmer on indeterminate bars. Page, card and window motion uses opacity and render
  transforms; the 46 px toggle knob slides its margin without causing a page layout.

Deliberately **not** included: disabling Windows Defender, a permanent Windows Update kill switch, or
destructive OneDrive removal. Those break machines and get utilities classified as unwanted software.
Windows Update is offered as deferral only.

---

## How a change is applied

```
UI toggle
  └─ TweakEngine.Apply(tweak)
       ├─ capture before-state for every op      → PatchRecord
       ├─ write (registry / service start / power plan)
       ├─ on any failure: unwind what was written, report why
       └─ append one JSON journal entry          → %ProgramData%\PyschoV1\journal
  └─ re-read the machine → the switch shows the real state, not a stored one
```

The switches show the real Windows state. A setting that was already enabled before Pyscho V1 is
labelled **On this PC** in amber and is read-only: the app never captured a previous value for it, so
it must not claim it can switch it back off. **Applied** means a journal entry exists and the toggle
can undo it. This is why a fresh install can show a mix of on and off switches before you change
anything — the machine, not the app, owns those values. Both tweak pages carry this legend above the
cards, and the counts are read live from Windows.

* `TweakEngine` is the only class allowed to mutate the system.
* Each op is declarative (`RegistryValueOp`, `RegistryDeleteOp`, `ServiceStartOp`, `PowerPlanOp`), so
  Plan Preview and Undo work generically instead of per-tweak code.
* Detection is derived from the ops, so a tweak can never claim a state it did not set.
* A restore point is taken once per session before the first write (skipped automatically when
  System Restore is off).
* The journal falls back to `%LocalAppData%\PyschoV1\journal` when ProgramData is not writable, so an
  unelevated session still records everything it can.

---

## Build

Requires the .NET 10 SDK. The installer additionally requires [Inno Setup 6](https://jrsoftware.org/isinfo.php):

```
winget install --id JRSoftware.InnoSetup -e --source winget
```

```
dotnet build src/PyschoV1.App/PyschoV1.App.csproj -c Release
dotnet run   --project src/PyschoV1.App/PyschoV1.App.csproj
```

Release build (single self-contained exe, ~65 MB):

```
dotnet publish src/PyschoV1.App/PyschoV1.App.csproj -c Release -r win-x64 `
  --self-contained true -p:PublishSingleFile=true `
  -p:IncludeNativeLibrariesForSelfExtract=true -p:EnableCompressionInSingleFile=true `
  -p:DebugType=none -o dist
```

Installer (publishes the app, verifies it starts, compiles the setup in a fresh folder, then
silently installs, launches and uninstalls a copy before publishing the setup):

```
powershell -ExecutionPolicy Bypass -File installer\build-installer.ps1
# package an exe that was already published:
powershell -ExecutionPolicy Bypass -File installer\build-installer.ps1 -SkipPublish
# skip the install/uninstall check (does not need an elevated shell):
powershell -ExecutionPolicy Bypass -File installer\build-installer.ps1 -SkipInstallerTest
```

The verification needs an elevated PowerShell because the setup installs per-machine. The script
refuses a version mismatch between the app and the installer, and it never leaves a setup in
`release\<version>\` unless that setup passed the install, launch and uninstall check. Each version
gets its own release folder, so a scanner holding an old setup open cannot block the next build.

Layout:

```
src/PyschoV1.Core      engine, catalog, journal, modules, hardware sampling (no UI references)
src/PyschoV1.App       WPF shell, design system, view models, pages
tests/PyschoV1.SelfTest  engine test suite + live read-only checks
```

Only documented Windows APIs are used: registry, `sc.exe`, `powercfg.exe`, `srclient.dll`,
`pdh.dll`, `kernel32` memory and CPU counters, `ipconfig.exe`, and PowerShell for AppX (the WinRT
projection is not worth a 100 MB SDK dependency for one page).

---

## Tests

```
# engine: 59 checks against an in-memory machine, including every catalog tweak round-trip
dotnet run --project tests/PyschoV1.SelfTest

# live read-only: hardware, specs, startup, adapters, cleaner, AppX, detection
dotnet run --project tests/PyschoV1.SelfTest -- --live

# offscreen render of all eleven pages + binding check + UI dry-run check
dist/PyschoV1.exe --render-preview artifacts/preview

# silent install, launch the installed exe, uninstall, verify nothing is left
powershell -ExecutionPolicy Bypass -File installer\test-installer.ps1
```

The render check writes `binding-errors.txt` (must read `none`) and `ui-check.txt`, which drives a
real toggle through the view model in preview mode and asserts nothing was written.The self-test covers: registry round-trip, delete-op restore, service start restore, power-plan
restore, partial-state detection, every one of the 60 catalog tweaks applying, detecting and
reverting against a fake machine, an absent service reporting unavailable instead of silently
succeeding, dry-run leaves no trace, journal persistence across reload, a failed journal write
rolling the tweak back, preset apply/revert, rollback after a mid-tweak failure, the elevation
error path, version gating, the cleaner's path guard, and the app-catalogue invariants (unique
ids, known categories, https links and word-boundary install-name matching).

---

## Measured performance

Published single-file build, i5-8250U / Windows 11 25H2, measured to first rendered frame with
`--benchmark`:

| Build | Size | Cold start | Private memory | Working set |
| --- | --- | --- | --- | --- |
| compressed, self-contained **(shipped)** | 65 MB | 0.69 – 0.83 s | 117 MB | 222 MB |
| compressed + ReadyToRun | 81 MB | 0.59 – 0.76 s | 163 MB | 222 MB |
| uncompressed + ReadyToRun | 205 MB | 0.60 – 0.75 s | 59 MB | 119 MB |
| uncompressed | 150 MB | 0.89 s | 49 MB | 108 MB |
| WPF baseline (empty window, same app resources) | — | 0.32 s | 38 MB | 89 MB |

The shipped build is the size/memory compromise: 65 MB download and 117 MB private memory, at
roughly 0.7–0.8 s to first frame on this i5-8250U. The catalog is lazy, so only the page you open is
built, and the dashboard's machine inventory and drive scan are deferred until after the first frame.
Working set includes shared runtime pages, which is why private memory is the meaningful column.
Hardware sampling runs on a background timer at 1 Hz and never blocks the UI thread.

---

## If the app does not open

1. Check `%AppData%\PyschoV1\startup.log`. A normal launch ends with `main window shown`; a failure
   records `FATAL: ...` there and a full stack trace in `errors.log` in the same folder.
2. If the exe came from a browser or chat download, right-click it → **Properties** → tick
   **Unblock** → **OK**, then run it again.
3. SmartScreen: **More info → Run anyway**. Antivirus can quarantine an unsigned registry tool; allow
   the file and retry. The app does not download anything at runtime.
4. The binary is x64 and needs Windows 10 1809 or newer. A 32-bit or ARM-only Windows cannot run it.
5. If the window is open but off-screen (for example after a monitor change), use `Alt+Tab`; the
   window restores to the centre of the primary display on the next launch.

The old pre-release build could crash while loading an animated toggle and leave no window; that
animation no longer targets template-owned brushes, and a failed launch now shows a dialog instead of
an invisible process.

## Known limits

* **No certificate is configured in this checkout.** The signing pipeline is ready, but the current
  artifacts remain unsigned, so Windows SmartScreen will warn. A publicly trusted certificate is
  required; see below.
* **Settings already on.** Many Windows privacy values arrive enabled by OEM images or previous
  tools. The app reports them as **On this PC** and leaves them read-only rather than claiming them.
* **The setup is per-machine.** It needs one administrator prompt and installs under Program Files.
  The portable exe remains available for machines where an install is not wanted.
* **Temperatures are not shown.** Reading CPU/GPU sensors needs a kernel driver or vendor SDK, which
  would mean shipping a third-party binary — the dashboard reports load, memory, disk and network
  instead.
* **AppX removal is per-user** and reversible by reinstalling from the Store.
* **Machine-wide tweaks are not exercised on a live Windows install by the build.** Everything was
  verified through the engine tests, dry-run, and read-only detection on the build machine; applying
  system-wide changes for real is a decision for you (or a VM).
* **App installs need the internet.** winget downloads from Microsoft's package source; the
  fallback downloads the publisher's own installer into `Downloads\PyschoV1`. Machine-wide
  installers may raise their own UAC prompt. Nothing is bundled or repackaged.

### Signing

A publicly trusted **OV/EV code-signing certificate** (or Microsoft Azure Artifact Signing) is the
only thing that makes SmartScreen identify **notThatanas** as the publisher on other people's PCs. A
self-signed certificate only works on machines where its root is explicitly trusted; it does not
remove the warning for downloaders.

Configure a certificate already in the Windows store, or a PFX, without putting secrets in the repo:

```
$env:CODESIGN_CERT_THUMBPRINT = 'ABCDEF0123456789...'
# or
$env:CODESIGN_PFX_PATH = 'C:\keys\notThatanas.pfx'
$env:CODESIGN_PFX_PASSWORD = '...'

powershell -ExecutionPolicy Bypass -File installer\build-installer.ps1
```

The build publishes to a staging folder, signs `release\<version>\PyschoV1.exe` before packaging,
signs `release\<version>\PyschoV1-Setup-<version>.exe` after compilation, requires an RFC 3161
timestamp, and verifies both signatures. [installer/sign.ps1](installer/sign.ps1) can also sign
existing files:

```
powershell -ExecutionPolicy Bypass -File installer\sign.ps1 -Path release\1.0.0\PyschoV1.exe, release\1.0.0\PyschoV1-Setup-1.0.0.exe
```

`-AllowUntrustedSignature` exists only to test the pipeline with a temporary self-signed certificate;
never publish those artifacts. The generated uninstaller is not separately Authenticode-signed.

SmartScreen reputation is separate from the signature. Microsoft no longer gives EV certificates an
automatic first-download bypass; a new certificate or a new file can still warn until it builds a
clean download history. Keep the same signing identity across releases.

---

## Licensing

Original work by **notThatanas**, no third-party source code. Windows registry paths, service names
and command-line switches are functional facts, not copyrightable expression. No telemetry, no
network calls, no bundled third-party binaries.
