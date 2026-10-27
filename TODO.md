<!--
SPDX-FileCopyrightText: 2026 Mohamed Hammad <Mohamed.Hammad@SpacecraftSoftware.org>
SPDX-License-Identifier: CC-BY-SA-4.0
-->
<!-- GFM Document
     title:      Capsule — TODO
     author:     Mohamed Hammad & Spacecraft Software
     date:       2026-10-27
     version:    0.1.0
     license:    CC-BY-SA-4.0
     project:    Capsule
     website:    https://Capsule.SpacecraftSoftware.org/
-->

# Capsule — TODO

MVP: M0–M5 — Capsule v0.1 in the Bravais greeter

Executable tasks for [`PLAN.md`](PLAN.md); each cites its work package. One task is one
branch and one PR unless noted. **Done rule:** a task is ticked when its PR is squash-merged
to `main` with the local gate (`nu tools/ci.nu`, from T-016) green; a task that lands a test
ticks only when that test passes. Identifiers are permanent.

## M0 — Foundation and gates

- [x] T-001 P-001 Run the §20.4 gate over CPS-SRS-001–105 and CPS-NEED-001–010; record
      findings in `COMPLIANCE.md` "Gate records / G1"
- [x] T-002 P-001 Fix every G1 finding in `doc/capsule.texi`, in its own PR
- [x] T-080 P-001 Fix the accepted second-pass findings (R-01–R-17 and the NEED-005 note),
      new obligations from CPS-SRS-113, in its own PR
- [ ] T-081 P-001 Third pass over the changed requirements only, by a fresh reader; record it
- [ ] T-003 P-001 Flip every status to `baselined` (after T-018); maintainer pushes signed
      tag `g1-requirements-0.1`
