<!--
SPDX-FileCopyrightText: 2026 Mohamed Hammad <Mohamed.Hammad@SpacecraftSoftware.org>
SPDX-License-Identifier: CC-BY-SA-4.0
-->

# Capsule — Plan

MVP: M0–M4 — Capsule v0.1 in the Bravais greeter

Each item is ticked when its work is done and checked, never when it is
started (Standard §17.5). Items ending `(optional)` never count toward a
milestone's figure. Scope and invariants live in [`AGENTS.md`](AGENTS.md).

## M0 — Workspace, gates and command line

- [ ] P-001 Cargo workspace: one `capsule` binary crate, pinned toolchain
- [ ] P-002 `flake.nix` dev shell: libseat, libinput, libxkbcommon, Mesa, pixman, udev
- [ ] P-003 CI: fmt, clippy `-D warnings`, tests, `cargo audit`, `cargo deny`, `reuse lint`, CR-byte check
- [ ] P-004 Release flags noted in the build config, applied and disabled alike (§3.2)
- [ ] P-005 Command line `capsule [OPTIONS] [--] <PROGRAM> [ARGS...]`, `--help` and `--version` with the maintainer line
- [ ] P-006 Exit-code enum and structured errors on stderr; output mode detection (CLI Standard §4–§5)
- [ ] P-007 Child spawn and exit-status mapping (status, or 128+N on signal), tested without a display
- [ ] P-008 `SECURITY.md` and `CREDITS.md` (Smithay)
- [ ] P-009 Assurance category and conformance claim declared in `README.md` and `AGENTS.md` (§19)
- [ ] P-010 Capsule registered in `PROJECTS.md` and in the Standard's §15.1 subdomain registry

## M1 — Nested in a window

- [ ] P-011 winit backend with the GLES renderer, one output
- [ ] P-012 Core protocols: `wl_compositor`, `wl_shm`, `linux-dmabuf`, `wl_seat`, `wl_output`, `xdg_shell`
- [ ] P-013 First toplevel fullscreen; later toplevels centred over it, newest focused
- [ ] P-014 Popups positioned and constrained to the output
- [ ] P-015 Keyboard through xkbcommon, keymap from the `XKB_DEFAULT_*` variables
- [ ] P-016 Pointer and cursor (client cursor surfaces and `cursor-shape`)
- [ ] P-017 Clipboard (`wl_data_device`) and primary selection
- [ ] P-018 No decorations: `xdg-decoration` answered server-side, nothing drawn
- [ ] P-019 Capsule exits with the child's status; the child sees `WAYLAND_DISPLAY`
- [ ] P-020 Headless integration tests drive a test client through the protocols above
- [ ] P-021 Alacritty running Nushell works in the nested window, under niri and under LeftWM

## M2 — Real hardware

- [ ] P-022 Seat access through libseat, with seatd and with logind
- [ ] P-023 GPU discovery through udev; primary GPU chosen
- [ ] P-024 DRM/KMS output: first connected output, preferred mode
- [ ] P-025 GBM + GLES rendering on the DRM output
- [ ] P-026 Software (pixman) fallback when GLES is unavailable, tested with GLES disabled
- [ ] P-027 Input devices through libinput
- [ ] P-028 Ctrl+Alt+F*n* switches virtual terminals; devices pause and resume
- [ ] P-029 Started from greetd on the ThinkPad, Alacritty + Nushell usable

## M3 — Hardening and release

- [ ] P-030 Output hotplug: the session moves to a remaining output instead of ending
- [ ] P-031 Suspend and resume keep the session
- [ ] P-032 Devices go back to the seat on every exit path, a panic included
- [ ] P-033 A client protocol error disconnects that client; the compositor keeps running
- [ ] P-034 No screen-reader modifier chord grabbed (§10.4); §18 tailoring entry recorded
- [ ] P-035 Budgets declared and measured: start-up time, memory, frame latency (§22)
- [ ] P-036 The single-threaded event loop documented as the §3.2 trade-off, with a benchmark
- [ ] P-037 Texinfo manual `doc/capsule.texi`, building to Info, HTML and PDF (§8)
- [ ] P-038 `packaging/guix.scm`, `packaging/default.nix`, `packaging/PKGBUILD` build (§5.5)
- [ ] P-039 `CHANGELOG.md` and the v0.1.0 tag

## M4 — Bravais switch-over

- [ ] P-040 Capsule added as a Bravais flake input
- [ ] P-041 `mkShellSession` wraps sessions in `capsule --` instead of `cosmic-comp`
- [ ] P-042 Brush, Ion, mjsh and Nushell sessions each log in and out cleanly
- [ ] P-043 Bravais `PRD.md`, `AGENTS.md`, `CHANGELOG.md` updated; seatd note re-checked

## M5 — XWayland: optional

- [ ] P-044 XWayland started on demand behind the `xwayland` Cargo feature (optional)
- [ ] P-045 An X11-only program runs as the one application (optional)

## M6 — X11 kiosk mode: optional

- [ ] P-046 x11rb window manager, run as the `xinit` client (optional)
- [ ] P-047 The program shown fullscreen with focus; later windows centred over it (optional)
- [ ] P-048 Same command line and exit-status contract as the Wayland mode (optional)
- [ ] P-049 Tested under Xephyr, then on a machine without KMS (optional)
