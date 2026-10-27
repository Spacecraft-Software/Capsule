<!--
SPDX-FileCopyrightText: 2026 Mohamed Hammad <Mohamed.Hammad@SpacecraftSoftware.org>
SPDX-License-Identifier: CC-BY-SA-4.0
-->
<!-- GFM Document
     title:      Capsule — PRD
     author:     Mohamed Hammad & Spacecraft Software
     date:       2026-10-27
     version:    0.1.0
     license:    CC-BY-SA-4.0
     project:    Capsule
     website:    https://Capsule.SpacecraftSoftware.org/
-->

# Capsule — PRD

MVP: M0–M5 — Capsule v0.1 in the Bravais greeter

**Maintainer:** Mohamed Hammad — <Mohamed.Hammad@SpacecraftSoftware.org>\
**Repository:** `Spacecraft-Software/Capsule` (GitHub) · `/spacecraft-software/capsule` (local)\
**Assurance:** Category B (Standard §19) · **Posture:** Personal / Hobby (§5.1)\
**Requirement prefix:** `CPS`\
**Status:** Planning — requirements `draft`, awaiting G1

Capsule runs one program fullscreen on a seat and exits with that program's exit status. It is
a Wayland kiosk compositor in Rust, built on [Smithay](https://github.com/Smithay/smithay), and
the Spacecraft Software successor to [Cage](https://github.com/cage-kiosk/cage). Its first user
is the Steelbore OS Bravais greeter, whose shell sessions (Brush, Ion, mjsh, Nushell) currently
run under cosmic-comp's kiosk mode.

This file is the §17.5 tracking document for the product. It sequences the work; it does not
define it. **The requirements themselves — with rationale, source, priority, verification
method, and status — live in [`doc/capsule.texi`](doc/capsule.texi), nodes `Needs` and
`Requirements`** (Standard §20.1). Every milestone item below is the §20.3 identifier of a
requirement or need; the work packages are in [`PLAN.md`](PLAN.md) and the tasks in
[`TODO.md`](TODO.md).


## Contents

- [Decisions](#decisions)
- [Architecture](#architecture)
- [Interfaces](#interfaces)
- [Out of scope](#out-of-scope)
- [Open items for G2](#open-items-for-g2)
- [Milestones M0–M7](#m0--foundation-and-gates)


## Decisions

Settled with the maintainer on 2026-10-27 unless marked otherwise.

| Topic | Decision |
|-------|----------|
| Name | **Capsule** — a one-person spacecraft, and a nod to Cage. Binary and crates `capsule`, `capsule-policy`, `capsule-x11` |
| Why not cosmic-comp | Its kiosk mode is an argument-handling quirk, not a contract; it loads COSMIC's configuration and shortcuts into a bare shell session; the patched build costs 22 minutes and ties the login path to the desktop (Bravais CONSTRAINTS #35) |
| Why not kiosk-rs | Third-party 0.1.0 with about 25 downloads; an unqualified dependency (§24) on the login path |
| Scope | One program, one output at a time, no configuration file. Cage-compatible command shape: `capsule [OPTIONS] [--] <PROGRAM> [ARGS...]` |
| Backends | DRM (KMS + libinput + libseat) for real seats; nested (Smithay's winit backend) for development, running as a window under Wayland **or X11**, LeftWM included |
| Rendering | OpenGL ES, with a required software (pixman) fallback for machines without a working driver (CPS-SRS-060, 061) |
| X11 | Three meanings, kept apart: nested under an X11 session (in scope, free); X11 programs through XWayland (optional, M6); a machine with no KMS (optional X11 kiosk mode, M7: Capsule as a tiny x11rb window manager under `xinit`). Running an X window manager such as LeftWM under Capsule is out of scope — Bravais keeps `startx` for those |
| Introspection | `--describe`, `--schema`, `--dry-run` are **flags, not sub-commands**: the first positional argument is always the program, so a program named `schema` stays runnable. Tailors CLI Standard rules 4 and 7 (see [`COMPLIANCE.md`](COMPLIANCE.md)) |
| Exit status | The child's status passes through unchanged (CPS-SRS-014, 015). Capsule's own failures use the canonical codes and **always** write an error object to stderr; the child's status never comes with one. That is how a caller tells a child that exited 4 from a Capsule that could not open the seat |
| Privacy | Exactly fourteen protocol globals, plus `xwayland_shell_v1` under `--xwayland` (CPS-SRS-035). No screen capture, input injection, layer shell, or virtual keyboard for Wayland clients, ever. X11 clients under XWayland or the X11 kiosk mode can capture and inject through the X server; that limit is accepted for those optional modes (CPS-NEED-006) |
| Scope per backend | Every requirement section states the backends it applies under (G1 second pass, R-01) |
| Output choice | Outputs on the chosen GPU only (no hybrid-GPU spanning). Internal panel first unless the lid is closed beside another output; otherwise the lowest connector name; re-run on every hotplug and lid event (CPS-SRS-057, 058, 068). Scale 1, normal transform (CPS-SRS-122, 123) |
| Keys | Capsule binds nothing but Ctrl+Alt+F1–F12 under DRM (CPS-SRS-077), so no screen-reader chord is taken (§10.4) |
| Palette | Steelbore Modern; the background is the resolved canvas colour (CPS-SRS-048) |
| Category | **B** — a compositor crash kills every program in the session and loses unsaved work (§19.1). Texinfo SRS, CI traceability, budgets, ICD, baselines |
| Reference machine | The maintainer's ThinkPad: Intel Core i7-8665U, 4C/8T, UHD Graphics 620, Bravais. Memory and storage class recorded at G2 |
| Requirement IDs | Prefix `CPS` (`CPS-NEED-nnn`, `CPS-SRS-nnn`), decided 2026-10-27; one obligation per requirement |
| Agentic CLI | `spacecraft-agentic-cli` applies in full. No MCP surface (one command, below the 10-command threshold) |


## Architecture

```text
 greetd ──exec──▶ capsule ──spawn──▶ child (alacritty -e nu)
                    │  ▲                 │
                    │  └──── Wayland ────┘   (socket in $XDG_RUNTIME_DIR)
                    │
   DRM backend:     ├── libseat ──▶ seatd / logind ──▶ /dev/dri/card*, /dev/input/event*
                    ├── udev (GPU, connectors, hotplug)   libinput (keys, pointer, touchpad)
                    └── GBM + EGL + GLES  ·  pixman fallback
   nested backend:  └── winit window on the host's Wayland or X11 display
   X11 kiosk (M7):  xinit ──▶ X server ◀── capsule (x11rb window manager) ──spawn──▶ child
```

| Crate | Role | Rules |
|-------|------|-------|
| `capsule-policy` | Pure decisions: backend choice, GPU and output choice, toplevel placement and stacking, focus stack, exit-status mapping, termination deadline | `#![forbid(unsafe_code)]`; no I/O, no clock (time comes in as an argument); carries the bulk of the test, fuzz, and mutation evidence |
| `capsule` | Binary: CLI (clap), diagnostics, child process, Smithay compositor state, DRM and nested backends | One documented `unsafe` site (below); no `unwrap`/`expect` outside tests |
| `capsule-x11` | X11 kiosk mode (M7): x11rb window manager, behind the `x11-kiosk` feature | Uses `capsule-policy` for placement, focus, and exit status, so both modes share one contract |

**Concurrency (§3.2) — serial by design, recorded trade-off.** One thread runs one calloop
event loop over the Wayland socket and clients, the seat session, udev, libinput, the DRM
vblank events, and a signal source for `SIGCHLD`, `SIGTERM`, `SIGINT`, and `SIGHUP`. Wayland
protocol state is single-owner and must be processed in order; each event costs microseconds;
the GPU already works asynchronously behind the driver. Threads would add synchronisation and
reordering risk and buy nothing at one client and one output. The decision is benchmarked
against the input-latency and idle budgets (CPS-SRS-081, 082) at G2 and revisited only if one
fails.

**Safety model (§3.1).** Device access comes only through the seat session (CPS-SRS-054). On every
exit path Capsule handles, a panic included, it leaves the virtual terminal in text mode
(CPS-SRS-071). After `SIGKILL` it can run nothing, so the kernel drops DRM master and the device
descriptors when the process dies and the seat manager restores the virtual terminal when its
client disconnects — a platform assumption (A-01 in the manual), checked by a system test at G2
and at every release. A fatal error while the child runs ends the child first and exits 1
(CPS-SRS-117); losing the GPU or the seat counts as one (CPS-SRS-118, 119). The child is started with
`PR_SET_PDEATHSIG = SIGTERM`, so it never outlives Capsule. Setting it needs
`CommandExt::pre_exec`, which is `unsafe`: that is the one `unsafe` site in Capsule's own code,
and it calls only the async-signal-safe `prctl(2)`, with the safety argument written at the
site. The release profile uses `panic = "abort"`: hand-back does not depend on unwinding, and
abort never unwinds through Smithay's C callbacks.

**Privacy model (§3.3, §9).** The advertised globals are a closed list (CPS-SRS-035). While the
session is inactive, no input reaches any client (CPS-SRS-066). The child's environment is
Capsule's own plus `WAYLAND_DISPLAY` (CPS-SRS-034); under the nested backend the host's
`DISPLAY` is removed, so an X11 fallback never reaches the host desktop (CPS-SRS-113).
Capsule opens no file in a configuration directory, libraries included (CPS-SRS-110): the
keymap is compiled with xkbcommon's default include paths disabled, because they would read
`$XDG_CONFIG_HOME/xkb`, `~/.xkb`, and `/etc/xkb`; the system XKB data path is added back
explicitly.

**Dependencies (to be qualified at G2, §24).** `smithay` 0.7.0 (MIT; the latest crates.io
release, 2025-06-24), `calloop` 0.14.5, `clap` 4.6.7, `x11rb` 0.14.0 (M7), plus `serde`,
`serde_json`, and a UTC timestamp crate — versions as of 2026-10-27.


## Interfaces

Drafted into the ICD at G2 (§23), recorded in the manual's `Interfaces` node (CPS-SRS-090):

| Interface | Shape |
|-----------|-------|
| Command line | `capsule [OPTIONS] [--] <PROGRAM> [ARGS...]`; `--backend auto\|drm\|nested\|x11` (`x11` exits 2 unless built with `x11-kiosk`); `--describe`, `--schema`, `--dry-run`; every CLI Standard §3 global flag; `--xwayland` (M6) |
| Exit status | Child's status, or 128+N on signal; Capsule's own: 1, 2, 4, 5, 126, 127 with an error object on stderr |
| Diagnostics | stderr only; `[ERROR]`/`[WARN]`/`[OK]`/`[INFO]` in human mode; CLI Standard JSON error and `diagnostic` envelopes in machine mode |
| Environment read | `XDG_RUNTIME_DIR`, `WAYLAND_DISPLAY`, `DISPLAY`, `XKB_DEFAULT_*`, `XCURSOR_THEME`, `XCURSOR_SIZE`, `AI_AGENT`, `AGENT`, `CI`, `NO_COLOR`, `SPACECRAFT_A11Y`, `SPACECRAFT_THEME` |
| Environment written (child) | `WAYLAND_DISPLAY`; `DISPLAY` with `--xwayland` |
| Wayland surface | The fourteen globals of CPS-SRS-035 |
| Packaging surface | Nix package and the Bravais session line `capsule -- alacritty -e <shell>`; Guix package; PKGBUILD |


## Out of scope

- More than one output at a time, tiling, workspaces, panels, a configuration file, IPC, plugins.
- Screen capture, session lock, layer shell, virtual keyboards, input methods.
- Touchscreens, tablets, and fractional scaling (v0.1; revisited only on a stated need).
- Running an X window manager such as LeftWM under Capsule.
- An MCP server.


## Open items for G2

- Record the reference machine's memory and storage class, and commit the reference workload's
  10 MiB text file (CPS-SRS-079 to 082, §22.4).
- Confirm the provisional budget figures before any code consumes them (§22.2).
- Smithay: the crates.io 0.7.0 release is 16 months old; decide between it and a pinned git
  revision, with the §24 qualification record for either.
- Confirm that both seatd and logind restore the virtual terminal when a client is killed
  (CPS-SRS-071), and record the evidence.
- x11rb: align the M7 version with the one Smithay's `xwayland` feature pulls, so the tree holds
  one copy.
- Whether Smithay's software renderer can import dmabufs. If it cannot, `zwp_linux_dmabuf_v1`
  cannot be advertised under the software fallback and CPS-SRS-035 needs a conditional form
  (carried from G1, finding record in `COMPLIANCE.md`).
- Register `Capsule.SpacecraftSoftware.org` in Standard §15.1 with the next Standard release.
  The `PROJECTS.md` row landed 2026-10-27 (Projects #17), without a subdomain.


## M0 — Foundation and gates

- [ ] CPS-SRS-001 REUSE
- [ ] CPS-SRS-002 Posture files
- [ ] CPS-SRS-003 Agent context files
- [ ] CPS-SRS-004 Category declared
- [ ] CPS-SRS-005 Security policy
- [ ] CPS-SRS-006 Tailoring register
- [ ] CPS-SRS-007 CI gate
- [ ] CPS-SRS-008 Text format
- [ ] CPS-SRS-009 Traceability gate


## M1 — Command line and child

- [ ] CPS-SRS-011 Child arguments
- [ ] CPS-SRS-012 End of options
- [ ] CPS-SRS-013 Missing program
- [ ] CPS-SRS-014 Exit status
- [ ] CPS-SRS-015 Signal exit
- [ ] CPS-SRS-016 Program not found
- [ ] CPS-SRS-017 Program not executable
- [ ] CPS-SRS-018 Global flags
- [ ] CPS-SRS-019 Describe
- [ ] CPS-SRS-020 Schema
- [ ] CPS-SRS-021 Dry run
- [ ] CPS-SRS-022 Version
- [ ] CPS-SRS-023 Help
- [ ] CPS-SRS-024 Structured errors
- [ ] CPS-SRS-025 Severity tags
- [ ] CPS-SRS-026 Agent environment
- [ ] CPS-SRS-027 Standard output
- [ ] CPS-SRS-028 Timestamps
- [ ] CPS-SRS-029 Backend flag
- [ ] CPS-SRS-030 Automatic backend
- [ ] CPS-SRS-031 Verbose report
- [ ] CPS-SRS-078 Plain diagnostics
- [ ] CPS-SRS-106 Quiet and verbose together
- [ ] CPS-SRS-107 Quiet
- [ ] CPS-SRS-108 Backend not built
- [ ] CPS-SRS-111 Version as JSON
- [ ] CPS-SRS-121 XWayland not built


## M2 — Nested compositor

- [ ] CPS-SRS-010 Start the program
- [ ] CPS-SRS-032 Socket
- [ ] CPS-SRS-033 Missing runtime directory
- [ ] CPS-SRS-034 Child environment
- [ ] CPS-SRS-035 Advertised globals
- [ ] CPS-SRS-036 Toplevels fullscreen
- [ ] CPS-SRS-037 Dialogs centred
- [ ] CPS-SRS-038 Focus newest
- [ ] CPS-SRS-039 Focus return
- [ ] CPS-SRS-040 Popups
- [ ] CPS-SRS-041 Keymap
- [ ] CPS-SRS-042 Keymap fallback
- [ ] CPS-SRS-043 Pointer focus
- [ ] CPS-SRS-044 Cursor
- [ ] CPS-SRS-045 Clipboard
- [ ] CPS-SRS-046 Primary selection
- [ ] CPS-SRS-047 No decorations
- [ ] CPS-SRS-048 Background
- [ ] CPS-SRS-049 Protocol errors
- [ ] CPS-SRS-050 Lifetime
- [ ] CPS-SRS-051 Termination
- [ ] CPS-SRS-052 Termination deadline
- [ ] CPS-SRS-053 Idle
- [ ] CPS-SRS-109 Stacking
- [ ] CPS-SRS-110 No configuration
- [ ] CPS-SRS-113 Host display removed
- [ ] CPS-SRS-115 Host unreachable
- [ ] CPS-SRS-117 Fatal error
- [ ] CPS-SRS-122 Scale
- [ ] CPS-SRS-123 Transform
- [ ] CPS-SRS-124 Unconstrained popup
- [ ] CPS-SRS-125 Built-in cursor
- [ ] CPS-SRS-126 One output
- [ ] CPS-SRS-073 Window
- [ ] CPS-SRS-074 Window resized
- [ ] CPS-SRS-075 Window closed
- [ ] CPS-SRS-076 No devices
- [ ] CPS-SRS-077 No key bindings


## M3 — Real hardware

- [ ] CPS-SRS-054 Seat session
- [ ] CPS-SRS-055 No seat
- [ ] CPS-SRS-056 GPU choice
- [ ] CPS-SRS-057 Internal panel first
- [ ] CPS-SRS-058 Output fallback
- [ ] CPS-SRS-059 Mode
- [ ] CPS-SRS-060 Hardware rendering
- [ ] CPS-SRS-061 Software rendering
- [ ] CPS-SRS-062 No renderer
- [ ] CPS-SRS-063 Input devices
- [ ] CPS-SRS-064 Device defaults
- [ ] CPS-SRS-065 Virtual terminal switch
- [ ] CPS-SRS-066 Inactive
- [ ] CPS-SRS-067 Reactivation
- [ ] CPS-SRS-068 Output re-chosen
- [ ] CPS-SRS-069 No output
- [ ] CPS-SRS-070 Suspend
- [ ] CPS-SRS-071 Hand-back
- [ ] CPS-SRS-072 Frame pacing
- [ ] CPS-SRS-116 Seat never active
- [ ] CPS-SRS-118 GPU lost
- [ ] CPS-SRS-119 Seat lost


## M4 — Budgets and release

- [ ] CPS-SRS-079 Start-up
- [ ] CPS-SRS-080 Memory
- [ ] CPS-SRS-081 Input latency
- [ ] CPS-SRS-082 Idle CPU
- [ ] CPS-SRS-083 Binary size
- [ ] CPS-SRS-084 Manual
- [ ] CPS-SRS-085 Nix package
- [ ] CPS-SRS-086 Guix package
- [ ] CPS-SRS-087 Arch package
- [ ] CPS-SRS-088 Info directory
- [ ] CPS-SRS-089 SBOM
- [ ] CPS-SRS-090 Interface control
- [ ] CPS-SRS-091 Coverage
- [ ] CPS-SRS-092 Fuzzing
- [ ] CPS-SRS-093 Mutation testing
- [ ] CPS-SRS-127 No desktop in the closure


## M5 — Bravais switch-over

Validation against the needs (§21.5), in the Bravais greeter, with Capsule installed from its
Nix package. Each need is ticked when it has been exercised end to end and recorded in the
release manifest.

- [ ] CPS-NEED-001 One program, whole screen
- [ ] CPS-NEED-002 Ends with its program
- [ ] CPS-NEED-003 Never stranded
- [ ] CPS-NEED-004 Works on the maintainer's hardware
- [ ] CPS-NEED-005 Independent and small
- [ ] CPS-NEED-006 Private by default
- [ ] CPS-NEED-007 Operable by people and agents
- [ ] CPS-NEED-008 Assistive technology first
- [ ] CPS-NEED-009 Developed without a spare seat


## M6 — XWayland: optional

- [ ] CPS-SRS-094 On demand (optional)
- [ ] CPS-SRS-095 Display variable (optional)
- [ ] CPS-SRS-096 X11 windows (optional)
- [ ] CPS-SRS-097 Off by default (optional)


## M7 — X11 kiosk mode: optional

- [ ] CPS-SRS-098 Window manager (optional)
- [ ] CPS-SRS-099 Display taken (optional)
- [ ] CPS-SRS-100 X11 windows fullscreen (optional)
- [ ] CPS-SRS-101 X11 dialogs centred (optional)
- [ ] CPS-SRS-102 X11 focus (optional)
- [ ] CPS-SRS-103 X11 exit status (optional)
- [ ] CPS-SRS-104 X11 key bindings (optional)
- [ ] CPS-SRS-105 X11 background (optional)
- [ ] CPS-SRS-112 X11 stacking (optional)
- [ ] CPS-SRS-114 X11 child environment (optional)
- [ ] CPS-SRS-120 X display unreachable (optional)
- [ ] CPS-NEED-010 Machines without kernel mode setting (optional)

---

*— Built by [Spacecraft Software](https://SpacecraftSoftware.org/) —*