- [x] T-004 P-002 `LICENSE` (GNU's canonical GPL-3.0 text), `LICENSES/` symlink and
      CC-BY-SA-4.0 text, `NOTICE.md`, `CONTRIBUTING.md`, `REUSE.toml`; `reuse lint` exits 0
- [ ] T-005 P-002 `README.md`: Category B, §19.6 claim "tailored, see COMPLIANCE.md", §13 N/A,
      support window, build and run
- [ ] T-006 P-002 `SECURITY.md`: private channel, acknowledgement target (days), scope,
      supported versions, disclosure terms, credit
- [ ] T-007 P-002 `CREDITS.md`: Cage (prior art), Smithay (MIT), ISO/IEC/IEEE 29148,
      ECSS-E-ST-40C
- [ ] T-008 P-002 `COMPLIANCE.md` tailoring register, re-read against the final CLI design
- [x] T-009 P-003 `AGENTS.md` scope, X11 decisions, invariants; `CLAUDE.md` imports it
- [ ] T-010 P-003 `AGENTS.md`: Category B, layout, build/test/lint commands, forbidden
      patterns, requirement markers (`Verifies:` / `Implements:`)
- [ ] T-011 P-003 `SKILL.md`: command line, flags, exit codes, examples; description at most
      1000 rendered characters
- [ ] T-012 P-004 `PROJECTS.md` row in the Projects repo (Category B, prefix `CPS`, no
      subdomain until §15.1 lists it)
- [ ] T-013 P-005 `Cargo.toml` workspace with `capsule-policy` and `capsule`;
      `workspace.package` (edition 2024, licence, repository, homepage, authors,
      `publish = false`, `rust-version`)
- [ ] T-014 P-005 Release profile: `lto = "fat"`, `codegen-units = 1`, `panic = "abort"`,
      `strip = true`; each flag commented, `-march` left off and why; workspace lints deny
      `unwrap_used`, `expect_used`, `panic`
- [ ] T-015 P-005 `clippy.toml`, `rustfmt.toml`, `deny.toml` (licences, advisories, bans,
      sources), `.cargo/audit.toml`; `flake.nix` devShell (Rust toolchain, cargo-audit, -deny,
      -nextest, -mutants, -llvm-cov, -fuzz, -cyclonedx, hyperfine, reuse, texinfo, nushell,
      libseat, libinput, libxkbcommon, Mesa, pixman, udev, wayland) and `flake.lock`; crate
      stubs build clean
- [ ] T-016 P-006 `tools/ci.nu` local gate and `.github/workflows/ci.yml` running the same
      steps; CR-byte check through `git ls-files --eol`
- [ ] T-017 P-007 `doc/requirements.toml` from the `Requirements` chapter (105 rows: id,
      title, text, rationale, source, method, priority, status, milestone)
- [ ] T-018 P-007 `tools/srs.nu` renders `doc/requirements.texi`; the manual `@include`s it
      and builds clean
- [ ] T-019 P-007 `tools/trace.nu` emits the matrix and fails on an orphan or unknown
      identifier; wired into CI
- [ ] T-020 P-008 Record the reference machine's memory and storage class; commit the
      reference workload input
- [ ] T-021 P-008 Qualification records (§24.1) for Smithay, calloop, clap, serde, serde_json,
      the timestamp crate; Smithay crates.io-vs-git decision in `Design`
- [ ] T-022 P-008 Show that seatd and logind restore the virtual terminal when a client is
      killed (a throwaway libseat client), recorded in `COMPLIANCE.md` "Gate records / G2"
- [ ] T-023 P-008 Manual `Design` node: architecture, serial trade-off, safety, privacy;
      `Interfaces` node drafted; budgets confirmed or re-baselined with a dated note
- [ ] T-024 P-008 Maintainer pushes signed tag `g2-design-0.1`

## M1 — Command line and child

- [ ] T-025 P-009 `capsule-policy::exit`: child status to Capsule status (normal, 128+N),
      with property tests over every status and signal number
- [ ] T-026 P-009 `capsule-policy::terminate`: the SIGTERM-then-SIGKILL deadline as a pure
      state machine fed by timestamps
- [ ] T-027 P-010 clap definition: global flags, `--backend`, `--` handling, trailing
      arguments passed verbatim; missing-program error with a runnable hint; exit 2 for
      `--quiet` with `--verbose` and for a backend not built in
- [ ] T-028 P-011 `capsule-policy::backend`: auto selection from `WAYLAND_DISPLAY` and
      `DISPLAY`, table-driven tests
- [ ] T-029 P-012 Output-mode cascade (flag, agent environment, TTY, non-TTY) with
      presence-based detection, tested under the host's real `AI_AGENT` value
- [ ] T-030 P-012 Diagnostic type: severity tags, JSON error object, `diagnostic` envelope,
      UTC timestamps; plain mode for `--accessible`, `--no-color`, `--color never`,
      `SPACECRAFT_A11Y`, `NO_COLOR`; the `--quiet` severity floor
- [ ] T-031 P-013 `--version` and `--help` (text and JSON) with the maintainer line and URL;
      `maintainer` and `website` keys in the JSON metadata
- [ ] T-032 P-013 `--describe` manifest and `--schema` (JSON Schema 2020-12, validated in a
      test)
- [ ] T-033 P-013 `--dry-run` JSON report (backend now, GPU and output from T-055)
- [ ] T-034 P-014 Child spawn with `PR_SET_PDEATHSIG` in `pre_exec`; the safety argument at
      the site; 126 and 127 detection; tests with missing and non-executable programs
- [ ] T-035 P-014 `SIGCHLD` and termination signals as calloop sources; test that Capsule
      writes nothing to standard output on a normal run
- [ ] T-036 P-015 `--verbose` `[INFO]` report of backend, renderer, output, and mode

## M2 — Nested compositor

- [ ] T-037 P-016 Smithay compositor state and calloop loop; socket in `XDG_RUNTIME_DIR`;
      child started after the socket listens; missing-runtime-directory and failed-socket
      errors; test that no file in a configuration directory is opened
- [ ] T-038 P-016 Child environment test: only `WAYLAND_DISPLAY` differs from Capsule's own
- [ ] T-039 P-017 The fourteen globals; a test client lists the registry and compares it to
      the closed set
- [ ] T-040 P-018 `capsule-policy::layout`: parentless toplevels fullscreen, dialogs
      centred, newest on top, focus stack; property tests over map and unmap sequences,
      including a splash window mapped before the main one
- [ ] T-041 P-018 Wire the layout into `xdg_shell` configure, `set_parent`, and unmap
      handling
- [ ] T-042 P-019 Popup placement with positioner constraint adjustment, tested at each edge
- [ ] T-043 P-020 Keymap from `XKB_DEFAULT_*`; fallback with `[WARN]` on a keymap that does
      not compile (`evdev`/`pc105`/`us`); xkbcommon default includes disabled, system XKB
      path added explicitly
- [ ] T-044 P-020 Pointer focus, client cursors, `cursor-shape`, default cursor from
      `XCURSOR_THEME` and `XCURSOR_SIZE` (`default`, 24); built-in cursor image when no theme
      loads
- [ ] T-045 P-021 `wl_data_device` and primary selection between two test clients
- [ ] T-046 P-022 `xdg-decoration` answered server-side; canvas colour from the §11.6
      resolution, read from `steelbore.toml` at build time, never retyped
- [ ] T-047 P-023 Protocol-error isolation test with a deliberately broken client
- [ ] T-048 P-023 Lifetime and termination: child outlives its windows; SIGTERM/SIGINT/SIGHUP
      forwarded; 5 s deadline then SIGKILL; fatal-error path ends the child and exits 1
- [ ] T-049 P-024 Damage tracking; test that an idle session submits no frame
- [ ] T-050 P-025 winit backend: one window, resize reconfigures the toplevel, close sends
      SIGTERM, no device node opened (checked through `/proc/self/fd`); exit 1 with a
      `--backend drm` hint when the host is unreachable; host `DISPLAY` removed from the child
- [ ] T-051 P-026 Key delivery test: every chord reaches the client, the Orca and NVDA
      modifier chords included
- [ ] T-052 P-027 Headless integration harness in CI; Alacritty + Nushell exercised nested
      under niri and under LeftWM, recorded

## M3 — Real hardware

- [ ] T-053 P-028 libseat session (seatd and logind); exit 4 with a hint when none opens;
      exit 1 after 30 s inactive; seat loss as a fatal error
- [ ] T-054 P-029 `capsule-policy::output`: `boot_vga` GPU, internal panel first unless the
      lid is closed beside another output, lowest
      connector name otherwise; tests from recorded sysfs, udev, and lid fixtures
- [ ] T-055 P-030 DRM output at the preferred mode (first listed when none is preferred);
      vblank-paced frames; `--dry-run` reports GPU and output from udev and sysfs;
      `--verbose` reports the real renderer and output
- [ ] T-056 P-031 GBM + EGL + GLES renderer; pixman fallback with `[WARN]`; exit 1 naming
      both failures; tested with GLES forced off
- [ ] T-057 P-032 libinput devices at their defaults; TrackPoint middle-button scrolling
      checked on the ThinkPad
- [ ] T-058 P-033 Ctrl+Alt+F1–F12; no input and no frames while inactive; frame within 1 s of
      reactivation
- [ ] T-059 P-034 Output choice re-run on hotplug and lid events within 2 s; GPU removal as a
      fatal error; no-output state keeps the
      child; resume on connect
- [ ] T-060 P-035 `hardware-tests` feature: suspend/resume and hand-back after panic and
      SIGKILL (the A-01 system test), each recorded with date and machine
- [ ] T-061 P-036 Ad-hoc greeter session entry on the ThinkPad; a working session in
      Alacritty + Nushell, recorded

## M4 — Budgets and release

- [ ] T-062 P-037 Budget harness: hyperfine start-up, peak RSS, key-to-client latency probe,
      idle CPU, stripped size; run on the reference machine for each release build;
      nested-backend figures tracked in CI; `BUDGET:` margin lines in the report
- [ ] T-063 P-038 `cargo fuzz` targets for the argument parser, backend choice, output
      choice, layout, and keymap-variable parsing; CI runs each for 60 s on every merge
- [ ] T-064 P-039 `cargo llvm-cov` gate at 80% line coverage of Capsule's own crates
- [ ] T-065 P-039 `cargo mutants` run; each survivor killed or justified in
      `COMPLIANCE.md` "Gate records / G3"
- [ ] T-066 P-040 Manual user chapters: Invocation, Backends, Exit status, Environment,
      Troubleshooting
- [ ] T-067 P-040 `Interfaces` node completed and baselined; Info, HTML (`spacecraft.css`),
      and A4 PDF builds in CI with no warning
- [ ] T-068 P-041 `packaging/default.nix`, `packaging/guix.scm`, `packaging/PKGBUILD` with
      version, SHA-256, and `install-info` hooks; each built in a clean environment; Nix
      closure checked free of COSMIC, GNOME, Plasma, and niri packages
- [ ] T-069 P-042 CycloneDX JSON SBOM from the lockfile, attached with its SHA-256 to the
      GitHub release by the release CI run
- [ ] T-070 P-042 Release manifest: identity, artifacts and checksums, provenance, V&V
      summary, budgets with margins, known issues, conformance claim; traceability matrix
      attached
- [ ] T-071 P-042 `CHANGELOG.md` for 0.1.0; §16 checklist run; maintainer pushes signed tag
      `v0.1.0`

## M5 — Bravais switch-over

- [ ] T-072 P-043 Add the `capsule` flake input to Bravais and expose the package
- [ ] T-073 P-044 `mkShellSession` `Exec` becomes `capsule -- alacritty -e <shell>`; remove
      the cosmic-comp kiosk comment; re-check the seatd and `seat` group notes
- [ ] T-074 P-044 Bravais `PRD.md` §8.2, `AGENTS.md`, `CHANGELOG.md` updated in the same PR
- [ ] T-075 P-045 Log in and out of the Brush, Ion, mjsh, and Nushell sessions; exercise each
      of CPS-NEED-001–009; record the validation note in the release manifest

## M6 — XWayland: optional

- [ ] T-076 P-046 `xwayland` Cargo feature and `--xwayland` flag; Xwayland on first X11
      client; `DISPLAY` for the child; X11 windows through the layout policy (optional)

## M7 — X11 kiosk mode: optional

- [ ] T-077 P-047 `capsule-x11` crate behind `x11-kiosk`; `SubstructureRedirect` ownership;
      exit 5 when another window manager holds the display (optional)
- [ ] T-078 P-048 Map, centre, stack, and focus through `capsule-policy`, with
      `WM_TRANSIENT_FOR` as the parent; no key grabs; root background in the canvas colour
      (optional)
- [ ] T-079 P-049 Exit-status contract under `xinit`; tests under Xephyr; validation on a
      machine without KMS (optional)
