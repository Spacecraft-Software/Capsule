<!--
SPDX-FileCopyrightText: 2026 Mohamed Hammad <Mohamed.Hammad@SpacecraftSoftware.org>
SPDX-License-Identifier: CC-BY-SA-4.0
-->

# AGENTS.md — Capsule

Capsule is a single-application Wayland kiosk compositor in Rust, built on
[Smithay](https://github.com/Smithay/smithay). It runs one program
fullscreen, and exits with that program's exit status when it ends. It
replaces [Cage](https://github.com/cage-kiosk/cage), and cosmic-comp's kiosk
mode, in the Steelbore OS Bravais greeter's shell sessions
(`bravais/modules/login/default.nix`, `mkShellSession`).

Status: pre-release, no code yet. There are no build, lint, or test commands
yet; this file gains them with the Cargo workspace. Milestones and progress
are in [`PLAN.md`](PLAN.md) (Standard §17.5).

## Scope

In scope:

- DRM/KMS output, libinput input, and seat access through libseat (seatd or
  logind). OpenGL ES rendering, with a software (pixman) fallback.
- One fullscreen `xdg_toplevel` on one output, plus its popups.
- Keyboard and pointer focus, the clipboard (`wl_data_device`), and the
  primary selection (Alacritty uses both).
- Ctrl+Alt+F*n* virtual-terminal switching.
- Output hotplug and suspend/resume without losing the session.
- A nested backend (Smithay's winit backend) for development. It runs as a
  window under both a Wayland session and an X11 one, LeftWM included.

Optional, later: XWayland, and the X11 kiosk mode (see [X11](#x11)).

Out of scope: multiple windows, tiling, workspaces, panels, a configuration
file, screen capture, session lock, plugins, IPC.

## X11

Three different things go by "X11 support". Keep them apart:

| Meaning | Decision |
|---|---|
| Run Capsule nested inside an X11 session, such as LeftWM, for development | **In scope, free.** The winit backend already does this |
| Run an X11-only program as Capsule's one application, through XWayland | **Optional, later milestone**, behind a Cargo feature `xwayland` that is off by default. The current Bravais children (Alacritty) are native Wayland |
| Run an X11 window manager such as LeftWM itself under Capsule | **Out of scope.** An X window manager needs an X server, not a Wayland compositor. Bravais starts those sessions with `startx` (`mkXSession`), and that stays as is |
| Kiosk on a machine with no KMS, where only an X server can drive the display | **Optional, later milestone: X11 kiosk mode.** Capsule acts as a tiny X11 window manager (x11rb), launched as the `xinit` client. It shows one program fullscreen, gives it focus, and exits with its status, which ends the X server. Same command line and exit-status contract as the Wayland mode |

A machine with KMS but no usable OpenGL ES (old GPUs, VMs, simpledrm) is not
an X11 case. The Wayland mode falls back to Smithay's software renderer
(pixman) there, and that fallback is required, not optional.

## Command line (planned)

`capsule [OPTIONS] [--] <PROGRAM> [ARGS...]`. Everything after the program
name belongs to the child. `--` ends Capsule's own options, so a child whose
name starts with `-` cannot be mistaken for a Capsule flag. Capsule's exit
status is the child's. A child killed by signal *N* gives 128+*N*. Capsule's
own failures use the CLI Standard exit codes.

## Invariants

- **Never grab the screen-reader modifier chords** (Standard §10.4). Capsule
  binds nothing except virtual-terminal switching.
- **The event loop is single-threaded** (calloop), as Wayland compositors are
  by design. That serial choice is the §3.2 trade-off, and it gets written
  down when the code lands. Work that does not touch protocol state may move
  off the loop only after benchmarks show it helps.
- **No `unwrap`/`expect` on device or client input.** A misbehaving client
  is disconnected; it never takes the compositor down.
- **Devices go back to the seat on every exit path**, a crash included.

## Conventions

- Branch + PR, squash-merge, signed commits (Standard §6.3). Never push to
  `main`. Push over HTTPS.
- GitHub repo: `Spacecraft-Software/Capsule`. The local directory and crate
  names are lowercase (`capsule`).
- Smithay is MIT-licensed. Credit it in `CREDITS.md` when the first
  dependency lands (§15.3).
