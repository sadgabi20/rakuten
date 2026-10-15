# Rakuten: Implementation Plan and Monorepo Structure

**Product:** Rakuten is a customizable and useful quake-style terminal. Its terminal can be set to a persistent position and size on your desktop so it works as a widget, and can be brought up with just a shortcut. It includes its own CLI, `rakuten`, with useful commands, including `rakuten open` for launching desktop apps, and a one-character alias (`k` by default) that works only inside Rakuten terminals. The default style of the terminal is glassmorphism.

**Platform target:** Windows 10 (1809+) and Windows 11 only, built so that macOS and Linux ports are additive rather than a rewrite.

---

## Table of contents

1. [Tech stack](#1-tech-stack)
2. [Architecture principles](#2-architecture-principles)
3. [Monorepo structure](#3-monorepo-structure)
4. [Windows implementation details](#4-windows-implementation-details)
5. [Configuration and theming model](#5-configuration-and-theming-model)
6. [CLI design](#6-cli-design)
7. [The `open` command](#7-the-open-command)
8. [Terminal-scoped alias](#8-terminal-scoped-alias)
9. [Phased roadmap](#9-phased-roadmap)
10. [Risks and mitigations](#10-risks-and-mitigations)
11. [Future port readiness](#11-future-port-readiness)
12. [First milestone](#12-first-milestone-vertical-slice)

---

## 1. Tech stack

| Concern | Choice | Rationale |
|---|---|---|
| App shell | Tauri 2 (Rust) on WebView2 | Small memory footprint, which matters for an always-resident app. Supports transparent, frameless windows. |
| PTY | `portable-pty` (ConPTY on Windows) | Pure Rust, no Node native modules. Cross-platform for later. |
| Terminal renderer | xterm.js with WebGL addon (canvas fallback) | Mature, themeable, good transparency support. |
| UI | React + TypeScript + Vite | Tabs, settings, command palette. |
| Glass effect | Windows Acrylic and Mica via DWM APIs | CSS `backdrop-filter` only blurs inside the webview. The real see-through blur must come from the OS. |
| CLI | Rust binary using `clap` | Single executable that controls the running app and launches apps. |
| IPC | JSON-RPC over a named pipe | No network port is opened, and access is restricted per user. |
| Config | TOML file with hot reload | Human-editable, with a JSON Schema for editor autocomplete. |
| Monorepo tooling | Cargo workspace + pnpm workspace, orchestrated by Turborepo | Each language uses native tooling, and Turbo handles caching and task order. |

---

## 2. Architecture principles

1. **Thin shell.** `src-tauri` only wires things together. All logic lives in `crates/`, so the app and CLI share the same types and the logic is testable without a window.
2. **One OS boundary.** Every OS-specific call lives behind a trait in the `rakuten-platform` crate, with `cfg`-gated implementations. No `#[cfg(windows)]` appears anywhere else, and CI enforces this with a grep check.
3. **Pure placement logic.** Window positioning, snapping and monitor validation are pure functions over plain data (`MonitorInfo`, `Rect`), so they are unit-tested without a display.
4. **Portable data formats.** Config, themes and IPC messages contain no Windows assumptions:
   - Shortcuts are stored as strings like `"Ctrl+Backquote"`, with a `Mod` alias for the primary modifier.
   - Shells are profiles (`{ name, command, args, env }`) with per-platform detection. Core never assumes `cmd.exe`.
   - Paths come from the `directories` crate, never hardcoded.
   - OS-specific settings live under a `[platform.windows]` table.
5. **Portability stays checked.** CI runs `cargo check` for macOS and Linux targets from day one, against stubs that return `Unsupported`.
6. **The CLI stands alone where it can.** Commands that don't need the running app (such as `open`, `config`, `doctor`) work even if Rakuten isn't running.
7. **Scope by inheritance, not installation.** Anything that should exist only inside Rakuten terminals (such as the one-character alias) is injected into the environment of the shells Rakuten spawns. Nothing is written to user profiles, the registry, or the global `PATH`.

### Platform trait surface

```rust
// crates/rakuten-platform/src/lib.rs
pub trait WindowEffects {
    fn apply_backdrop(&self, win: &Window, style: BackdropStyle) -> Result<()>;
    fn set_click_through(&self, win: &Window, on: bool) -> Result<()>;
    fn set_pin_level(&self, win: &Window, level: PinLevel) -> Result<()>; // Normal | AlwaysOnTop | Desktop
    fn set_taskbar_visible(&self, win: &Window, visible: bool) -> Result<()>;
}

pub trait Monitors {
    fn list(&self) -> Vec<MonitorInfo>;
    fn work_area(&self, id: MonitorId) -> Rect;
}

pub trait Autostart {
    fn enable(&self) -> Result<()>;
    fn disable(&self) -> Result<()>;
}

pub trait ShellDiscovery {
    fn detect(&self) -> Vec<ShellProfile>;
}

pub trait AppLauncher {
    fn discover(&self, extra_paths: &[PathBuf]) -> Result<Vec<AppEntry>>;
    fn launch(&self, app: &AppEntry, opts: &LaunchOptions) -> Result<LaunchHandle>;
    fn open_target(
        &self,
        target: &OpenTarget,
        with: Option<&AppEntry>,
        opts: &LaunchOptions,
    ) -> Result<LaunchHandle>;
}

pub struct LaunchOptions {
    pub args: Vec<String>,
    pub cwd: Option<PathBuf>,
    pub elevated: bool,
    pub new_instance: bool,
    pub wait: bool,
}

pub trait AliasShims {
    fn install_shim(&self, name: char, target: &Path, dir: &Path) -> Result<()>; // hardlink, else copy
    fn remove_shim(&self, name: char, dir: &Path) -> Result<()>;
    fn shim_dir(&self) -> PathBuf;
}

pub trait IpcTransport { /* named pipe now, unix socket later */ }

pub fn current() -> Platform; // cfg-selected implementation
```

---

## 3. Monorepo structure

```
rakuten/
├── apps/
│   ├── desktop/
│   │   ├── src/                     # React UI: tabs, settings, command palette
│   │   └── src-tauri/               # Thin Tauri shell (wiring only)
│   └── docs/                        # Documentation + landing site
│
├── crates/
│   ├── rakuten-core/                # Config schema, shared types, errors, events
│   ├── rakuten-pty/                 # Session manager over portable-pty, session env builder
│   ├── rakuten-ipc/                 # JSON-RPC protocol, server + client
│   ├── rakuten-window/              # Placement, snapping, persistence, monitor validation
│   ├── rakuten-apps/                # App index, fuzzy matching, aliases, target classification
│   ├── rakuten-platform/            # ALL OS-specific code
│   │   └── src/
│   │       ├── lib.rs               # Traits + current()
│   │       ├── windows/             # Implemented
│   │       ├── macos/               # Stub (Unsupported)
│   │       └── linux/               # Stub (Unsupported)
│   └── rakuten-cli/                 # `rakuten` binary (multicall: also runs as the alias)
│
├── packages/
│   ├── terminal/                    # xterm.js wrapper + PTY bridge hooks
│   ├── theme/                       # Design tokens, glass presets, theme loader
│   ├── ui/                          # Shared React components
│   ├── config-schema/               # JSON Schema generated from rakuten-core
│   ├── tsconfig/
│   └── eslint-config/
│
├── themes/                          # Built-in themes (JSON)
├── scripts/                         # Release, codegen, icon generation
├── .github/workflows/
│   ├── ci.yml                       # Windows: test + build. Mac/Linux: cargo check only
│   └── release.yml                  # Signed installer build + publish
├── Cargo.toml                       # Workspace root
├── pnpm-workspace.yaml
├── turbo.json
└── README.md
```

**Dependency direction (no cycles):**

```
rakuten-core  <-  rakuten-platform
              <-  rakuten-pty
              <-  rakuten-ipc
              <-  rakuten-window
              <-  rakuten-apps   (uses platform::AppLauncher)
                        ^
        rakuten-cli, apps/desktop/src-tauri
```

---

## 4. Windows implementation details

| Feature | Approach |
|---|---|
| Glass backdrop | Acrylic (Win10 1803+/Win11) and Mica/Tabbed Mica (Win11), via `window-vibrancy` or direct `DwmSetWindowAttribute` / `SetWindowCompositionAttribute`. Windows build number is detected at runtime. |
| Fallback chain | Mica -> Acrylic -> translucent solid color, selected automatically. |
| Rounded corners and border | DWM corner preference on Win11. CSS clip and border on Win10. |
| Global shortcut | `RegisterHotKey` through Tauri's global-shortcut plugin. Registration failures are surfaced to the user. |
| Hide from taskbar | `WS_EX_TOOLWINDOW` plus skip-taskbar, with a tray icon as the entry point. |
| Click-through | `WS_EX_LAYERED \| WS_EX_TRANSPARENT`. |
| Widget (desktop-level) mode | Bottom z-order, or parenting to the WorkerW layer. This is the least documented feature, so it is spiked early. |
| Frameless window chrome | Custom resize borders, drag region, and Windows 11 snap-layout compatibility. |
| IPC | Named pipe `\\.\pipe\rakuten-<user-sid>` with a per-user ACL. |
| Autostart | `HKCU\Software\Microsoft\Windows\CurrentVersion\Run`. |
| Shell detection | PowerShell 7, Windows PowerShell, cmd, WSL distros, Git Bash. |
| App discovery | Start Menu shortcuts, Store/UWP apps via `shell:AppsFolder`, App Paths registry keys, `PATH`. Details in [section 7](#7-the-open-command). |
| Alias shim | `k.exe` hardlink (copy fallback) of `rakuten.exe` in `%LOCALAPPDATA%\Rakuten\shims\`, on the `PATH` of Rakuten-spawned shells only. Details in [section 8](#8-terminal-scoped-alias). |
| Installer | NSIS/MSI via the Tauri bundler, plus a winget manifest. Scoop optional. |
| Updates | Tauri updater with signed releases. |

---

## 5. Configuration and theming model

**Config (`config.toml`)** holds these groups:

- `[window]`: edge or position, size, monitor, pin level, hide-on-blur, animation
- `[shortcut]`: toggle key and other bindings
- `[terminal]`: font, cursor, scrollback, default shell profile
- `[[profiles]]`: shell definitions
- `[theme]`: active theme plus overrides
- `[apps]`: app index and alias settings for `rakuten open`
- `[cli.alias]`: the one-character, terminal-scoped alias
- `[platform.windows]`: backdrop preference, reduce-effects-while-moving, and similar

**Glass theme tokens:** tint color, tint opacity, blur intensity (where the API allows), border highlight, noise grain, corner radius, shadow, and a terminal text contrast guard. Themes are plain JSON, so they can be shared and imported.

**Default theme:** a frosted dark tint with a subtle 1px light border, soft noise, rounded corners and a diffuse shadow. The terminal background is transparent over the tint layer, and text contrast is checked so it stays readable over bright wallpapers.

---

## 6. CLI design

```
rakuten toggle | show | hide
rakuten pin | unpin
rakuten position <top|bottom|left|right|x,y>
rakuten size <width>x<height>
rakuten tab new | list | close
rakuten run <command>              # run in a new or existing tab
rakuten open <target> [args...]    # launch desktop apps, files, folders, URLs
rakuten config get | set | edit | path
rakuten theme list | set | install
rakuten doctor                     # diagnose shortcut, backdrop, WebView2, ConPTY, shells, app index
rakuten completions <shell>
```

General behavior:

- If the app is not running, commands that need it launch it first and then send the command over IPC.
- Every command supports `--json` for scripting.
- Inside a Rakuten terminal, every command is also available through the one-character alias (see [section 8](#8-terminal-scoped-alias)), for example `k toggle` or `k open spotify`.
- Exit codes are stable and documented per command.

---

## 7. The `open` command

`rakuten open` launches apps by name from any shell, so `rakuten open code` or `rakuten open "visual studio code"` works without knowing install paths. It also opens files, folders and URLs with the default handler.

### 7.1 Usage

```
rakuten open <target> [args...]

  <target>          App name (fuzzy), alias, executable, file path, folder, or URL
  -a, --app         Force the target to be treated as an app name
  -w, --with <app>  Open a file/folder/URL with a specific app
  -n, --new         Start a new instance even if one is already running
  --admin           Launch elevated (UAC prompt)
  --cwd <dir>       Working directory for the launched app
  --wait            Block until the app exits
  --first           On ambiguous matches, take the top result without prompting
  --dry-run         Print what would be launched, without launching
  --list            List launchable apps (filterable: --list code)
  --reindex         Rebuild the app index
  --json            Machine-readable output
```

Examples:

```
rakuten open spotify
rakuten open code .                       # app "code" with arg "."
rakuten open ./report.pdf                 # default handler
rakuten open ./notes.md --with notepad++
rakuten open https://example.com
rakuten open "windows terminal" --admin
```

### 7.2 Resolution order for `<target>`

1. Existing path or URL -> default handler (or the `--with` app)
2. User alias from `config.toml`
3. Exact app name match
4. Executable on `PATH` or in App Paths
5. Fuzzy match over the app index

**Ambiguity:** with a TTY, show an inline picker. Without one (scripts), exit with a distinct code and print the candidates, unless `--first` is passed.

### 7.3 Exit codes

| Code | Meaning |
|---|---|
| `0` | Launched |
| `2` | No match |
| `3` | Ambiguous |
| `4` | Launch failed |
| `5` | Elevation declined |

### 7.4 Config

```toml
[apps]
fuzzy = true
index_refresh = "daily"        # or "on-demand"
extra_search_paths = ["D:\\Tools"]

[apps.aliases]
code = "Visual Studio Code"
ff = "Firefox"

# Aliases can carry default args and elevation
[apps.aliases.dev]
target = "Visual Studio Code"
args = ["C:\\work"]
```

### 7.5 Architecture

Discovery and launch are OS-specific. Indexing and matching are portable.

**`rakuten-apps` crate (OS-agnostic):**

- `AppEntry` model: `{ id, name, kind, launch_ref, icon_ref }`
- Index storage and cache (JSON file in the cache dir) and refresh policy
- Fuzzy matcher and ranking (exact > prefix > word > fuzzy, with a recent-use boost)
- Alias resolution and target classification (path, URL, app name)
- Argument quoting rules, kept free of OS assumptions

**`AppLauncher` trait in `rakuten-platform`:** shown in [section 2](#platform-trait-surface).

The CLI launches apps directly, so `rakuten open` works even when the Rakuten app isn't running. When the app is running, it can share the index over IPC (`apps.list`, `apps.launch`) for the command palette, but the CLI never depends on that.

### 7.6 Windows implementation

| Concern | Approach |
|---|---|
| Classic app discovery | Scan Start Menu folders (`%ProgramData%` and `%AppData%\Microsoft\Windows\Start Menu\Programs`) for `.lnk` files, and resolve shortcut targets via `IShellLink`. |
| Store/UWP apps | Enumerate `shell:AppsFolder` through the Shell COM API to get AppUserModelIDs. |
| Executables | `HKLM/HKCU\Software\Microsoft\Windows\CurrentVersion\App Paths` plus `PATH` lookup. |
| Launching | `ShellExecuteExW` for `.lnk`, `.exe`, files and URLs. UWP via `shell:AppsFolder\<AUMID>`. |
| Elevation | `ShellExecuteExW` with the `runas` verb. |
| `--new` | Skip single-instance activation where possible. Some UWP apps cannot do this, so report it rather than fail silently. |
| `--wait` | Wait on the process handle from `SEE_MASK_NOCLOSEPROCESS`. UWP launches don't return a reliable handle, so `--wait` is best-effort there. |
| Detaching | Launch outside the ConPTY job so closing a Rakuten tab never kills the launched app. |
| Args | Quote per Windows command-line rules in a single tested function. Never route through `cmd /c`, which avoids injection through crafted arguments. |
| Icons | Extract from the shortcut or exe lazily, used only by the palette UI. |

### 7.7 App integration

- **Command palette:** an "Open app..." mode backed by the same index and matcher, with icons and recents.
- **Tray menu:** optional pinned-apps section driven by aliases.
- **`rakuten doctor`:** reports index size, last refresh time, discovery errors per source (for example, an unreadable Start Menu folder), and slow-scan warnings.
- **Shell completions:** `rakuten open <TAB>` completes app names and aliases from the cached index.

### 7.8 Suggested slice

Discover Start Menu shortcuts only -> fuzzy match -> `rakuten open <name>`. Then add UWP, App Paths, `--admin`, and `--with`. Start Menu shortcuts cover most real-world apps, so this gives a useful command early.

---

## 8. Terminal-scoped alias

A single-letter alias (default `k`) that runs `rakuten` commands, but only inside shells that Rakuten spawned. Outside a Rakuten terminal it doesn't exist, so it can't clash with other tools or pollute the system.

### 8.1 Behavior

```
k toggle            # same as: rakuten toggle
k open spotify      # same as: rakuten open spotify
k theme set frost
k spotify           # optional fallback: unknown subcommand -> `open`
```

- Available in every shell profile Rakuten launches (PowerShell, cmd, Git Bash, WSL).
- Not installed globally, and not on the system or user `PATH`.
- Configurable name, and it can be turned off entirely.

```toml
[cli.alias]
enabled = true
name = "k"                 # must be exactly one character
fallback = "open"          # or "none": unknown subcommands run `open`
```

The default is `k` instead of `r` because `r` is a built-in PowerShell alias for `Invoke-History`. PowerShell resolves aliases before external commands, so a shim named `r` would never run there.

### 8.2 How scoping works

Scoping comes from process inheritance, so no profile or registry edits are needed.

1. **Session environment.** When `rakuten-pty` spawns a shell, it injects:
   - `RAKUTEN_SESSION=<session-id>`
   - `RAKUTEN_ALIAS=k`
   - a shim directory prepended to that process's `PATH` only, for example `%LOCALAPPDATA%\Rakuten\shims\`
2. **The shim.** The directory contains `k.exe`, a hardlink to `rakuten.exe` (with a copy fallback if hardlinks fail). The CLI is multicall: it checks its own executable name, and when invoked as `k` it behaves as `rakuten`.
3. **A guard inside the binary.** When invoked as `k` without a valid `RAKUTEN_SESSION`, it behaves like an unknown command (prints the shell's usual "not recognized" style message and exits with `9009`). This protects against a leaked `PATH` entry, and against the alias being inherited by a terminal opened from inside Rakuten.

This works identically across cmd, PowerShell, Git Bash and WSL, because it relies on `PATH` rather than shell-specific alias syntax.

### 8.3 Shell-specific init

Init scripts are only needed for features a shim can't provide, such as completions on `k`. They are injected per session and never written to the user's profiles.

| Shell | Injection |
|---|---|
| PowerShell | `-NoExit -Command` dot-sourcing a temp init script |
| cmd | `/K` with an init script |
| Bash (Git Bash/WSL) | `--rcfile` wrapper that sources the user's rc, then Rakuten's |
| WSL | Pass `RAKUTEN_SESSION` and `RAKUTEN_ALIAS` through `WSLENV`. The shim is a Linux-side wrapper that calls the Windows `rakuten.exe` via interop |

### 8.4 Where the code lives

- **`rakuten-core`:** the `[cli.alias]` config section, with validation (exactly one character, valid filename character, not a reserved device name).
- **`rakuten-pty`:** a single function builds the session environment (`RAKUTEN_SESSION`, `RAKUTEN_ALIAS`, shim-prefixed `PATH`), so every shell profile gets the same treatment.
- **`rakuten-platform`:** the `AliasShims` trait (see [section 2](#platform-trait-surface)). Windows creates the shim via hardlink, and future ports will use symlinks.
- **`rakuten-cli`:** multicall dispatch (`argv[0]` stem equals the configured alias routes to the main parser), the session guard, and the optional `fallback = "open"` routing.

Shims are created at first run and rebuilt when the alias name or the binary path changes (for example after an update). They are removed on uninstall.

### 8.5 `doctor` and completions

`rakuten doctor` additions:

- Alias enabled, name valid, shim present and pointing to the current binary
- **Shadowing check:** warn if the chosen name collides with a PowerShell alias, a cmd builtin, or another executable on the normal `PATH`
- Whether the current shell is a Rakuten session

Completions for the alias mirror `rakuten`, via `rakuten completions <shell> --as <name>`.

### 8.6 Smallest slice

Shim directory + env injection + multicall dispatch for `k toggle`. That proves the scoping model on PowerShell and cmd before the extras are added.

---

## 9. Phased roadmap

### Phase 0: Foundation

- Set up Cargo and pnpm workspaces, Turborepo, lint and format rules, and CI.
- Define the `rakuten-platform` traits (including `AppLauncher`), the config schema (including `[apps]`), and the IPC message types before any feature work.
- Create the `rakuten-apps` crate skeleton.
- Add the `[cli.alias]` schema, the `AliasShims` trait, and the session-environment builder in `rakuten-pty`.
- Add the `cfg` grep check and the mac/linux `cargo check` jobs.

### Phase 1: Working terminal

- Frameless, transparent Tauri window.
- Spawn a ConPTY session and stream it to xterm.js.
- Resize, copy/paste, scrollback, clickable links, shell profiles, and tabs.
- Inject the session environment into spawned shells and create the alias shim.

### Phase 2: Quake behavior

- Global shortcut toggle with focus handling and an optional hide-on-blur.
- Persistent position and size, stored in physical pixels plus monitor identity, and re-validated against current displays on every show.
- Snap to edge, pin levels, tray icon, and taskbar hiding.
- Widget mode: always-on-top first, desktop-level once the spike succeeds.

### Phase 3: Glassmorphism

- Acrylic/Mica backdrop through the platform trait, with the automatic fallback chain.
- Theme token system, settings panel with live preview, and theme import/export.
- "Reduce effects while moving" toggle to avoid acrylic drag lag.

### Phase 4: CLI and IPC

- Named-pipe JSON-RPC server inside the app and the CLI client, with auto-launch.
- All commands listed in [section 6](#6-cli-design), including `doctor`.
- `rakuten open`: discovery, matching, launching, and all flags from [section 7](#7-the-open-command).
- Alias: multicall dispatch, the session guard, the `open` fallback, and the `doctor` checks.

### Phase 5: Polish and distribution

- Command palette (including "Open app..." mode), scrollback search, and shell integration (cwd tracking and prompt marks via OSC 7 and OSC 133).
- Tray pinned apps, `open` completions, and icon extraction.
- Per-shell completions for the alias, and WSL alias support.
- Autostart, single-instance enforcement, installer, auto-update, and winget/Scoop packaging.
- Code signing and optional opt-in crash reporting.

---

## 10. Risks and mitigations

| Risk | Mitigation |
|---|---|
| Frameless transparent window resize and snap behavior on WebView2 | Prototype in the first week, with custom resize handling. |
| Acrylic and Mica differ by Windows build and can lag when dragging | Build-number gating, the fallback chain, and the reduce-effects toggle. |
| Desktop-level widget mode is poorly documented | Spike early, and ship always-on-top first if it proves unreliable. |
| Hotkey conflicts (reserved Win-key combos, `` Ctrl+` `` clashes with editors) | Report registration errors clearly in the UI and in `rakuten doctor`, and let users rebind easily. |
| Mixed-DPI, multi-monitor setups | Store physical pixels plus monitor ID and re-validate on show. Test with scaled secondary displays. |
| WebGL transparency performance on some GPUs and drivers | Benchmark early, with an automatic canvas-renderer fallback. |
| Antivirus false positives (global hotkeys, named pipes, unsigned binary) | Sign binaries and installers before any public release. |
| `open`: fuzzy match launches the wrong app | Prefer exact and alias matches, show a picker on ambiguity, and support `--list` and `--dry-run`. |
| `open`: index goes stale after installs/uninstalls | Daily refresh, `--reindex`, and a fallback re-scan when a launch target no longer exists. |
| `open`: UWP inconsistencies (`--new`, `--wait`, args) | Document per-kind limitations and report them as warnings in `--json` output. |
| `open`: elevated launches from a non-elevated terminal | `--admin` is explicit only, never inferred, and a declined UAC returns its own exit code. |
| `open`: argument injection | No shell involved in launching, one tested quoting function, and fuzz tests on it. |
| `open`: slow first scan on large machines | Run it in the background, serve partial results, and cache aggressively. |
| Alias name shadows a PowerShell alias or another tool | Default to `k`, run the shadowing check on startup and in `doctor`, and warn when the config is changed to a colliding name. |
| A terminal launched from within Rakuten (for example `wt`) inherits the env and `PATH` | The guard validates `RAKUTEN_SESSION` against the running app over IPC where possible, and only the app's own session IDs are accepted. If the app isn't running, already-open sessions keep working. |
| Hardlinks fail (non-NTFS volumes, restricted policies) | Fall back to a copy and rebuild it when the binary version changes. |
| Antivirus flags a renamed executable | The shim is a hardlink of the signed binary, so signing the release covers it. |
| Stale shim after an update | Version-check the shim on startup and refresh it. |
| WSL interop disabled | `doctor` reports it, and the alias is skipped for that profile instead of failing. |

---

## 11. Future port readiness

Adding another OS is additive:

1. Implement `rakuten-platform/src/<os>/`.
2. Add the OS-specific `[platform.<os>]` config section.
3. Add a CI build job.

Core, PTY, window math, IPC protocol, the app index and matcher, CLI logic and all TypeScript packages should not need changes. Features an OS or compositor cannot provide return `Unsupported` and are reported by `rakuten doctor`.

Platform-specific notes for `open`:

- **macOS:** discover `.app` bundles in `/Applications` and `~/Applications`, launch via `open -a` or LaunchServices.
- **Linux:** parse `.desktop` files per the XDG spec, launch via `gio launch` or `Exec` expansion.

The index model, matcher, aliases, CLI flags and exit codes stay identical across platforms.

Platform-specific notes for the terminal-scoped alias: macOS and Linux implement `AliasShims` with symlinks, and the session-environment approach works unchanged.

---

## 12. First milestone (vertical slice)

Tauri window -> ConPTY -> xterm.js -> global hotkey toggle -> Acrylic backdrop -> `rakuten toggle` over the named pipe, with every OS call already routed through `rakuten-platform`.

It touches every layer and validates both the product feel and the portability boundary before more features are built. `rakuten open` follows as the first CLI feature that works without the app running, and the `k` alias slice (shim + env injection + multicall) can be added alongside it.